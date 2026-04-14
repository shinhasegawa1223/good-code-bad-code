## static メソッドの誤用 ― カプセル化を破壊する手続き型の罠

> 💡 **用語: static メソッド（Static Method）とは？**
> クラスの **インスタンスを作らずに** 直接呼び出せるメソッド。`Math.max(a, b)` のように、クラス名から直接使える。便利だが、多用すると **データとロジックが分離** してしまう。

> 💡 **用語: インスタンスメソッド（Instance Method）とは？**
> クラスの **インスタンス（実体）を作ってから** 呼び出すメソッド。`static` が付いていないメソッドのこと。インスタンス変数（そのオブジェクト固有のデータ）にアクセスできるため、 **データとロジックを一体化** できる。

```java
// static メソッド → インスタンス不要で呼べる
int result = Math.max(3, 5);               // Math クラスから直接呼び出し

// インスタンスメソッド → インスタンスを作ってから呼ぶ
List<String> list = new ArrayList<>();      // インスタンスを生成
list.add("hello");                          // インスタンスに対してメソッドを呼ぶ
```

### ❌ 悪い例（static メソッドでロジックを書いてしまう）

**Java:**
```java
// ❌ 金額データだけを持つクラス（ロジックがない = データクラス）
class MoneyData {
    int amount;
}

// ❌ ロジックだけを持つクラス（データがない）
//    static メソッドなのでインスタンス変数を使えない
class OrderManager {
    static int add(int moneyAmount1, int moneyAmount2) {
        return moneyAmount1 + moneyAmount2;
    }
}
```

📝 **使い方:**
```java
MoneyData money1 = new MoneyData();
money1.amount = 1000;
MoneyData money2 = new MoneyData();
money2.amount = 500;

// ❌ データ(MoneyData)とロジック(OrderManager)が別々のクラス
int total = OrderManager.add(money1.amount, money2.amount);
```

**Python:**
```python
# ❌ 金額データだけを持つクラス（ロジックがない）
class MoneyData:
    def __init__(self) -> None:
        self.amount: int = 0

# ❌ ロジックだけを持つクラス（データがない）
class OrderManager:
    @staticmethod
    def add(money_amount1: int, money_amount2: int) -> int:
        return money_amount1 + money_amount2
```

📝 **使い方:**
```python
money1 = MoneyData()
money1.amount = 1000
money2 = MoneyData()
money2.amount = 500

# ❌ データ(MoneyData)とロジック(OrderManager)が別々のクラス
total = OrderManager.add(money1.amount, money2.amount)
```

**TypeScript:**
```typescript
// ❌ 金額データだけを持つクラス（ロジックがない）
class MoneyData {
    amount: number = 0;
}

// ❌ ロジックだけを持つクラス（データがない）
class OrderManager {
    static add(moneyAmount1: number, moneyAmount2: number): number {
        return moneyAmount1 + moneyAmount2;
    }
}
```

📝 **使い方:**
```typescript
const money1 = new MoneyData();
money1.amount = 1000;
const money2 = new MoneyData();
money2.amount = 500;

// ❌ データ(MoneyData)とロジック(OrderManager)が別々のクラス
const total = OrderManager.add(money1.amount, money2.amount);
```

---

### 🚨 何がダメなのか？ (static メソッド誤用の問題点)

> 💡 **用語: データクラス（Data Class）とは？**
> データ（フィールド）だけを持ち、 **ロジック（メソッド）を持たない** クラスのこと。上の `MoneyData` のように、値を入れる箱でしかなく、自分自身を守るバリデーションや計算ができない。

<br>

#### 1. 🔴 データとロジックが別のクラスに分離している（低凝集）
> 対象: `MoneyData`（データのみ） と `OrderManager`（ロジックのみ）

| 名前 | 問題 |
|---|---|
| データとロジックの分離 | 「金額」のデータは `MoneyData` に、「金額を足す」ロジックは `OrderManager` に分かれている。 **関連するものが1箇所にまとまっていない（低凝集）** ため、金額に関する修正をするとき複数クラスを変更する必要がある |

<br>

#### 2. 🔴 カプセル化ができていない
> 対象: `MoneyData` クラスの `amount` フィールド

| 名前 | 問題 |
|---|---|
| `amount` が丸見え | `MoneyData.amount` は **外部から自由に読み書きできる** 。負の金額を入れてもエラーにならない。データクラスには自分自身を守る力がない |

<br>

#### 3. 🔴 static メソッドはインスタンス変数を使えない
> 対象: `OrderManager.add()` の `static` 修飾子

| 名前 | 問題 |
|---|---|
| `static` の制約 | static メソッドは **クラスに属する** ためインスタンス変数にアクセスできない。結果、必要なデータはすべて **引数で受け取る** しかなく、データとロジックの一体化（カプセル化）が構造的に不可能になる |

<br>

#### 4. 🔴 インスタンスメソッドのフリをした static メソッドにも注意
> 対象: `static` が付いていないが、インスタンス変数を使っていないメソッド

```java
// ⚠️ インスタンスメソッドに見えるが、インスタンス変数を一切使っていない
class PaymentManager {
    private int discountRate;  // 割引率（あるのに使っていない！）

    int add(int moneyAmount1, int moneyAmount2) {
        return moneyAmount1 + moneyAmount2;  // ← discountRate を使っていない
    }
}
```

| 名前 | 問題 |
|---|---|
| 隠れた static メソッド | `static` キーワードは付いていないが、 **インスタンス変数 `discountRate` を一切使っていない** 。実質的に static メソッドと同じ。このようなメソッドはカプセル化に貢献しておらず、データとロジックの分離を放置している |

<br>

#### 5. 🔴 なぜ static メソッドが多用されてしまうのか？
> 対象: 手続き型プログラミングの影響

| 名前 | 問題 |
|---|---|
| 手続き型の癖 | C言語などの **手続き型言語** では、データ（構造体）とロジック（関数）を **別々に定義するのが当たり前** 。このクセが残ったまま Java や Python を書くと、データクラス＋ static メソッドという構造になりやすい |

> 💡 **用語: 手続き型プログラミング（Procedural Programming）とは？**
> プログラムを **手順（手続き＝関数）の集まり** として書くスタイル。データと関数が分離しているのが特徴。C言語が代表例。オブジェクト指向では、データとロジックを **1つのクラスにまとめる** ことで保守性を高める。

---

### 🌟 改善版コード（データとロジックを1つのクラスにまとめる）

**一言でいうと:** `MoneyData`（データだけ）と `OrderManager`（ロジックだけ）を統合し、 **`Money` クラスにデータもロジックも閉じ込める** 。static メソッドの引数で渡していたデータを、 **インスタンス変数** として持たせる。

**Java:**
```java
// ✅ 金額を表すクラス（データ + ロジック = カプセル化）
class Money {
    private final int amount;  // 🔒 外部から変更不可

    Money(final int amount) {
        if (amount < 0) {
            throw new IllegalArgumentException("金額は0以上を指定してください。");
        }
        this.amount = amount;
    }

    // ✅ インスタンスメソッド：自分自身の amount を使って計算
    Money add(final Money other) {
        return new Money(this.amount + other.amount);
    }

    int getAmount() {
        return amount;
    }
}
```

📝 **使い方:**
```java
Money money1 = new Money(1000);
Money money2 = new Money(500);

// ✅ データとロジックが一体化！
Money total = money1.add(money2);
System.out.println(total.getAmount()); // → 1500

// ❌ 不正な値はそもそも作れない
// new Money(-100);  // → IllegalArgumentException
```

**Python:**
```python
# ✅ 金額を表すクラス（データ + ロジック = カプセル化）
class Money:
    def __init__(self, amount: int) -> None:
        if amount < 0:
            raise ValueError("金額は0以上を指定してください。")
        self._amount: int = amount

    @property
    def amount(self) -> int:
        return self._amount

    # ✅ インスタンスメソッド：自分自身の _amount を使って計算
    def add(self, other: "Money") -> "Money":
        return Money(self._amount + other.amount)
```

📝 **使い方:**
```python
money1 = Money(1000)
money2 = Money(500)

# ✅ データとロジックが一体化！
total = money1.add(money2)
print(total.amount)  # → 1500

# ❌ 不正な値はそもそも作れない
# Money(-100)  # → ValueError
```

**TypeScript:**
```typescript
// ✅ 金額を表すクラス（データ + ロジック = カプセル化）
class Money {
    private readonly amount: number;

    constructor(amount: number) {
        if (amount < 0) throw new Error("金額は0以上を指定してください。");
        this.amount = amount;
    }

    // ✅ インスタンスメソッド：自分自身の amount を使って計算
    add(other: Money): Money {
        return new Money(this.amount + other.amount);
    }

    getAmount(): number {
        return this.amount;
    }
}
```

📝 **使い方:**
```typescript
const money1 = new Money(1000);
const money2 = new Money(500);

// ✅ データとロジックが一体化！
const total = money1.add(money2);
console.log(total.getAmount()); // → 1500

// ❌ 不正な値はそもそも作れない
// new Money(-100);  // → Error
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| クラス構成 | `MoneyData`（データ）+ `OrderManager`（ロジック） | `Money`（データ + ロジック） | **1つのクラスにカプセル化** 。関連するものが1箇所にまとまった |
| メソッドの種類 | `static int add(int, int)` | `Money add(Money other)` | **インスタンスメソッド** に変更。自分自身のデータを使って計算 |
| データの安全性 | `amount` が外部から自由に変更可能 | `private final` で **外部から変更不可** | 不正な値の代入を防止 |
| バリデーション | なし（負の金額も入れられる） | コンストラクタで **0以上を保証** | 不正な `Money` は作れない |

---

### 💡 static メソッドの正しい使い方

すべての static メソッドが悪いわけではない。以下のようなケースでは **static メソッドが適切** である。

| ケース | 例 | 理由 |
|---|---|---|
| ファクトリメソッド | `Money.of(1000)` | インスタンス生成の方法を制御するため |
| ユーティリティ関数 | `Math.max(a, b)` | **特定のインスタンスに属さない** 汎用的な計算 |
| 定数的な値の提供 | `Color.RED` | 事前定義された不変の値 |

> ⚠️ ただし「引数にデータを渡して操作する static メソッド」は、 **そのデータを持つクラスのインスタンスメソッドにすべきでないか？** と常に疑うこと。

---

### 🎯 まとめ（static メソッドの誤用における考え方）

#### 1. 目的（Why）
> **「データとロジックをカプセル化し、低凝集とデータクラスの発生を防ぐため」**

* static メソッドでロジックを書くと、データとロジックが分離し **低凝集** になる。
* データだけのクラス（データクラス）が生まれ、 **カプセル化が崩壊** する。
* 手続き型プログラミングの癖が原因で、無意識にこのパターンに陥りやすい。

#### 2. 目標（What）
> **「データを持つクラスが、自分のデータを使うロジックも持っている状態」**

* 金額のデータと金額の計算ロジックが、同じ `Money` クラスに閉じ込められている。
* インスタンス変数を使わないメソッド（隠れた static メソッド）が存在しない。

#### 3. 手段（How）
> **「static メソッドの引数を、インスタンス変数に置き換える」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| クラス構成 | `MoneyData` + `OrderManager` | `Money`（統合） |
| メソッド | `static int add(int, int)` | `Money add(Money other)`（インスタンスメソッド） |
| データ保護 | `int amount`（公開） | `private final int amount`（非公開・不変） |

> ※ static メソッドが **すべて悪い** わけではない。ファクトリメソッドやユーティリティ関数には適切な用途がある。「引数で受け取ったデータを操作しているだけの static メソッド」を見つけたら、 **そのデータを持つクラスのインスタンスメソッドにすべき** と考えよう。
