## スプラウトクラス（Sprout Class）

> 💡 **用語: スプラウトクラス（Sprout Class）とは？**
> 既存の巨大なクラスに新機能を追加したい時、既存クラスを直接修正するのではなく、 **新しい小さなクラス（芽＝スプラウト）を別に作って** そこに新機能を実装する手法。既存コードを壊すリスクをゼロにしながら機能追加できる。

### ❌ 悪い例（既存の巨大クラスに直接追加）

**Java:**
```java
// ❌ 既に500行以上ある巨大クラスに新機能を直接追加
class OrderService {
    // ... 既存のコード500行 ...

    void processOrder(Order order) { /* 既存の複雑な処理 */ }
    void cancelOrder(Order order) { /* 既存の複雑な処理 */ }

    // ↓ 新しい「ギフトラッピング機能」を直接追加してしまう
    void applyGiftWrapping(Order order, String message) {
        // ギフト用バリデーション
        if (message == null || message.length() > 200) {
            throw new IllegalArgumentException("メッセージは200文字以内");
        }
        // ラッピング料金計算
        int wrappingFee = order.getItemCount() * 300;
        order.addFee(wrappingFee);
        order.setGiftMessage(message);
        // ← 既存の500行のコードと混ざり合い、どんどん肥大化する
    }
}
```

**Python:**
```python
# ❌ 巨大クラスに直接追加
class OrderService:
    # ... 既存コード500行 ...

    def apply_gift_wrapping(self, order: 'Order', message: str) -> None:
        if not message or len(message) > 200:
            raise ValueError("メッセージは200文字以内")
        wrapping_fee: int = order.item_count * 300
        order.add_fee(wrapping_fee)
        order.gift_message = message
```

**TypeScript:**
```typescript
// ❌ 巨大クラスに直接追加
class OrderService {
    // ... 既存コード500行 ...

    applyGiftWrapping(order: Order, message: string): void {
        if (!message || message.length > 200) throw new Error("メッセージは200文字以内");
        const wrappingFee = order.itemCount * 300;
        order.addFee(wrappingFee);
        order.giftMessage = message;
    }
}
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 既存コードを壊すリスク | 500行のクラスに手を入れると、 **既に動いている処理に予期しない影響** を与える可能性がある |
| 2 | 🔴 クラスの肥大化 | 機能が追加されるたびにクラスが膨らみ、 **理解・テスト・修正が困難** になっていく |
| 3 | 🔴 単一責任原則の違反 | 「注文処理」と「ギフトラッピング」は別の関心事なのに **1つのクラスに混在** している |

> 💡 **用語: 単一責任原則（Single Responsibility Principle / SRP）とは？**
> 1つのクラスは **1つの責任（理由）だけ** を持つべきという設計原則。「変更する理由が2つ以上あるクラスは分割すべき」とも言い換えられる。

---

### 🌟 改善版コード

**一言でいうと:** 新機能を **独立した小さなクラス（スプラウトクラス）** として作り、既存クラスから呼び出す。既存クラスのコードには最小限の変更（1〜2行の呼び出し追加）だけで済む。

**Java:**
```java
// ✅ スプラウトクラス：ギフトラッピングの責任だけを持つ新クラス
class GiftWrapping {
    private static final int MAX_MESSAGE_LENGTH = 200;
    private static final int FEE_PER_ITEM = 300;

    private final String message;
    private final int fee;

    GiftWrapping(final String message, final int itemCount) {
        if (message == null || message.length() > MAX_MESSAGE_LENGTH) {
            throw new IllegalArgumentException(
                "メッセージは" + MAX_MESSAGE_LENGTH + "文字以内です。"
            );
        }
        if (itemCount < 1) {
            throw new IllegalArgumentException("商品数は1以上です。");
        }
        this.message = message;
        this.fee = itemCount * FEE_PER_ITEM;
    }

    String getMessage() { return message; }
    int getFee() { return fee; }
}

// ✅ 既存クラスには最小限の変更だけ（1行追加）
class OrderService {
    // ... 既存の500行はそのまま触らない ...

    void applyGiftWrapping(Order order, String message) {
        // 新クラスに処理を委譲する（既存コードへの影響はこの1行だけ）
        GiftWrapping gift = new GiftWrapping(message, order.getItemCount());
        order.addFee(gift.getFee());
        order.setGiftMessage(gift.getMessage());
    }
}
```

📝 **使い方:**
```java
// ✅ GiftWrapping は独立してテスト可能！
GiftWrapping gift = new GiftWrapping("お誕生日おめでとう！", 3);
System.out.println(gift.getFee());     // 900円
System.out.println(gift.getMessage()); // "お誕生日おめでとう！"

// ❌ バリデーションも GiftWrapping が担当
// new GiftWrapping(null, 0);  // → 例外！OrderService を巻き込まない
```

**Python:**
```python
# ✅ スプラウトクラス
class GiftWrapping:
    MAX_MESSAGE_LENGTH: int = 200
    FEE_PER_ITEM: int = 300

    def __init__(self, message: str, item_count: int) -> None:
        if not message or len(message) > self.MAX_MESSAGE_LENGTH:
            raise ValueError(f"メッセージは{self.MAX_MESSAGE_LENGTH}文字以内です。")
        if item_count < 1:
            raise ValueError("商品数は1以上です。")
        self._message: str = message
        self._fee: int = item_count * self.FEE_PER_ITEM

    @property
    def message(self) -> str:
        return self._message

    @property
    def fee(self) -> int:
        return self._fee

# ✅ 既存クラスからは委譲するだけ
class OrderService:
    # ... 既存コードはそのまま ...
    def apply_gift_wrapping(self, order: 'Order', message: str) -> None:
        gift = GiftWrapping(message, order.item_count)
        order.add_fee(gift.fee)
        order.gift_message = gift.message
```

**TypeScript:**
```typescript
// ✅ スプラウトクラス
class GiftWrapping {
    private static readonly MAX_MESSAGE_LENGTH = 200;
    private static readonly FEE_PER_ITEM = 300;

    readonly message: string;
    readonly fee: number;

    constructor(message: string, itemCount: number) {
        if (!message || message.length > GiftWrapping.MAX_MESSAGE_LENGTH) {
            throw new Error(`メッセージは${GiftWrapping.MAX_MESSAGE_LENGTH}文字以内です。`);
        }
        if (itemCount < 1) throw new Error("商品数は1以上です。");
        this.message = message;
        this.fee = itemCount * GiftWrapping.FEE_PER_ITEM;
    }
}

// ✅ 既存クラスからは委譲するだけ
class OrderService {
    // ... 既存コードはそのまま ...
    applyGiftWrapping(order: Order, message: string): void {
        const gift = new GiftWrapping(message, order.itemCount);
        order.addFee(gift.fee);
        order.giftMessage = gift.message;
    }
}
```

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| 既存コードへの影響 | 500行のクラスに直接追加 | 既存クラスは1〜2行の呼び出し追加のみ |
| テスト | 巨大クラス全体をテスト対象にする必要がある | `GiftWrapping` だけを独立してテスト可能 |
| 責任の分離 | 「注文処理」+「ギフト」が混在 | 各クラスが1つの責任だけを持つ |

<br>

---

### 🎯 まとめ（6つの設計パターン一覧）

| パターン | 解決する問題 | 核心 |
|---|---|---|
| 完全コンストラクタ | 生焼けオブジェクト | 生成時に **全チェック完了** 。セッター不要 |
| 値オブジェクト | プリミティブ型の取り違え | 概念ごとに **専用クラス** を作る |
| ストラテジ | `if-else` の肥大化 | 処理を **クラスに切り出して差し替え** 可能に |
| ポリシー | 条件判定の固定化 | 条件を **独立クラスにして組み合わせ** 自由 |
| ファーストクラスコレクション | コレクション操作の散在 | コレクションを **専用クラスで包んで隠蔽** |
| スプラウトクラス | 巨大クラスの肥大化 | 新機能は **別クラスとして芽吹かせる** |

> ※ これらのパターンは **組み合わせて使う** のが前提です。例えば「完全コンストラクタ + 値オブジェクト」は常にセットで使います。
