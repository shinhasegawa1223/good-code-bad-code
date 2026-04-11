## プリミティブ型執着 ― 低凝集の原因と値オブジェクトによる改善

> 💡 **用語: プリミティブ型（Primitive Type）とは？**
> `boolean`, `int`, `float`, `String` などの **言語が最初から用意している基本データ型** のこと。便利だが、これだけで設計しようとすすると **ロジックがあちこちに散らばる** 原因になる。

> 💡 **用語: プリミティブ型執着（Primitive Obsession）とは？**
> 本来は専用クラスにすべきデータ（金額、割引率、メールアドレス等）を、 **`int` や `String` のまま使い続けてしまう** アンチパターン。バリデーションや計算ロジックがコード中に散在し、低凝集の原因となる。

### ❌ 悪い例（プリミティブ型だけで実装する）

**Java:**
```java
// ❌ 定価も割引率もプリミティブ型 → バリデーションがここに書かれる
class Common {
    int discountedPrice(int regularPrice, float discountRate) {
        if (regularPrice < 0) {
            throw new IllegalArgumentException();
        }
        if (discountRate < 0.0f) {
            throw new IllegalArgumentException();
        }
        return (int)(regularPrice * (1.0f - discountRate));
    }
}
```

```java
// ❌ 別のクラスでも同じバリデーションが書かれている（重複！）
class Util {
    boolean isFairPrice(int regularPrice) {
        if (regularPrice < 0) {                // ← Common と同じチェック！
            throw new IllegalArgumentException();
        }
        return regularPrice < 100000;
    }
}
```

**Python:**
```python
# ❌ プリミティブ型だけで実装 → バリデーションが散在
class Common:
    def discounted_price(self, regular_price: int, discount_rate: float) -> int:
        if regular_price < 0:
            raise ValueError()
        if discount_rate < 0.0:
            raise ValueError()
        return int(regular_price * (1.0 - discount_rate))

# ❌ 別の場所でも同じバリデーション（重複！）
class Util:
    def is_fair_price(self, regular_price: int) -> bool:
        if regular_price < 0:               # ← Common と同じチェック！
            raise ValueError()
        return regular_price < 100000
```

**TypeScript:**
```typescript
// ❌ プリミティブ型だけで実装 → バリデーションが散在
class Common {
    discountedPrice(regularPrice: number, discountRate: number): number {
        if (regularPrice < 0) throw new Error();
        if (discountRate < 0) throw new Error();
        return Math.floor(regularPrice * (1 - discountRate));
    }
}

// ❌ 別の場所でも同じバリデーション（重複！）
class Util {
    isFairPrice(regularPrice: number): boolean {
        if (regularPrice < 0) throw new Error(); // ← Common と同じチェック！
        return regularPrice < 100000;
    }
}
```

---

### 🚨 何がダメなのか？ (プリミティブ型執着の問題点)

> 💡 **用語: 低凝集（Low Cohesion）とは？**
> 関連するデータとロジックが **1つのクラスにまとまっていない** 状態。「定価」に関するデータ（値）とロジック（バリデーション）が `Common` にも `Util` にもバラバラに存在しているのが低凝集の典型例。

<br>

#### 1. 🔴 バリデーションが複数箇所に重複する
> 対象: `if (regularPrice < 0)` が `Common` と `Util` の **両方** に存在

| 名前 | 問題 |
|---|---|
| `regularPrice < 0` チェックの重複 | 「定価は0以上」というルールが **`Common` にも `Util` にもコピペ** されている。ルールが変わった時（例：「定価は100以上」に変更）、 **全箇所を漏れなく修正しなければならない** 。1箇所でも漏れるとバグになる |

<br>

#### 2. 🔴 データとロジックがバラバラ（カプセル化できていない）
> 対象: `int regularPrice` というプリミティブ型の引数

| 名前 | 問題 |
|---|---|
| プリミティブ型の引数 | `int regularPrice` は **ただの数字** 。この数字が「定価」であるという情報も、「0以上でなければならない」というルールも、 **プリミティブ型には含まれていない** 。結果、使う側のクラスが毎回バリデーションを書く羽目になる |

> 💡 **用語: カプセル化（Encapsulation）とは？**
> データとそのデータを操作するロジックを **1つのクラスにまとめて隠す** 設計手法。外部からはメソッド経由でしかアクセスできないようにすることで、データの整合性を保つ。

<br>

#### 3. 🔴 引数の取り違えが検知できない
> 対象: `discountedPrice(int regularPrice, float discountRate)` の引数

| 名前 | 問題 |
|---|---|
| 型がプリミティブ | `discountedPrice(500, 0.1f)` と `discountedPrice(0, 500)` のように **引数を逆に渡してもコンパイルエラーにならない** 。プリミティブ型はただの数字なので、コンパイラが「定価と割引率が逆だよ」と教えてくれない |

---

### 🌟 改善版コード（値オブジェクトでカプセル化する）

> 💡 **用語: 値オブジェクト（Value Object）とは？**
> 「定価」「割引率」「メールアドレス」など、 **概念そのものを専用クラスとして定義** したオブジェクト。バリデーションを内部に持ち、 **不正な値ではそもそもインスタンスが作れない** ようにする。

**一言でいうと:** `int regularPrice` の代わりに **`RegularPrice` クラス** を作り、バリデーションをクラス内に閉じ込める。これにより、どこで使っても「定価は必ず0以上」が保証される。

**Java:**
```java
// ✅ 定価を表す値オブジェクト（バリデーション内蔵）
class RegularPrice {
    final int amount;

    RegularPrice(final int amount) {
        if (amount < 0) {
            throw new IllegalArgumentException("定価は0以上を指定してください。");
        }
        this.amount = amount;
    }
}

// ✅ 割引率を表す値オブジェクト
class DiscountRate {
    final float rate;

    DiscountRate(final float rate) {
        if (rate < 0.0f || rate > 1.0f) {
            throw new IllegalArgumentException("割引率は0.0〜1.0の範囲です。");
        }
        this.rate = rate;
    }
}

// ✅ 割引価格を表す値オブジェクト
//    引数がプリミティブ型ではなくクラスになった！
//    ↓↓↓ int, float ではなく RegularPrice, DiscountRate ↓↓↓
class DiscountedPrice {
    final int amount;

    DiscountedPrice(final RegularPrice regularPrice, final DiscountRate discountRate) {
        this.amount = (int)(regularPrice.amount * (1.0f - discountRate.rate));
    }
}
```

📝 **使い方:**
```java
RegularPrice price = new RegularPrice(1000);      // ✅ 0以上が保証済み
DiscountRate rate  = new DiscountRate(0.2f);       // ✅ 0.0〜1.0が保証済み
DiscountedPrice discounted = new DiscountedPrice(price, rate);

System.out.println(discounted.amount); // → 800

// ❌ 不正な値はそもそも作れない
// new RegularPrice(-100);    // → IllegalArgumentException
// new DiscountRate(1.5f);    // → IllegalArgumentException

// ❌ 引数の順番を間違えると型エラー！（プリミティブ型では検知できなかった）
// new DiscountedPrice(rate, price);  // → コンパイルエラー！
```

**Python:**
```python
# ✅ 定価を表す値オブジェクト
class RegularPrice:
    def __init__(self, amount: int) -> None:
        if amount < 0:
            raise ValueError("定価は0以上を指定してください。")
        self._amount: int = amount

    @property
    def amount(self) -> int:
        return self._amount

# ✅ 割引率を表す値オブジェクト
class DiscountRate:
    def __init__(self, rate: float) -> None:
        if rate < 0.0 or rate > 1.0:
            raise ValueError("割引率は0.0〜1.0の範囲です。")
        self._rate: float = rate

    @property
    def rate(self) -> float:
        return self._rate

# ✅ 割引価格を表す値オブジェクト
class DiscountedPrice:
    def __init__(self, regular_price: RegularPrice, discount_rate: DiscountRate) -> None:
        self._amount: int = int(regular_price.amount * (1.0 - discount_rate.rate))

    @property
    def amount(self) -> int:
        return self._amount
```

📝 **使い方:**
```python
price = RegularPrice(1000)
rate = DiscountRate(0.2)
discounted = DiscountedPrice(price, rate)

print(discounted.amount)  # → 800

# ❌ 不正な値はそもそも作れない
# RegularPrice(-100)  # → ValueError
# DiscountRate(1.5)    # → ValueError
```

**TypeScript:**
```typescript
// ✅ 定価を表す値オブジェクト
class RegularPrice {
    readonly amount: number;

    constructor(amount: number) {
        if (amount < 0) throw new Error("定価は0以上を指定してください。");
        this.amount = amount;
    }
}

// ✅ 割引率を表す値オブジェクト
class DiscountRate {
    readonly rate: number;

    constructor(rate: number) {
        if (rate < 0 || rate > 1) throw new Error("割引率は0.0〜1.0の範囲です。");
        this.rate = rate;
    }
}

// ✅ 割引価格を表す値オブジェクト
class DiscountedPrice {
    readonly amount: number;

    constructor(regularPrice: RegularPrice, discountRate: DiscountRate) {
        this.amount = Math.floor(regularPrice.amount * (1 - discountRate.rate));
    }
}
```

📝 **使い方:**
```typescript
const price = new RegularPrice(1000);
const rate = new DiscountRate(0.2);
const discounted = new DiscountedPrice(price, rate);

console.log(discounted.amount); // → 800

// ❌ 不正な値はそもそも作れない
// new RegularPrice(-100);  // → Error
// new DiscountRate(1.5);   // → Error

// ❌ 引数の順番を間違えると型エラー！
// new DiscountedPrice(rate, price);  // → コンパイルエラー！
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| 定価の型 | `int`（ただの数字） | `RegularPrice`（バリデーション内蔵） | **不正な値では作れない** 。0以上が常に保証される |
| 割引率の型 | `float`（ただの小数） | `DiscountRate`（0.0〜1.0を保証） | **範囲外の値では作れない** |
| バリデーション | `Common` にも `Util` にも重複 | `RegularPrice` の **コンストラクタに1箇所だけ** | ルール変更時の修正箇所が1箇所に集約 |
| 引数の取り違え | `int` 同士で逆に渡してもエラーにならない | **型が異なるのでコンパイルエラー** で即座に検出 | バグが実行前に発見される |

---

### 🎯 まとめ（プリミティブ型執着における考え方）

#### 1. 目的（Why）
> **「データとロジックをカプセル化し、バリデーションの重複と取り違えを根絶するため」**

* プリミティブ型のまま設計すると、バリデーションがあちこちにコピペされ（低凝集）、保守が困難になる。
* 型が同じ（`int` と `int`）なので、引数の取り違えをコンパイラが検知できない。

#### 2. 目標（What）
> **「概念ごとに専用クラスを持ち、不正な値が存在できない状態」**

* 「定価」「割引率」「割引価格」がそれぞれ独立したクラスを持ち、各クラスが自分のルールを守る。

#### 3. 手段（How）
> **「プリミティブ型を値オブジェクトに置き換える」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| 定価 | `int regularPrice` | `RegularPrice`（0以上を保証） |
| 割引率 | `float discountRate` | `DiscountRate`（0.0〜1.0を保証） |
| バリデーション | `Common` にも `Util` にも重複 | 各クラスのコンストラクタに **1箇所だけ** |

> ※ 「値オブジェクトを作るのは面倒」と感じるかもしれませんが、バリデーションの重複排除と引数の型安全性によって、 **長期的には圧倒的にバグが減り、保守コストが下がります** 。
