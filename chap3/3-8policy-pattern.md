## ポリシーパターン（Policy Pattern）

> 💡 **用語: ポリシーパターン（Policy Pattern）とは？**
> 複数の **条件判定（ルール）** をそれぞれ独立したクラスに切り出し、 **自由に組み合わせられる** ようにする設計パターン。ストラテジパターンが「処理の切り替え」なのに対し、ポリシーパターンは **「条件の組み合わせ」** に焦点を置く。

### ❌ 悪い例（条件判定が1メソッドに詰め込まれている）

**Java:**
```java
// ❌ 購入可能かどうかの条件が全部1メソッドに詰め込まれている
class PurchaseChecker {
    boolean canPurchase(int age, int totalPurchase, boolean isGoldMember) {
        // 条件が増えるたびにこのメソッドが肥大化する
        if (age < 18) return false;                           // 年齢制限
        if (totalPurchase < 0) return false;                  // 購入金額チェック
        if (totalPurchase > 100000 && !isGoldMember) return false; // 高額購入はゴールド会員のみ
        return true;
    }
}
```

**Python:**
```python
# ❌ 条件が全部1メソッドに詰め込まれている
class PurchaseChecker:
    def can_purchase(self, age: int, total_purchase: int, is_gold_member: bool) -> bool:
        if age < 18:
            return False
        if total_purchase < 0:
            return False
        if total_purchase > 100000 and not is_gold_member:
            return False
        return True
```

**TypeScript:**
```typescript
// ❌ 条件が全部1メソッドに詰め込まれている
class PurchaseChecker {
    canPurchase(age: number, totalPurchase: number, isGoldMember: boolean): boolean {
        if (age < 18) return false;
        if (totalPurchase < 0) return false;
        if (totalPurchase > 100000 && !isGoldMember) return false;
        return true;
    }
}
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 条件の追加・変更が困難 | 新しいルール（「未成年は保護者の同意が必要」等）を追加するたびに **既存メソッドを修正** しなければならない |
| 2 | 🔴 条件の組み合わせが固定 | 「通常会員向け」と「ゴールド会員向け」でルールを変えたい場合、 **別のメソッドをまるごと作り直す** ことになる |
| 3 | 🔴 個別テストが困難 | 各条件を **独立してテストできない** 。全条件を組み合わせたテストが必要になる |

---

### 🌟 改善版コード

**一言でいうと:** 各条件を **独立したクラス（ポリシー）** にして、使う側で自由に組み合わせる。

**Java:**
```java
// ✅ 各条件を独立したポリシークラスにする
interface PurchasePolicy {
    boolean isAllowed(PurchaseContext context);
}

// 年齢制限ポリシー
class AgePolicy implements PurchasePolicy {
    public boolean isAllowed(PurchaseContext context) {
        return context.getAge() >= 18;
    }
}

// 購入金額チェックポリシー
class AmountPolicy implements PurchasePolicy {
    public boolean isAllowed(PurchaseContext context) {
        return context.getTotalPurchase() >= 0;
    }
}

// 高額購入制限ポリシー
class GoldMemberPolicy implements PurchasePolicy {
    public boolean isAllowed(PurchaseContext context) {
        if (context.getTotalPurchase() > 100000) {
            return context.isGoldMember();
        }
        return true;
    }
}

// ✅ ポリシーを自由に組み合わせるクラス
class PurchaseChecker {
    private final List<PurchasePolicy> policies;

    PurchaseChecker(final List<PurchasePolicy> policies) {
        this.policies = policies;
    }

    boolean canPurchase(PurchaseContext context) {
        // 全てのポリシーがOKなら購入可能
        return policies.stream().allMatch(p -> p.isAllowed(context));
    }
}
```

📝 **使い方:**
```java
// 通常会員向け：全ルール適用
PurchaseChecker normalChecker = new PurchaseChecker(List.of(
    new AgePolicy(),
    new AmountPolicy(),
    new GoldMemberPolicy()
));

// ゴールド会員向け：高額制限なし（ポリシーの組み合わせを変えるだけ！）
PurchaseChecker goldChecker = new PurchaseChecker(List.of(
    new AgePolicy(),
    new AmountPolicy()
    // GoldMemberPolicy を外すだけ！既存コード変更なし
));

// 🎉 新しいルール追加 → クラスを1つ作ってリストに追加するだけ！
class RegionPolicy implements PurchasePolicy {
    public boolean isAllowed(PurchaseContext ctx) {
        return !ctx.getRegion().equals("制限地域");
    }
}
```

**Python:**
```python
from abc import ABC, abstractmethod

class PurchasePolicy(ABC):
    @abstractmethod
    def is_allowed(self, context: 'PurchaseContext') -> bool:
        pass

class AgePolicy(PurchasePolicy):
    def is_allowed(self, context: 'PurchaseContext') -> bool:
        return context.age >= 18

class AmountPolicy(PurchasePolicy):
    def is_allowed(self, context: 'PurchaseContext') -> bool:
        return context.total_purchase >= 0

class GoldMemberPolicy(PurchasePolicy):
    def is_allowed(self, context: 'PurchaseContext') -> bool:
        if context.total_purchase > 100000:
            return context.is_gold_member
        return True

class PurchaseChecker:
    def __init__(self, policies: list[PurchasePolicy]) -> None:
        self._policies: list[PurchasePolicy] = policies

    def can_purchase(self, context: 'PurchaseContext') -> bool:
        return all(p.is_allowed(context) for p in self._policies)
```

**TypeScript:**
```typescript
interface PurchasePolicy {
    isAllowed(context: PurchaseContext): boolean;
}

class AgePolicy implements PurchasePolicy {
    isAllowed(context: PurchaseContext): boolean { return context.age >= 18; }
}

class AmountPolicy implements PurchasePolicy {
    isAllowed(context: PurchaseContext): boolean { return context.totalPurchase >= 0; }
}

class GoldMemberPolicy implements PurchasePolicy {
    isAllowed(context: PurchaseContext): boolean {
        return context.totalPurchase <= 100000 || context.isGoldMember;
    }
}

class PurchaseChecker {
    constructor(private readonly policies: PurchasePolicy[]) {}

    canPurchase(context: PurchaseContext): boolean {
        return this.policies.every(p => p.isAllowed(context));
    }
}
```

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| 新しいルール追加 | 既存メソッドを修正 | ポリシークラスを1つ追加してリストに入れるだけ |
| ルールの組み合わせ | 固定（変えるならメソッドを書き直し） | リストの中身を入れ替えるだけで自由に組み合わせ可能 |
| 個別テスト | 全条件を一括テスト | 各ポリシーを独立してテスト可能 |

> 💡 **ストラテジパターンとの違い:**
> ストラテジパターン = 処理を **1つ選んで切り替える** （配送方法A or B or C）
> ポリシーパターン = 条件を **複数組み合わせる** （ルールA **かつ** ルールB **かつ** ルールC）

<br>
