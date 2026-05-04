## 共通処理クラス（Common / Util）の罠 ― 低凝集を招く「なんでも箱」

> 💡 **用語: 共通処理クラス（Common / Util Class）とは？**
> `Common` や `Util` という名前で、いろいろな処理を **static メソッドとしてまとめたクラス** のこと。便利に見えるが、 **関連性の薄いロジックが1つのクラスに集まる** ため、低凝集の温床になりやすい。

> 💡 **用語: 横断的関心事（Cross-Cutting Concern）とは？**
> ログ出力、エラーハンドリング、認証チェックなど、 **アプリケーション全体で横断的に必要な処理** のこと。これらは特定のドメインオブジェクトに属さないため、例外的に共通クラスにまとめてもよい場合がある。

### ❌ 悪い例（なんでも Common クラスに入れてしまう）

**Java:**
```java
// ❌ 税計算も割引計算もポイント計算も全部「共通処理」として詰め込み
class Common {
    // 税込み価格を計算
    static int calcTaxIncludedPrice(int price, float taxRate) {
        if (price < 0) throw new IllegalArgumentException();
        if (taxRate < 0) throw new IllegalArgumentException();
        return (int)(price * (1.0f + taxRate));
    }

    // 割引価格を計算
    static int calcDiscountedPrice(int price, float discountRate) {
        if (price < 0) throw new IllegalArgumentException();
        if (discountRate < 0.0f || discountRate > 1.0f) {
            throw new IllegalArgumentException();
        }
        return (int)(price * (1.0f - discountRate));
    }

    // ポイント付与率を計算
    static int calcBonusPoint(int price, float pointRate) {
        if (price < 0) throw new IllegalArgumentException();
        if (pointRate < 0) throw new IllegalArgumentException();
        return (int)(price * pointRate);
    }
}
```

📝 **使い方:**
```java
int taxIncluded = Common.calcTaxIncludedPrice(1000, 0.10f);
int discounted  = Common.calcDiscountedPrice(1000, 0.20f);
int bonus       = Common.calcBonusPoint(1000, 0.01f);
```

**Python:**
```python
# ❌ 税計算も割引計算もポイント計算も全部「共通処理」として詰め込み
class Common:
    @staticmethod
    def calc_tax_included_price(price: int, tax_rate: float) -> int:
        if price < 0:
            raise ValueError()
        if tax_rate < 0:
            raise ValueError()
        return int(price * (1.0 + tax_rate))

    @staticmethod
    def calc_discounted_price(price: int, discount_rate: float) -> int:
        if price < 0:
            raise ValueError()
        if discount_rate < 0.0 or discount_rate > 1.0:
            raise ValueError()
        return int(price * (1.0 - discount_rate))

    @staticmethod
    def calc_bonus_point(price: int, point_rate: float) -> int:
        if price < 0:
            raise ValueError()
        if point_rate < 0:
            raise ValueError()
        return int(price * point_rate)
```

📝 **使い方:**
```python
tax_included = Common.calc_tax_included_price(1000, 0.10)
discounted   = Common.calc_discounted_price(1000, 0.20)
bonus        = Common.calc_bonus_point(1000, 0.01)
```

**TypeScript:**
```typescript
// ❌ 税計算も割引計算もポイント計算も全部「共通処理」として詰め込み
class Common {
    static calcTaxIncludedPrice(price: number, taxRate: number): number {
        if (price < 0) throw new Error();
        if (taxRate < 0) throw new Error();
        return Math.floor(price * (1 + taxRate));
    }

    static calcDiscountedPrice(price: number, discountRate: number): number {
        if (price < 0) throw new Error();
        if (discountRate < 0 || discountRate > 1) throw new Error();
        return Math.floor(price * (1 - discountRate));
    }

    static calcBonusPoint(price: number, pointRate: number): number {
        if (price < 0) throw new Error();
        if (pointRate < 0) throw new Error();
        return Math.floor(price * pointRate);
    }
}
```

📝 **使い方:**
```typescript
const taxIncluded = Common.calcTaxIncludedPrice(1000, 0.10);
const discounted  = Common.calcDiscountedPrice(1000, 0.20);
const bonus       = Common.calcBonusPoint(1000, 0.01);
```

---

### 🚨 何がダメなのか？ (共通処理クラスの問題点)

> 💡 **用語: 低凝集（Low Cohesion）とは？**
> 関連するデータとロジックが **1つのクラスにまとまっていない** 状態。逆に、 **関連のない処理が1つのクラスに雑多に詰め込まれている** 状態も低凝集。共通処理クラスは後者の典型例。

<br>

#### 1. 🔴 関連性のない処理が1つのクラスに同居している
> 対象: `Common` クラスの3つの static メソッド

| 名前 | 問題 |
|---|---|
| `Common` の責務 | 「税込み計算」「割引計算」「ポイント計算」はそれぞれ **異なるドメイン概念** 。本来は別々のクラスに属すべきロジックが、「共通」という漠然とした名前のもとに **1箇所に雑多に詰め込まれている** 。新しい処理が追加されるたびに `Common` が際限なく肥大化する |

<br>

#### 2. 🔴 「共通」という名前がさらなる低凝集を誘発する
> 対象: クラス名 `Common` / `Util`

| 名前 | 問題 |
|---|---|
| 名前の誘引力 | `Common` や `Util` は **「ここに入れて良い」という心理的なハードルが低い** 。開発者が「どのクラスに書けばいいかわからない」と迷ったとき、安易に `Common` に追加してしまう。結果、 **関連のないメソッドが次々と追加** され、クラスが巨大な「なんでも箱」になる |

<br>

#### 3. 🔴 データとロジックの分離（カプセル化の崩壊）
> 対象: `int price` というプリミティブ型の引数

| 名前 | 問題 |
|---|---|
| データの所在 | `price` や `taxRate` は **引数として外から渡される** だけで、 `Common` クラスはデータを持っていない。5-2（static メソッドの誤用）と同じ構造で、 **データとロジックが分離** している。各メソッドで同じバリデーション（`price < 0`）が繰り返されている |

<br>

#### 4. 🔴 クラスが肥大化すると修正の影響範囲が読めなくなる
> 対象: `Common` クラスを参照するすべてのコード

| 名前 | 問題 |
|---|---|
| 影響範囲 | `Common` があらゆる箇所から使われるため、1つのメソッドを修正すると **どこに影響するか把握しづらい** 。本来無関係な機能（税計算とポイント計算）が同じクラスにあることで、変更時の **テスト範囲も不必要に広がる** |

---

### 🌟 改善版コード（ドメインごとに専用クラスを作る）

> 💡 **用語: 単一責任の原則（Single Responsibility Principle / SRP）とは？**
> クラスが持つべき責任（役割）は **1つだけ** にすべきという原則。「税込み価格を計算する」と「ポイントを計算する」は別の責任なので、別のクラスに分ける。

**一言でいうと:** `Common` に雑多に詰め込んでいた処理を、 **「税込み価格」「割引価格」「ポイント」それぞれの専用クラス** に分割し、データとロジックを一体化する。

**Java:**
```java
// ✅ 税込み価格を表す専用クラス
class TaxIncludedPrice {
    private final int amount;

    TaxIncludedPrice(final int price, final float taxRate) {
        if (price < 0) {
            throw new IllegalArgumentException("価格は0以上を指定してください。");
        }
        if (taxRate < 0) {
            throw new IllegalArgumentException("税率は0以上を指定してください。");
        }
        this.amount = (int)(price * (1.0f + taxRate));
    }

    int getAmount() {
        return amount;
    }
}

// ✅ 割引価格を表す専用クラス
class DiscountedPrice {
    private final int amount;

    DiscountedPrice(final int price, final float discountRate) {
        if (price < 0) {
            throw new IllegalArgumentException("価格は0以上を指定してください。");
        }
        if (discountRate < 0.0f || discountRate > 1.0f) {
            throw new IllegalArgumentException("割引率は0.0〜1.0の範囲です。");
        }
        this.amount = (int)(price * (1.0f - discountRate));
    }

    int getAmount() {
        return amount;
    }
}

// ✅ ボーナスポイントを表す専用クラス
class BonusPoint {
    private final int point;

    BonusPoint(final int price, final float pointRate) {
        if (price < 0) {
            throw new IllegalArgumentException("価格は0以上を指定してください。");
        }
        if (pointRate < 0) {
            throw new IllegalArgumentException("ポイント率は0以上を指定してください。");
        }
        this.point = (int)(price * pointRate);
    }

    int getPoint() {
        return point;
    }
}
```

📝 **使い方:**
```java
TaxIncludedPrice taxIncluded = new TaxIncludedPrice(1000, 0.10f);
System.out.println(taxIncluded.getAmount()); // → 1100

DiscountedPrice discounted = new DiscountedPrice(1000, 0.20f);
System.out.println(discounted.getAmount());  // → 800

BonusPoint bonus = new BonusPoint(1000, 0.01f);
System.out.println(bonus.getPoint());        // → 10

// ❌ 不正な値はそもそも作れない
// new TaxIncludedPrice(-100, 0.10f);  // → IllegalArgumentException
```

**Python:**
```python
# ✅ 税込み価格を表す専用クラス
class TaxIncludedPrice:
    def __init__(self, price: int, tax_rate: float) -> None:
        if price < 0:
            raise ValueError("価格は0以上を指定してください。")
        if tax_rate < 0:
            raise ValueError("税率は0以上を指定してください。")
        self._amount: int = int(price * (1.0 + tax_rate))

    @property
    def amount(self) -> int:
        return self._amount


# ✅ 割引価格を表す専用クラス
class DiscountedPrice:
    def __init__(self, price: int, discount_rate: float) -> None:
        if price < 0:
            raise ValueError("価格は0以上を指定してください。")
        if discount_rate < 0.0 or discount_rate > 1.0:
            raise ValueError("割引率は0.0〜1.0の範囲です。")
        self._amount: int = int(price * (1.0 - discount_rate))

    @property
    def amount(self) -> int:
        return self._amount


# ✅ ボーナスポイントを表す専用クラス
class BonusPoint:
    def __init__(self, price: int, point_rate: float) -> None:
        if price < 0:
            raise ValueError("価格は0以上を指定してください。")
        if point_rate < 0:
            raise ValueError("ポイント率は0以上を指定してください。")
        self._point: int = int(price * point_rate)

    @property
    def point(self) -> int:
        return self._point
```

📝 **使い方:**
```python
tax_included = TaxIncludedPrice(1000, 0.10)
print(tax_included.amount)  # → 1100

discounted = DiscountedPrice(1000, 0.20)
print(discounted.amount)    # → 800

bonus = BonusPoint(1000, 0.01)
print(bonus.point)          # → 10

# ❌ 不正な値はそもそも作れない
# TaxIncludedPrice(-100, 0.10)  # → ValueError
```

**TypeScript:**
```typescript
// ✅ 税込み価格を表す専用クラス
class TaxIncludedPrice {
    private readonly amount: number;

    constructor(price: number, taxRate: number) {
        if (price < 0) throw new Error("価格は0以上を指定してください。");
        if (taxRate < 0) throw new Error("税率は0以上を指定してください。");
        this.amount = Math.floor(price * (1 + taxRate));
    }

    getAmount(): number {
        return this.amount;
    }
}

// ✅ 割引価格を表す専用クラス
class DiscountedPrice {
    private readonly amount: number;

    constructor(price: number, discountRate: number) {
        if (price < 0) throw new Error("価格は0以上を指定してください。");
        if (discountRate < 0 || discountRate > 1) {
            throw new Error("割引率は0.0〜1.0の範囲です。");
        }
        this.amount = Math.floor(price * (1 - discountRate));
    }

    getAmount(): number {
        return this.amount;
    }
}

// ✅ ボーナスポイントを表す専用クラス
class BonusPoint {
    private readonly point: number;

    constructor(price: number, pointRate: number) {
        if (price < 0) throw new Error("価格は0以上を指定してください。");
        if (pointRate < 0) throw new Error("ポイント率は0以上を指定してください。");
        this.point = Math.floor(price * pointRate);
    }

    getPoint(): number {
        return this.point;
    }
}
```

📝 **使い方:**
```typescript
const taxIncluded = new TaxIncludedPrice(1000, 0.10);
console.log(taxIncluded.getAmount()); // → 1100

const discounted = new DiscountedPrice(1000, 0.20);
console.log(discounted.getAmount());  // → 800

const bonus = new BonusPoint(1000, 0.01);
console.log(bonus.getPoint());        // → 10

// ❌ 不正な値はそもそも作れない
// new TaxIncludedPrice(-100, 0.10);  // → Error
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| クラス構成 | `Common`（税計算・割引・ポイントを全部詰め込み） | `TaxIncludedPrice` / `DiscountedPrice` / `BonusPoint`（各専用クラス） | **概念ごとにクラスが分離** 。それぞれが自分の責任だけを持つ |
| メソッドの種類 | `static` メソッド（データなし） | コンストラクタ + インスタンスメソッド | **データとロジックが一体化** 。カプセル化が実現 |
| バリデーション | 各 static メソッド内で毎回同じチェック | 各クラスのコンストラクタに **1箇所だけ** | 重複が排除され、ルール変更時の修正箇所が1箇所に |
| 拡張性 | 新しい処理を追加 → `Common` がさらに肥大化 | 新しい概念 → **新しいクラスを追加** するだけ | 既存クラスへの影響なし |

---

### 💡 共通処理クラスが許されるケース（横断的関心事）

すべての共通処理クラスが悪いわけではない。以下のような **横断的関心事** は、共通クラスにまとめてもよい。

| ケース | 例 | 理由 |
|---|---|---|
| ログ出力 | `Logger.info("処理開始")` | **すべての機能で横断的に使われる** 。特定のドメインに属さない |
| エラーハンドリング | `ErrorHandler.handle(e)` | アプリ全体のエラー処理ポリシーを統一するため |
| フォーマット変換 | `DateFormatter.format(date)` | 日付や数値のフォーマットは **アプリ全体で統一** すべき |

> ⚠️ 判断基準は **「そのロジックは特定のドメインオブジェクトに属するか？」** 。属するなら **そのオブジェクトのインスタンスメソッドにすべき** 。属さないなら共通クラスに入れてもよい。

---

### 🎯 まとめ（共通処理クラスにおける考え方）

#### 1. 目的（Why）
> **「関連するロジックを適切なクラスに配置し、低凝集な "なんでも箱" の発生を防ぐため」**

* `Common` / `Util` に雑多な処理を詰め込むと、 **クラスの責任が不明確** になり保守性が低下する。
* 「共通」という名前の安心感が、 **さらなるメソッド追加を誘発** し、際限なく肥大化する。
* データとロジックが分離したまま（static メソッドの誤用）になりやすい。

#### 2. 目標（What）
> **「各概念が専用クラスを持ち、Common / Util クラスが不要な状態」**

* 「税込み価格」「割引価格」「ポイント」がそれぞれ独立したクラスで管理されている。
* 共通クラスには **横断的関心事のみ** が残り、ドメインロジックは含まれていない。

#### 3. 手段（How）
> **「Common の各メソッドを、扱うデータに対応する専用クラスに移動する」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| 税込み計算 | `Common.calcTaxIncludedPrice(price, rate)` | `new TaxIncludedPrice(price, rate)`（専用クラス） |
| 割引計算 | `Common.calcDiscountedPrice(price, rate)` | `new DiscountedPrice(price, rate)`（専用クラス） |
| ポイント計算 | `Common.calcBonusPoint(price, rate)` | `new BonusPoint(price, rate)`（専用クラス） |

> ※ 共通処理クラスが **すべて悪い** わけではない。ログ出力やエラーハンドリングなどの **横断的関心事** は例外的に共通クラスにまとめてよい。迷ったら「このロジックは特定のデータに紐づくか？」と自問し、紐づくなら **そのデータを持つ専用クラスに移動** しよう。
