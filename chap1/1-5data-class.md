## データクラス (Data Class)

**Java:**
```java
public class ContractAmount {
    public int amountIncludingTax;
    public int amountExcludingTax;
    public int tax;
    public int discount;
    public int totalAmount;
}
```

**Python:**
```python
class ContractAmount:
    def __init__(self):
        self.amount_including_tax = 0
        self.amount_excluding_tax = 0
        self.tax = 0
        self.discount = 0
        self.total_amount = 0
```

**TypeScript:**
```typescript
class ContractAmount {
    public amountIncludingTax: number = 0;
    public amountExcludingTax: number = 0;
    public tax: number = 0;
    public discount: number = 0;
    public totalAmount: number = 0;
}
```

---

### 🚨 何がダメなのか？ (データクラスの問題点)

> 💡 **用語: データクラス（Data Class）のアンチパターンとは？**
> **注意：データだけのクラス自体が悪いわけではない。** DTO / Record は正当なパターン。  
> **問題になるのは、** 本来ロジック（計算・検証）を持つべきクラスなのに「ただの入れ物」になっている状態のこと。

<br>

#### 1. 🔴 ロジック（振る舞い）がない → 計算が外にバラまかれる

> 対象: `ContractAmount` クラス全体

| 状態 | 説明 |
|:---:|---|
| ❌ **今のクラス** | フィールドを `public` で並べているだけ。**計算ロジックがゼロ** |
| 😱 **結果** | 「税込金額の計算」がアプリのあちこちに **コピペで散らばる** |

→ 1箇所の計算式を直しても、他の場所に同じ計算が残っていて **修正漏れ** が起きる。

<br>

#### 2. 🔴 フィールドが全て `public` → 誰でも書き換え放題

> 対象: `public int amountIncludingTax` 等

| やりたいこと | 現状 |
|:---:|---|
| 税込金額だけ変更したい | `contract.amountIncludingTax = 9999;` ← **自由にできてしまう** |
| 税や税抜との整合性 | **誰も守ってくれない** → データが壊れる 💥 |

> 💡 **用語: カプセル化（Encapsulation）とは？**
> データとロジックを **1つのクラスに閉じ込め** 、外部から直接触れないようにする考え方。
> `private` で隠し、メソッドだけ公開 → データが勝手に壊されるのを防ぐ。

<br>

#### 3. 🔴 関連データの整合性が保証されない

> 対象: `amountIncludingTax`, `amountExcludingTax`, `tax` の関係

**本来のルール:** `税込 ＝ 税抜 ＋ 税`

でも今のクラスだと…こんな矛盾した値も設定できてしまう 👇

```
税込 = 1000, 税抜 = 500, 税 = 200  ← 計算が合わない！💀
```

> 💡 **用語: 不変条件（Invariant）とは？**
> 「税込 ＝ 税抜 ＋ 税」のように、**常に成り立たなければならないルール** のこと。
> カプセル化で **クラス内部にこのルールを強制** → 外部がルールを破れなくなる。

---

### 🌟 改善版コード（ロジックをクラス内に閉じ込める / カプセル化）

**一言でいうと:** データ + 計算ロジックを **セットで** クラスに閉じ込める。

**Java:**
```java
public class ContractAmount {
    // 🔒 private → 外部から直接書き換え不可
    private int amountExcludingTax;
    private int taxRate;      // 税率（例: 10 = 10%）
    private int discountRate;  // 割引率（例: 5 = 5%）

    // 📦 コンストラクタ（作成時に必要な値を渡す）
    public ContractAmount(int amountExcludingTax, int taxRate, int discountRate) {
        if (amountExcludingTax < 0) throw new IllegalArgumentException("金額は0以上");
        this.amountExcludingTax = amountExcludingTax;
        this.taxRate = taxRate;
        this.discountRate = discountRate;
    }

    // 🧮 税額（自動計算）
    public int tax() {
        return amountExcludingTax * taxRate / 100;
    }

    // 🧮 税込金額 = 税抜 + 税
    public int amountIncludingTax() {
        return amountExcludingTax + tax();
    }

    // 🧮 割引額
    public int discount() {
        return amountIncludingTax() * discountRate / 100;
    }

    // 🧮 最終合計 = 税込 - 割引
    public int totalAmount() {
        return amountIncludingTax() - discount();
    }
}
```

**Python:**
```python
class ContractAmount:
    def __init__(self, amount_excluding_tax: int, tax_rate: int, discount_rate: int):
        if amount_excluding_tax < 0:
            raise ValueError("金額は0以上")
        self._amount_excluding_tax = amount_excluding_tax  # 🔒 _で外部非推奨
        self._tax_rate = tax_rate
        self._discount_rate = discount_rate

    def tax(self) -> int:                    # 🧮 税額
        return self._amount_excluding_tax * self._tax_rate // 100

    def amount_including_tax(self) -> int:    # 🧮 税込金額
        return self._amount_excluding_tax + self.tax()

    def discount(self) -> int:               # 🧮 割引額
        return self.amount_including_tax() * self._discount_rate // 100

    def total_amount(self) -> int:            # 🧮 最終合計
        return self.amount_including_tax() - self.discount()
```

**TypeScript:**
```typescript
class ContractAmount {
    // 🔒 private → 外部から直接書き換え不可
    private amountExcludingTax: number;
    private taxRate: number;
    private discountRate: number;

    constructor(amountExcludingTax: number, taxRate: number, discountRate: number) {
        if (amountExcludingTax < 0) throw new Error("金額は0以上");
        this.amountExcludingTax = amountExcludingTax;
        this.taxRate = taxRate;
        this.discountRate = discountRate;
    }

    tax(): number {                  // 🧮 税額
        return Math.floor(this.amountExcludingTax * this.taxRate / 100);
    }

    amountIncludingTax(): number {   // 🧮 税込金額
        return this.amountExcludingTax + this.tax();
    }

    discount(): number {             // 🧮 割引額
        return Math.floor(this.amountIncludingTax() * this.discountRate / 100);
    }

    totalAmount(): number {          // 🧮 最終合計
        return this.amountIncludingTax() - this.discount();
    }
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| **税額** | `public int tax;`（外から自由に設定） | `public int tax()`（内部で自動計算） |
| **フィールド** | 全て `public`（書き換え放題） | 全て `private`（コンストラクタ経由のみ） |
| **計算ロジック** | クラスの外にバラバラ | **クラスの中に集約** |
| **データの整合性** | 保証なし 💀 | **常に正しい状態を保証** ✨ |

<br>

---

#### 📝 外部からの使い方（使用例）

**作って → 聞くだけ！** 計算式を外に書く必要なし。

**Java:**
```java
// 📦 税抜10,000円、税率10%、割引率5%
ContractAmount contract = new ContractAmount(10000, 10, 5);

contract.tax();              // → 1000（税額）
contract.amountIncludingTax(); // → 11000（税込）
contract.discount();          // → 550（割引）
contract.totalAmount();       // → 10450（最終合計）

// ❌ コンパイルエラー！ private だから触れない
// contract.amountExcludingTax = 99999;
```

**Python:**
```python
# 📦 税抜10,000円、税率10%、割引率5%
contract: ContractAmount = ContractAmount(10000, 10, 5)

contract.tax()                  # → 1000（税額）
contract.amount_including_tax()  # → 11000（税込）
contract.discount()             # → 550（割引）
contract.total_amount()         # → 10450（最終合計）

# ⚠️ Python には厳密な private はない（_ は "触らないで" という慣習）
# contract._amount_excluding_tax = 99999  ← やってはいけない
```

**TypeScript:**
```typescript
// 📦 税抜10,000円、税率10%、割引率5%
const contract: ContractAmount = new ContractAmount(10000, 10, 5);

contract.tax();              // → 1000（税額）
contract.amountIncludingTax(); // → 11000（税込）
contract.discount();          // → 550（割引）
contract.totalAmount();       // → 10450（最終合計）

// ❌ コンパイルエラー！ private だから触れない
// contract.amountExcludingTax = 99999;
```

> 📌 **ポイント:** 外部は「税抜・税率・割引率」を渡してインスタンスを作り、あとは **メソッドに聞くだけ** 。どこで使っても結果がブレない。

---

### 🎯 まとめ（データクラス排除における考え方）

#### 1. 目的（Why）
> **「データの不整合（壊れた状態）を防ぐ」**

* データとロジックが分離 → 外部が勝手にデータを書き換え → 矛盾した値が生まれる
* 同じ計算がコピペで散らばる → 修正漏れが発生する

#### 2. 目標（What）
> **「クラスが常に正しい状態であることを自ら保証する」**

* いつ作られても、どこから使われても、フィールドの値が矛盾しない

#### 3. 手段（How）
> **「データとロジックを同じクラスに閉じ込める（カプセル化）」**

| ステップ | 内容 |
|:---:|---|
| **① `private` にする** | ❌ `public int tax;` → ⭕️ `private int amountExcludingTax;` |
| | *外部から勝手にデータを壊せなくする* |
| **② ロジックをメソッドにする** | ❌ 外部で `tax = amount * 0.1;` → ⭕️ `public int tax() { ... }` |
| | *常にクラス内部のルールで自動計算 → 矛盾が起きない* |
