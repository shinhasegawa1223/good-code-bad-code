## ロジックの散在 (Scattered Business Logic)

**Java:**
```java
public class ContractManager {
    public ContractAmount contractAmount;

    // 税込金額を計算する。
    public int calculateAmountIncludingTax(int amountExcludingTax, BigDecimal salesTaxRate) {
        BigDecimal multiplier = salesTaxRate.add(new BigDecimal("1.0"));
        BigDecimal amountIncludingTax = multiplier.multiply(new BigDecimal(amountExcludingTax));
        return amountIncludingTax.intValue();
    }

    // 契約締結する。
    public void conclude() {
        int amountExcludingTax = 1000;
        BigDecimal salesTaxRate = new BigDecimal("0.10");
        // 省略
        int amountIncludingTax = calculateAmountIncludingTax(amountExcludingTax, salesTaxRate);
        contractAmount = new ContractAmount();
        contractAmount.amountIncludingTax = amountIncludingTax;
        contractAmount.salesTaxRate = salesTaxRate;
        // 省略
    }
}
```

**Python:**
```python
from decimal import Decimal

class ContractManager:
    def __init__(self) -> None:
        self.contract_amount: ContractAmount | None = None

    # 税込金額を計算する
    def calculate_amount_including_tax(self, amount_excluding_tax: int, sales_tax_rate: Decimal) -> int:
        multiplier: Decimal = sales_tax_rate + Decimal("1.0")
        amount_including_tax: Decimal = multiplier * Decimal(amount_excluding_tax)
        return int(amount_including_tax)

    # 契約締結する
    def conclude(self) -> None:
        amount_excluding_tax: int = 1000
        sales_tax_rate: Decimal = Decimal("0.10")
        # 省略
        amount_including_tax: int = self.calculate_amount_including_tax(amount_excluding_tax, sales_tax_rate)
        self.contract_amount = ContractAmount()
        self.contract_amount.amount_including_tax = amount_including_tax
        self.contract_amount.sales_tax_rate = sales_tax_rate
        # 省略
```

**TypeScript:**
```typescript
class ContractManager {
    public contractAmount: ContractAmount | null = null;

    // 税込金額を計算する
    calculateAmountIncludingTax(amountExcludingTax: number, salesTaxRate: number): number {
        const multiplier: number = salesTaxRate + 1.0;
        const amountIncludingTax: number = Math.floor(multiplier * amountExcludingTax);
        return amountIncludingTax;
    }

    // 契約締結する
    conclude(): void {
        const amountExcludingTax: number = 1000;
        const salesTaxRate: number = 0.10;
        // 省略
        const amountIncludingTax: number = this.calculateAmountIncludingTax(amountExcludingTax, salesTaxRate);
        this.contractAmount = new ContractAmount();
        this.contractAmount.amountIncludingTax = amountIncludingTax;
        this.contractAmount.salesTaxRate = salesTaxRate;
        // 省略
    }
}
```

---

### 🚨 何がダメなのか？ (ロジック散在の問題点)

> 💡 **用語: 低凝集（Low Cohesion）とは？**
> 本来1つのクラスにまとまるべきデータとロジックが **バラバラに別のクラスに散らばっている** 状態のこと。
> 凝集度が低い ＝ 関連するものが一箇所にまとまっていない ＝ **保守性・可読性が悪い** 。

<br>

#### 1. 🔴 「金額の計算」が `ContractAmount` ではなく `ContractManager` にある
> 対象: `calculateAmountIncludingTax` メソッド

| クラス | 本来の責任 | 実際 |
|---|---|---|
| `ContractAmount` | 金額データ **＋ その計算ロジック** | データだけ（ただの入れ物）😶 |
| `ContractManager` | 契約の管理・フローの制御 | **税込計算まで肩代わり** している 😱 |

→ 「金額の計算」は `ContractAmount` 自身が知るべき **自分の仕事** 。
  それを `ContractManager` がやっている ＝ **責任の漏洩** 。

> 💡 **用語: Feature Envy（機能の横恋慕）とは？**
> あるクラスのメソッドが、自分のデータよりも **他のクラスのデータばかり使っている** 状態のこと。
> 「そのロジック、あっちのクラスに置くべきでは？」というサイン。

<br>

#### 2. 🔴 `ContractAmount` のフィールドを外部から直接書き換えている
> 対象: `contractAmount.amountIncludingTax = amountIncludingTax;`

```java
contractAmount.amountIncludingTax = amountIncludingTax;  // 😱 直接代入！
contractAmount.salesTaxRate = salesTaxRate;               // 😱 これも直接！
```

| 問題 | 説明 |
|:---:|---|
| **整合性なし** | 税込金額と税率を別々に設定 → 矛盾してもエラーにならない |
| **散らばる** | 他のクラスでも同じように直接設定できてしまう → 修正漏れの温床 |

<br>

#### 3. 🔴 同じ計算が他のクラスにも重複する（DRY原則の違反）

> 💡 **用語: DRY原則（Don't Repeat Yourself）とは？**
> 「同じ知識（ロジック）を2箇所以上に書くな」という原則。
> 重複すると、片方を直してもう片方を直し忘れ → **バグが生まれる** 。

`ContractManager` に税込計算があるなら、他のクラス（`InvoiceManager`, `PaymentProcessor` 等）にも **同じような計算がコピペ** される可能性が高い。

→ 1箇所を直しても **他の場所に修正漏れ** が出る 💀

---

### 🌟 改善版コード（ロジックを本来のクラスに戻す / 高凝集にする）

**一言でいうと:** 「税込金額の計算」は `ContractAmount` 自身に持たせ、`ContractManager` は **依頼するだけ** にする。

> 💡 **用語: 高凝集（High Cohesion）とは？**
> 関連するデータとロジックが **1つのクラスにきちんとまとまっている** 状態。
> データを持つクラスが自分で計算もする → 外部は結果を **聞くだけ** で良い。

**Java:**
```java
// ✅ ContractAmount がロジックを持つ（1-5 の改善版と同じ考え方）
public class ContractAmount {
    private int amountExcludingTax;
    private BigDecimal salesTaxRate;

    public ContractAmount(int amountExcludingTax, BigDecimal salesTaxRate) {
        if (amountExcludingTax < 0) throw new IllegalArgumentException("金額は0以上");
        this.amountExcludingTax = amountExcludingTax;
        this.salesTaxRate = salesTaxRate;
    }

    // 🧮 税込金額の計算は ContractAmount 自身が行う
    public int amountIncludingTax() {
        BigDecimal multiplier = salesTaxRate.add(new BigDecimal("1.0"));
        return multiplier.multiply(new BigDecimal(amountExcludingTax)).intValue();
    }
}

// ✅ ContractManager は「契約フローの管理」だけに専念する
public class ContractManager {
    public void conclude() {
        // 📦 ContractAmount を作るだけ。計算は ContractAmount に任せる
        ContractAmount contractAmount = new ContractAmount(1000, new BigDecimal("0.10"));

        // メソッドに聞くだけで正しい税込金額が得られる
        int tax = contractAmount.amountIncludingTax();  // → 1100
        // 省略
    }
}
```

**Python:**
```python
from decimal import Decimal

# ✅ ContractAmount がロジックを持つ
class ContractAmount:
    def __init__(self, amount_excluding_tax: int, sales_tax_rate: Decimal) -> None:
        if amount_excluding_tax < 0:
            raise ValueError("金額は0以上")
        self._amount_excluding_tax = amount_excluding_tax
        self._sales_tax_rate = sales_tax_rate

    # 🧮 税込金額の計算は ContractAmount 自身が行う
    def amount_including_tax(self) -> int:
        multiplier: Decimal = self._sales_tax_rate + Decimal("1.0")
        return int(multiplier * Decimal(self._amount_excluding_tax))

# ✅ ContractManager は「契約フローの管理」だけに専念する
class ContractManager:
    def conclude(self) -> None:
        # 📦 作るだけ。計算は ContractAmount に任せる
        contract_amount: ContractAmount = ContractAmount(1000, Decimal("0.10"))

        tax: int = contract_amount.amount_including_tax()  # → 1100
        # 省略
```

**TypeScript:**
```typescript
// ✅ ContractAmount がロジックを持つ
class ContractAmount {
    private amountExcludingTax: number;
    private salesTaxRate: number;

    constructor(amountExcludingTax: number, salesTaxRate: number) {
        if (amountExcludingTax < 0) throw new Error("金額は0以上");
        this.amountExcludingTax = amountExcludingTax;
        this.salesTaxRate = salesTaxRate;
    }

    // 🧮 税込金額の計算は ContractAmount 自身が行う
    amountIncludingTax(): number {
        return Math.floor((this.salesTaxRate + 1.0) * this.amountExcludingTax);
    }
}

// ✅ ContractManager は「契約フローの管理」だけに専念する
class ContractManager {
    conclude(): void {
        // 📦 作るだけ。計算は ContractAmount に任せる
        const contractAmount: ContractAmount = new ContractAmount(1000, 0.10);

        const tax: number = contractAmount.amountIncludingTax();  // → 1100
        // 省略
    }
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| **税込計算の場所** | `ContractManager` が計算 😱 | `ContractAmount` **自身** が計算 ✨ |
| **ContractManagerの役割** | 管理 + 計算（責任過多） | **管理だけ** に専念 |
| **フィールドへの代入** | `contractAmount.amountIncludingTax = ...` （直接書き換え） | コンストラクタで渡すだけ（`private` で保護）|
| **他のクラスで同じ計算** | コピペが広がる 💀 | `ContractAmount` に聞くだけ → **重複なし** |

---

### 🎯 まとめ（ロジック散在の排除における考え方）

#### 1. 目的（Why）
> **「変更時の修正漏れと責任の混乱を防ぐ」**

* ロジックが散らばっている → 仕様変更時に **全箇所を直す必要** → 修正漏れ → バグ
* 「どこを直せばいいのか」が **分からなくなる**

#### 2. 目標（What）
> **「データを持つクラスが、そのデータに関するロジックも持つ（高凝集）」**

* 税額の計算 → `ContractAmount` に聞く
* 契約の管理 → `ContractManager` に聞く
* 各クラスが **自分の責任だけ** を果たす

#### 3. 手段（How）
> **「ロジックを本来あるべきクラスに移動する」**

| ステップ | 内容 |
|:---:|---|
| **① 「このロジックは誰のデータを使っている？」** | `calculateAmountIncludingTax` は `ContractAmount` のデータを使っている → **そっちに移すべき** |
| **② データを持つクラスにメソッドを移動** | `ContractManager.calculateAmountIncludingTax` → `ContractAmount.amountIncludingTax` |
| **③ 呼び出し元は「聞くだけ」にする** | `ContractManager` は `contractAmount.amountIncludingTax()` を呼ぶだけ |
