## 未初期化・不正値の混入 (Uninitialized & Invalid State)

**Java:**
```java
// 😱 未初期化のまま使用 → NullPointerException
ContractAmount contractAmount = new ContractAmount();
System.out.println(contractAmount.salesTaxRate.toString());  // 💥 NullPointerException!
```

```java
// 😱 不正な値を自由に設定できてしまう
ContractAmount contractAmount = new ContractAmount();
contractAmount.salesTaxRate = new BigDecimal("-0.10");  // マイナスの税率!? 💀
```

**Python:**
```python
# 😱 未初期化のまま使用 → AttributeError
contract_amount: ContractAmount = ContractAmount()
print(contract_amount.sales_tax_rate)  # 💥 None or AttributeError!
```

```python
# 😱 不正な値を自由に設定できてしまう
contract_amount: ContractAmount = ContractAmount()
contract_amount.sales_tax_rate = Decimal("-0.10")  # マイナスの税率!? 💀
```

**TypeScript:**
```typescript
// 😱 未初期化のまま使用 → undefined
const contractAmount: ContractAmount = new ContractAmount();
console.log(contractAmount.salesTaxRate!.toString());  // 💥 TypeError!
```

```typescript
// 😱 不正な値を自由に設定できてしまう
const contractAmount: ContractAmount = new ContractAmount();
contractAmount.salesTaxRate = -0.10;  // マイナスの税率!? 💀
```

---

### 🚨 何がダメなのか？ (未初期化・不正値の問題点)

> 💡 **用語: 生成時の不変条件（Construction Invariant）とは？**
> オブジェクトが **作られた時点で** 必ず正しい状態であることを保証するルール。
> コンストラクタで検証を行い、不正な状態のオブジェクトが **そもそも作られない** ようにする。

<br>

#### 1. 🔴 未初期化状態（NullPointerException の温床）

> 対象: `new ContractAmount()` 直後の `salesTaxRate`

| 状態 | 何が起きるか |
|:---:|---|
| `new ContractAmount()` 直後 | `salesTaxRate` は **`null`（未設定）** |
| `salesTaxRate.toString()` を呼ぶ | 💥 **NullPointerException** でクラッシュ |

→ **利用する側が「このフィールドは初期化済みか？」を常に気にしなければならない** 。

これは使う側の責任ではなく **クラスの設計の問題** 。

<br>

#### 2. 🔴 不正値の混入（ビジネスルール違反）

> 対象: `salesTaxRate = new BigDecimal("-0.10")`

| 設定した値 | ビジネス的に正しい？ |
|:---:|---|
| `-0.10`（マイナス10%の税率） | ❌ **ありえない**。税率は0%以上のはず |
| `999.99`（99999%の税率） | ❌ **ありえない**。でも設定できてしまう |

→ `public` フィールドに対して **何の検証もなく** 値を代入できるため、**ビジネス的にあり得ない値** が簡単に入り込む。

<br>

#### 3. 🔴 重複コードの発生

> 💡 **用語: DRY原則（Don't Repeat Yourself）とは？**
> 同じロジック（検証・計算）を **2箇所以上に書かない** という原則。
> 重複 → 片方を直し忘れ → **バグの温床** 。

「税率がマイナスでないかチェックする」コードを書きたくなるが、
クラスにバリデーションがないため **使う側がそれぞれでチェック** する必要がある。

```java
// 😱 使う場所ごとに同じチェックをコピペ…
if (salesTaxRate.compareTo(BigDecimal.ZERO) < 0) {
    throw new IllegalArgumentException("税率は0以上");
}
```

→ チェックが10箇所に散らばる → 1箇所直し忘れ → **バグ** 💀

---

### 🌟 改善版コード（コンストラクタで完全な状態を保証する）

**一言でいうと:** オブジェクト生成時に **必要な値を全て渡し + 不正値を弾く** → 「壊れた状態」が **そもそも作れない** 。

> 💡 **用語: Complete Constructor（完全コンストラクタ）とは？**
> オブジェクトの生成時に **全ての必須値をコンストラクタで受け取り** 、不正な値はその場で拒否するパターン。
> これにより、「初期化忘れ」も「不正値の混入」も **原理的に発生しなくなる** 。

**Java:**
```java
public class ContractAmount {
    private int amountExcludingTax;
    private BigDecimal salesTaxRate;

    // 📦 コンストラクタで全ての値を受け取り + バリデーション
    public ContractAmount(int amountExcludingTax, BigDecimal salesTaxRate) {
        // ✅ 不正値を即座に拒否
        if (amountExcludingTax < 0) {
            throw new IllegalArgumentException("金額は0以上");
        }
        if (salesTaxRate.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("税率は0以上");
        }
        this.amountExcludingTax = amountExcludingTax;
        this.salesTaxRate = salesTaxRate;
    }

    // 🧮 税込金額（常に正しいデータで計算される）
    public int amountIncludingTax() {
        BigDecimal multiplier = salesTaxRate.add(new BigDecimal("1.0"));
        return multiplier.multiply(new BigDecimal(amountExcludingTax)).intValue();
    }
}
```

**Python:**
```python
from decimal import Decimal

class ContractAmount:
    # 📦 コンストラクタで全ての値を受け取り + バリデーション
    def __init__(self, amount_excluding_tax: int, sales_tax_rate: Decimal) -> None:
        # ✅ 不正値を即座に拒否
        if amount_excluding_tax < 0:
            raise ValueError("金額は0以上")
        if sales_tax_rate < Decimal("0"):
            raise ValueError("税率は0以上")
        self._amount_excluding_tax = amount_excluding_tax
        self._sales_tax_rate = sales_tax_rate

    # 🧮 税込金額（常に正しいデータで計算される）
    def amount_including_tax(self) -> int:
        multiplier: Decimal = self._sales_tax_rate + Decimal("1.0")
        return int(multiplier * Decimal(self._amount_excluding_tax))
```

**TypeScript:**
```typescript
class ContractAmount {
    private amountExcludingTax: number;
    private salesTaxRate: number;

    // 📦 コンストラクタで全ての値を受け取り + バリデーション
    constructor(amountExcludingTax: number, salesTaxRate: number) {
        // ✅ 不正値を即座に拒否
        if (amountExcludingTax < 0) {
            throw new Error("金額は0以上");
        }
        if (salesTaxRate < 0) {
            throw new Error("税率は0以上");
        }
        this.amountExcludingTax = amountExcludingTax;
        this.salesTaxRate = salesTaxRate;
    }

    // 🧮 税込金額（常に正しいデータで計算される）
    amountIncludingTax(): number {
        return Math.floor((this.salesTaxRate + 1.0) * this.amountExcludingTax);
    }
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| **生成直後の状態** | `null` / 未初期化 💥 | **全フィールドが確実に初期化済み** |
| **不正値** | `-0.10` が入れられる 💀 | コンストラクタで **即エラー** → 入り込めない |
| **バリデーションの場所** | 使う側がそれぞれチェック（重複） | **コンストラクタ1箇所だけ** → DRY ✨ |
| **利用側の知識** | 「初期化済みか？」「不正値でないか？」を常に気にする | **気にしなくてよい**（保証済み） |

<br>

#### 📝 外部からの使い方（使用例）

**作った瞬間に正しい状態が保証される。** 使う側は安心してメソッドを呼ぶだけ。

**Java:**
```java
// ✅ 正常：税抜1,000円、税率10%
ContractAmount contract = new ContractAmount(1000, new BigDecimal("0.10"));
contract.amountIncludingTax();  // → 1100 ✨

// ❌ 不正値は即エラー！ → 壊れたオブジェクトが作れない
new ContractAmount(-500, new BigDecimal("0.10"));   // 💥 IllegalArgumentException!
new ContractAmount(1000, new BigDecimal("-0.10"));  // 💥 IllegalArgumentException!
```

**Python:**
```python
# ✅ 正常：税抜1,000円、税率10%
contract: ContractAmount = ContractAmount(1000, Decimal("0.10"))
contract.amount_including_tax()  # → 1100 ✨

# ❌ 不正値は即エラー！
ContractAmount(-500, Decimal("0.10"))   # 💥 ValueError!
ContractAmount(1000, Decimal("-0.10"))  # 💥 ValueError!
```

**TypeScript:**
```typescript
// ✅ 正常：税抜1,000円、税率10%
const contract: ContractAmount = new ContractAmount(1000, 0.10);
contract.amountIncludingTax();  // → 1100 ✨

// ❌ 不正値は即エラー！
new ContractAmount(-500, 0.10);   // 💥 Error!
new ContractAmount(1000, -0.10);  // 💥 Error!
```

---

### 🎯 まとめ（未初期化・不正値の排除における考え方）

#### 1. 目的（Why）
> **「壊れたオブジェクトが存在できない状態にする」**

* 未初期化 → NullPointerException → **本番クラッシュ**
* 不正値 → ビジネスルール違反 → **誤った計算結果** → 顧客への影響

#### 2. 目標（What）
> **「オブジェクトが作られた瞬間から、常に正しい状態であること」**

* `new` した直後 → 全フィールドが意味のある正しい値
* `null` や不正値が **入り込む隙がない**

#### 3. 手段（How）
> **「Complete Constructor（完全コンストラクタ）パターン」**

| ステップ | 内容 |
|:---:|---|
| **① 引数なしコンストラクタを廃止** | ❌ `new ContractAmount()` → ⭕️ `new ContractAmount(1000, 0.10)` |
| **② コンストラクタでバリデーション** | 不正値は即 `throw` → 壊れたオブジェクトが **作れない** |
| **③ フィールドを `private` に** | 外部からの直接書き換えを禁止 → 初期化後も安全 |
