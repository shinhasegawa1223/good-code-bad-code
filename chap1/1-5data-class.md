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

記載されているコードには、以下の問題があります。

#### 1. データを持つだけでロジック（振る舞い）がない
> 対象: `ContractAmount` クラス全体

このクラスはフィールド（変数）を `public` で並べているだけで、**「税込金額をどう計算するか」「割引をどう適用するか」** といったロジック（メソッド）が一切ありません。
結果として「計算ロジック」が **このクラスの外のあちこちにバラバラに散らばってしまい** 、同じような計算が複数箇所に重複する（DRY原則の違反）原因になります。

#### 2. フィールドが全て `public` で外部から自由に書き換えられる
> 対象: `public int amountIncludingTax` 等のフィールド

フィールドが全て `public` のため、アプリのどこからでも値を直接書き換え可能な状態です。
例えば、外部のコードが `amountIncludingTax` だけを直接変更し、`tax` や `amountExcludingTax` との整合性が崩れても、それを防ぐ仕組みがありません。**データの不整合（壊れた状態）** を許してしまいます。

#### 3. 関連するデータ同士の整合性が保証されない
> 対象: `amountIncludingTax`, `amountExcludingTax`, `tax` の関係

「税込金額 ＝ 税抜金額 ＋ 税」という **ビジネスルール（不変条件）** が存在するはずですが、このクラスにはそのルールを強制する仕組みがないため、`amountIncludingTax = 1000`, `amountExcludingTax = 500`, `tax = 200` のような矛盾した値が設定できてしまいます。

---

### 🌟 改善版コード（ロジックをクラス内に閉じ込める / カプセル化）

データを持つだけでなく、**そのデータに関連するロジック（計算・検証）もクラスの中に閉じ込める（カプセル化）** ことで、外部からの不正な変更を防ぎ、データの整合性を保証します。

**Java:**
```java
public class ContractAmount {
    // フィールドを private にして外部から直接書き換えられないようにする
    private int amountExcludingTax;
    private int taxRate;     // 税率（例: 10 = 10%）
    private int discountRate; // 割引率（例: 5 = 5%）

    // コンストラクタで必要な値を受け取り、内部で計算する
    public ContractAmount(int amountExcludingTax, int taxRate, int discountRate) {
        if (amountExcludingTax < 0) throw new IllegalArgumentException("金額は0以上");
        this.amountExcludingTax = amountExcludingTax;
        this.taxRate = taxRate;
        this.discountRate = discountRate;
    }

    // 税額を計算するロジックをクラス内に持たせる
    public int tax() {
        return amountExcludingTax * taxRate / 100;
    }

    // 税込金額 = 税抜 + 税（ビジネスルールをコードで保証する）
    public int amountIncludingTax() {
        return amountExcludingTax + tax();
    }

    // 割引額
    public int discount() {
        return amountIncludingTax() * discountRate / 100;
    }

    // 最終合計金額 = 税込 - 割引
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
        self._amount_excluding_tax = amount_excluding_tax
        self._tax_rate = tax_rate
        self._discount_rate = discount_rate

    # 税額を計算するロジックをクラス内に持たせる
    def tax(self) -> int:
        return self._amount_excluding_tax * self._tax_rate // 100

    # 税込金額 = 税抜 + 税（ビジネスルールをコードで保証する）
    def amount_including_tax(self) -> int:
        return self._amount_excluding_tax + self.tax()

    # 割引額
    def discount(self) -> int:
        return self.amount_including_tax() * self._discount_rate // 100

    # 最終合計金額 = 税込 - 割引
    def total_amount(self) -> int:
        return self.amount_including_tax() - self.discount()
```

**TypeScript:**
```typescript
class ContractAmount {
    // フィールドを private にして外部から直接書き換えられないようにする
    private amountExcludingTax: number;
    private taxRate: number;
    private discountRate: number;

    constructor(amountExcludingTax: number, taxRate: number, discountRate: number) {
        if (amountExcludingTax < 0) throw new Error("金額は0以上");
        this.amountExcludingTax = amountExcludingTax;
        this.taxRate = taxRate;
        this.discountRate = discountRate;
    }

    // 税額を計算するロジックをクラス内に持たせる
    tax(): number {
        return Math.floor(this.amountExcludingTax * this.taxRate / 100);
    }

    // 税込金額 = 税抜 + 税（ビジネスルールをコードで保証する）
    amountIncludingTax(): number {
        return this.amountExcludingTax + this.tax();
    }

    // 割引額
    discount(): number {
        return Math.floor(this.amountIncludingTax() * this.discountRate / 100);
    }

    // 最終合計金額 = 税込 - 割引
    totalAmount(): number {
        return this.amountIncludingTax() - this.discount();
    }
}
```

#### 改善のポイント
- `public int tax` → **`public int tax()`**: 税額をフィールドとして「外から設定する値」にするのではなく、メソッドとして「税抜金額から自動的に計算される値」に変更。これにより、税抜金額と税額の矛盾が原理的に発生しなくなります。
- `public` → **`private`**: フィールドを `private` にすることで、外部のコードが勝手に値を書き換えることが不可能になりました。値を変えたければ、クラスが提供する正規のルート（コンストラクタ）を通すしかありません。
- ロジックの集約: 「税込 ＝ 税抜 ＋ 税」「合計 ＝ 税込 − 割引」というビジネスルールがクラスの中に閉じ込められたため、アプリ内のどこでこのクラスを使っても **常に正しい計算結果が保証されます** 。

#### 📝 外部からの使い方（使用例）

改善版クラスは、インスタンスを作成して **メソッドを呼ぶだけ** で正しい計算結果が得られます。
外部のコードが計算式を知る必要はなく、**クラスに「聞く」だけ** で良いのがポイントです。

**Java:**
```java
// 税抜10,000円、税率10%、割引率5% で契約金額を作成
ContractAmount contract = new ContractAmount(10000, 10, 5);

// メソッドを呼ぶだけで、正しい計算結果が返ってくる
System.out.println(contract.tax());              // → 1000（税額）
System.out.println(contract.amountIncludingTax()); // → 11000（税込金額）
System.out.println(contract.discount());          // → 550（割引額）
System.out.println(contract.totalAmount());       // → 10450（最終合計）

// ❌ 以下はコンパイルエラー！ private なので外部から直接書き換えられない
// contract.amountExcludingTax = 99999;  // エラー: privateフィールドにアクセスできない
```

**Python:**
```python
# 税抜10,000円、税率10%、割引率5% で契約金額を作成
contract: ContractAmount = ContractAmount(10000, 10, 5)

# メソッドを呼ぶだけで、正しい計算結果が返ってくる
print(contract.tax())                # → 1000（税額）
print(contract.amount_including_tax())  # → 11000（税込金額）
print(contract.discount())           # → 550（割引額）
print(contract.total_amount())       # → 10450（最終合計）

# ❌ 以下は慣習的にNG！ _（アンダースコア）付きは「外部から触らないでください」の意味
# contract._amount_excluding_tax = 99999  # やってはいけない
```

**TypeScript:**
```typescript
// 税抜10,000円、税率10%、割引率5% で契約金額を作成
const contract: ContractAmount = new ContractAmount(10000, 10, 5);

// メソッドを呼ぶだけで、正しい計算結果が返ってくる
console.log(contract.tax());              // → 1000（税額）
console.log(contract.amountIncludingTax()); // → 11000（税込金額）
console.log(contract.discount());          // → 550（割引額）
console.log(contract.totalAmount());       // → 10450（最終合計）

// ❌ 以下はコンパイルエラー！ private なので外部から直接書き換えられない
// contract.amountExcludingTax = 99999;  // エラー: privateプロパティにアクセスできない
```

> **使い方のまとめ:** 外部のコードは「税抜金額・税率・割引率」をコンストラクタに渡してインスタンスを作り、あとは `tax()` や `totalAmount()` などの **メソッドに聞く** だけ。計算式を外に書く必要が一切なくなり、常にクラス内部の正しいルールで計算されるため、どこで使っても結果がブレません。

---

### 🎯 まとめ（データクラス排除における考え方）

データだけのクラスから脱却し、正しいコードを書くための考え方を「目的・目標・手段」の順に整理します。

#### 1. 目的（Why: なぜこれにこだわるのか？）
* **「データの不整合（壊れた状態）を防ぐため」**
  * データとロジックが分離していると、外部のコードが自由にデータを書き換えてしまい、本来あり得ない矛盾した状態（税込金額 ≠ 税抜 + 税）を生み出してしまうため。
  * 同じ計算が色々な場所にコピペされ、修正漏れが発生するため。

#### 2. 目標（What: どのような状態を目指すのか？）
* **「クラスが常に正しい状態であることを自ら保証する」**
  * どの場面でインスタンスが作られても、どこから参照されても、クラスの各フィールドの値が矛盾を起こさない状態を目指します。

#### 3. 手段（How: どうやって達成するのか？）
* **「データとロジックを同じクラスに閉じ込める（カプセル化）」**

  1. **フィールドを `private` にする**
     * ❌ `public int tax;`（外部から自由に書き換え可能）
     * ⭕️ `private int amountExcludingTax;`（外部からのアクセスを禁止）
     * *→ 外部のコードが勝手にデータを壊せないようにします。*

  2. **計算ロジックをメソッドとしてクラス内に持たせる**
     * ❌ 外部で `tax = amount * 0.1;` と手動計算
     * ⭕️ `public int tax() { return amountExcludingTax * taxRate / 100; }`
     * *→ 常にクラス内部の正しいルールで自動計算されるため、データの矛盾が起きません。*
