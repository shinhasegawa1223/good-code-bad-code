## 成熟したクラス設計 ― 貧血ドメインモデルを卒業する

> 🏗️ **身近なたとえ:** 「名前と住所だけが書いてある空っぽのダンボール箱」を想像してください。
> 中身の管理ルール（壊れ物注意・上限重量）は箱に書かれておらず、運ぶ人が毎回自分で気をつけなければなりません。
> クラスも同じで、 **データだけ持ってルールが外にある状態** は危険です。データとルールを一体化させて **「自分で自分を守れるクラス」** に育てることが大切です。

> 💡 **この章のキーメッセージ:**
> クラスは **自分自身のインスタンス変数を操作するように** 実装すること。
> データの加工・検証・計算を外部任せにせず、クラスの中に閉じ込めるのが「成熟したクラス」の条件。

**Java:**
```java
// データだけ持っていて、ルールやロジックが何もない「貧血ドメインモデル」
class Money {
    int amount;
    Currency currency;
}
```

**Python:**
```python
# データだけ持っていて、ルールやロジックが何もない「貧血ドメインモデル」
class Money:
    def __init__(self, amount: int, currency: str):
        self.amount = amount
        self.currency = currency
```

**TypeScript:**
```typescript
// データだけ持っていて、ルールやロジックが何もない「貧血ドメインモデル」
class Money {
    amount: number;
    currency: string;

    constructor(amount: number, currency: string) {
        this.amount = amount;
        this.currency = currency;
    }
}
```

---

### 🚨 何がダメなのか？ (貧血ドメインモデルの問題点)

> 💡 **用語: 貧血ドメインモデル（Anemic Domain Model）とは？**
> データ（フィールド）だけを持ち、そのデータを操作するロジック（メソッド）を **一切持たない** クラスのこと。「貧血」＝中身がスカスカという意味。データの加工や検証は全て外部のコードが行うため、バグが散らばりやすくなる。

<br>

#### 1. 🔴 不正な値を自由に入れられてしまう（生焼けオブジェクト）
> 対象: `new Money(-100, null)`

| 名前 | 問題 |
|---|---|
| バリデーション不在 | コンストラクタにチェックがないため、 **金額がマイナス** や **通貨が `null`** のオブジェクトが普通に作れてしまう。不正な状態のオブジェクトがシステム内を流通し、後から原因不明のバグを引き起こす。 |

> 💡 **用語: 生焼けオブジェクト（Half-Baked Object）とは？**
> 初期化が不完全なまま使われてしまうオブジェクトのこと。必要なデータが揃っていない、またはバリデーションされていない状態で外部に渡ると、予期しない動作やエラーの原因になる。

<br>

#### 2. 🔴 データを外部から直接書き換えられてしまう（副作用）
> 対象: `money.amount += other`

| 名前 | 問題 |
|---|---|
| ミュータブルなフィールド | フィールドが `public` かつ変更可能なため、 **どこからでも値を上書きできる** 。複数箇所から書き換えられると、「今この `Money` はいくらなのか？」が追いかけにくくなり、バグの温床になる。 |

> 💡 **用語: 副作用（Side Effect）とは？**
> メソッドを呼んだ結果、引数やオブジェクト自身の状態が **予期しない形で変化** してしまうこと。例えば `add()` を呼んだだけなのに元の金額が書き換わってしまうと、呼び出し元は混乱する。

<br>

#### 3. 🔴 意図しない型の値を渡せてしまう
> 対象: `money.add(3)` （ただの `int` を渡している）

| 名前 | 問題 |
|---|---|
| プリミティブ型の引数 | `add(int other)` のように `int` を受け取ると、 **「個数」なのか「金額」なのか区別がつかない** 。`money.add(count)` のように無関係な数値を渡してもコンパイルエラーにならず、計算結果がおかしくなる。 |

<br>

#### 4. 🔴 ロジックがクラスの外に散らばる（重複コード）
> 対象: 外部に書かれたバリデーションや計算処理

| 名前 | 問題 |
|---|---|
| ロジックの散在 | 「金額がマイナスにならないようチェック」「通貨単位が同じか確認」などの処理が **使う側のあちこちにコピペされる** 。1箇所直し忘れると即バグになり、修正漏れのリスクが高い。 |

---

### 🌟 改善版コード（ステップバイステップで成熟したクラスに育てる）

以下では、貧血ドメインモデルを **5つのステップ** で段階的に改善していく過程を見ていきます。

<br>

---

#### 📦 Step 1: コンストラクタを追加する（インスタンス変数の確実な初期化）

> **一言でいうと:** フィールドに値を必ずセットするようにする。

```java
class Money {
    int amount;
    Currency currency;

    Money(int amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }
}
```

> ✅ **良くなった点:** コンストラクタにより、インスタンス生成時に **確実に `amount` と `currency` が初期化される** ようになった。フィールドが未設定のまま使われる心配がなくなる。

> ⚠️ **まだ不十分な点:** しかし、こういう呼び方もできてしまう：

```java
Money money = new Money(-100, null);  // ← マイナス金額＆通貨がnull！
```

> 不正な値を渡しても **エラーにならずそのまま生成される** ため、問題を先送りしているだけ。

<br>

---

#### 🔒 Step 2: バリデーションを追加する（不正な値の排除）

> **一言でいうと:** コンストラクタに **「門番（ガード節）」** を置いて、不正な値を弾く。

> 💡 **用語: ガード節（Guard Clause）とは？**
> メソッドやコンストラクタの冒頭で「この条件を満たさなければ即座にエラーにする」チェック処理のこと。門番のように不正な入力をブロックする。

```java
class Money {
    int amount;
    Currency currency;

    Money(final int amount, final Currency currency) {
        if (amount < 0) {
            throw new IllegalArgumentException("金額には0以上を指定してください。");
        }
        if (currency == null) {
            throw new NullPointerException("通貨単位を指定してください。");
        }
        this.amount = amount;
        this.currency = currency;
    }
}
```

> ✅ **良くなった点:** `new Money(-100, null)` と書くと **即座に例外が発生** し、不正なオブジェクトは作れなくなった。

> ⚠️ **まだ不十分な点:** 生成後にフィールドを直接書き換えられてしまう（`money.amount = -500;` が可能）。

<br>

---

#### 🧮 Step 3: 計算ロジックをクラス内に移す + 不変（イミュータブル）にする

> **一言でいうと:** データを持っている側にロジックを寄せ、フィールドを `final` にして変更を禁止する。

まず、計算ロジックをクラスの中に入れた **第一段階** ：

```java
class Money {
    // 省略
    void add(int other) {
        amount += other;  // ⚠️ 元のオブジェクトの値を直接書き換えている
    }
}
```

> ⚠️ **問題:** `amount += other` は **元のオブジェクトを直接変更（ミューテーション）** している。以下のようなコードが書かれると、途中で `money` の値がどんどん変わっていって追跡が困難になる：

```java
// ❌ 読みにくい例：money の値が途中で何度も変わる
money.amount = originalPrice;
if (specialServiceAdded) {
    money.add(additionalServiceFee);
    if (seasonOffApplied) {
        money.amount = money.amount - seasonPrice();
    }
}
// ← ここで money.amount はいくら？ 条件分岐を全部追わないと分からない！
```

> 💡 **用語: イミュータブル（Immutable / 不変）とは？**
> 一度作ったオブジェクトの中身を **後から変更できない** ようにすること。値を変えたい場合は、元のオブジェクトはそのまま残して **新しいオブジェクトを作って返す** 。これにより「いつの間にか値が変わっていた」という副作用を根絶できる。

**改善：`final` を使ってフィールドを不変にする:**

```java
class Money {
    final int amount;      // ← final = 一度代入したら二度と変更できない
    final Currency currency;

    Money(final int amount, final Currency currency) {
        if (amount < 0) {
            throw new IllegalArgumentException("金額には0以上を指定してください。");
        }
        if (currency == null) {
            throw new NullPointerException("通貨単位を指定してください。");
        }
        this.amount = amount;
        this.currency = currency;
    }
}
```

> 💡 **`final` キーワードの詳細解説**
>
> `final` を付けたインスタンス変数は **一度しか代入できない** 。代入できるのは以下の2パターンのみ：

**パターン①: 変数宣言時に代入する**
```java
class Config {
    final int MAX_RETRY = 3;        // ← 宣言と同時に代入（全インスタンス共通の値に使うことが多い）
    final String DEFAULT_NAME = "Unknown";
}
```

**パターン②: コンストラクタで代入する**
```java
class Config {
    final int maxRetry;             // ← 宣言時には代入しない
    final String name;

    Config(final int maxRetry, final String name) {
        this.maxRetry = maxRetry;   // ← コンストラクタで代入（インスタンスごとに異なる値に使う）
        this.name = name;
    }
}
```

> ⚠️ **注意:** 上記のどちらでも代入しなかった場合、 **コンパイルエラー** になる。また、一度代入した後に再代入しようとしても **コンパイルエラー** になる。

**値を変更したい場合は、新しいインスタンスを生成して返す:**

```java
Money add(int other) {
    int added = amount + other;
    return new Money(added, currency);  // ← 元の Money は変わらない！新しい Money を返す
}
```

> ✅ **良くなった点:** フィールドが `final` なので外部からも内部からも書き換えられない。`add()` は元のオブジェクトを変更せず新しいオブジェクトを返すため、副作用がゼロになった。

<br>

**引数やローカル変数にも `final` を付けるべき理由：**

```java
// ❌ NG: 引数の値を上書きしている（何のために引数を受け取った？）
void doSomething(int value) {
    value = 10;  // ← 引数が10に上書きされる。呼び出し元の意図が無視されてしまう
}

// ✅ OK: final を付けると再代入はコンパイルエラーになる
void doSomething(final int value) {
    // value = 10;  // ← コンパイルエラー！意図しない再代入を防げる
}
```

> 💡 **ポイント:** `final` は「この変数は変えません」という **意思表示** であると同時に、 **コンパイラに変更を禁止させるセーフティネット** でもある。引数やローカル変数にも積極的に付けることで、バグの混入を防げる。

**`final` を全体に適用した状態:**

```java
Money add(final int other) {           // ← 引数も final
    final int added = amount + other;  // ← ローカル変数も final
    return new Money(added, currency);
}
```

<br>

---

#### 🎯 Step 4: 引数の型を `Money` に限定する（型安全）

> **一言でいうと:** 引数を `int` ではなく `Money` 型にして、意図しないものを渡せなくする。

**問題：`int` だと何でも渡せてしまう：**

```java
final int count = 3;     // ← これは「個数」であって「金額」ではない
money.add(count);         // ← でもコンパイルが通ってしまう！結果は意味不明な数値に
```

**改善：引数を `Money` 型に限定する：**

```java
Money add(final Money other) {             // ← Money 型だけ受け付ける
    final int added = amount + other.amount;
    return new Money(added, currency);
}
```

> ✅ **良くなった点:** `money.add(3)` や `money.add(count)` は **コンパイルエラー** になる。 `Money` 型のオブジェクトしか渡せないため、「金額」以外の値が紛れ込む心配がなくなった。

<br>

---

#### 🛡️ Step 5: ビジネスルールのバリデーションを追加する

> **一言でいうと:** 「通貨単位が異なる金額同士の加算」など、 **現実にありえない操作をブロック** する。

```java
Money add(final Money other) {
    if (!currency.equals(other.currency)) {
        throw new IllegalArgumentException("通貨単位が違います。");
    }
    final int added = amount + other.amount;
    return new Money(added, currency);
}
```

> ✅ **良くなった点:** 「円ドル混合加算」のようなビジネスルール違反が **コードレベルで不可能** になった。

<br>

---

### 🚫 現実にそぐわないメソッドを追加しない

成熟したクラスを作る際、 **「便利そうだから」** と現実にはあり得ない操作を追加してはいけません。

```java
// ❌ 金額どうしの「掛け算」は現実にはありえない
class Money {
    Money multiply(Money other) {
        final int multiplied = amount * other.amount;
        return new Money(multiplied, currency);
    }
}
// 「1000円 × 500円 = 500,000円」？ 意味不明！
// 金額 × 個数（int）ならありえるが、金額 × 金額 は現実の業務にそぐわない
```

> 💡 **ポイント:** メソッドを追加する前に **「現実世界でその操作は意味があるか？」** を問いかけること。善意で作った不要なメソッドが、将来バグや混乱の原因になる。

<br>

---

### ✅ 最終形態（全ステップを適用した成熟クラス）

**Java:**
```java
import java.util.Currency;

// ✅ 成熟したクラス（不変 + バリデーション + 自己完結したロジック）
class Money {
    // 📦 final = 一度代入したら変更不可（不変性を保証）
    final int amount;
    final Currency currency;

    // 🔒 コンストラクタで門番チェック（不正値は存在させない）
    Money(final int amount, final Currency currency) {
        if (amount < 0) {
            throw new IllegalArgumentException("金額には0以上を指定してください。");
        }
        if (currency == null) {
            throw new NullPointerException("通貨単位を指定してください。");
        }
        this.amount = amount;
        this.currency = currency;
    }

    // 🧮 加算：元のオブジェクトは変更せず、新しい Money を返す（不変）
    // 引数も Money 型に限定 → int の取り違えを防ぐ
    // 通貨単位の一致チェック → ビジネスルールを守る
    Money add(final Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("通貨単位が違います。");
        }
        final int added = amount + other.amount;
        return new Money(added, currency);
    }
}
```

📝 **使い方:**
```java
Currency jpy = Currency.getInstance("JPY");

Money price     = new Money(1000, jpy);
Money tax       = new Money(100, jpy);
Money total     = price.add(tax);   // total = 1100円の新しい Money が返る
// price は変わらず1000円のまま（不変！）

// ❌ 不正な値はコンストラクタが即座にブロック
// Money bad = new Money(-100, null);  // → 例外が発生！

// ❌ int を渡すことはできない（型安全）
// price.add(3);  // → コンパイルエラー！Money 型しか受け付けない

// ❌ 通貨が異なる加算もブロック
// Currency usd = Currency.getInstance("USD");
// Money dollar = new Money(50, usd);
// price.add(dollar);  // → IllegalArgumentException: 通貨単位が違います。
```

**Python:**
```python
# ✅ 成熟したクラス（不変 + バリデーション + 自己完結したロジック）
class Money:
    def __init__(self, amount: int, currency: str) -> None:
        # 🔒 門番チェック
        if amount < 0:
            raise ValueError("金額には0以上を指定してください。")
        if not currency:
            raise ValueError("通貨単位を指定してください。")
        # _プレフィックス = 外部から直接触らない慣習
        self._amount: int = amount
        self._currency: str = currency

    @property
    def amount(self) -> int:
        """読み取り専用"""
        return self._amount

    @property
    def currency(self) -> str:
        """読み取り専用"""
        return self._currency

    # 🧮 加算：新しい Money を返す（不変）
    def add(self, other: 'Money') -> 'Money':
        if self._currency != other._currency:
            raise ValueError("通貨単位が違います。")
        added: int = self._amount + other._amount
        return Money(added, self._currency)
```

📝 **使い方:**
```python
price = Money(1000, "JPY")
tax   = Money(100, "JPY")
total = price.add(tax)      # total = 1100円の新しい Money
# price.amount は変わらず1000（不変！）

# ❌ 不正な値は即座にブロック
# bad = Money(-100, "")   # → ValueError!
```

**TypeScript:**
```typescript
// ✅ 成熟したクラス（不変 + バリデーション + 自己完結したロジック）
class Money {
    // readonly = 一度代入したら変更不可
    readonly amount: number;
    readonly currency: string;

    constructor(amount: number, currency: string) {
        // 🔒 門番チェック
        if (amount < 0) {
            throw new Error("金額には0以上を指定してください。");
        }
        if (!currency) {
            throw new Error("通貨単位を指定してください。");
        }
        this.amount = amount;
        this.currency = currency;
    }

    // 🧮 加算：新しい Money を返す（不変）
    add(other: Money): Money {
        if (this.currency !== other.currency) {
            throw new Error("通貨単位が違います。");
        }
        const added = this.amount + other.amount;
        return new Money(added, this.currency);
    }
}
```

📝 **使い方:**
```typescript
const price = new Money(1000, "JPY");
const tax   = new Money(100, "JPY");
const total = price.add(tax);  // total = 1100円の新しい Money
// price.amount は変わらず1000（不変！）

// ❌ 不正な値は即座にブロック
// const bad = new Money(-100, ""); // → Error!
```

<br>

#### ✅ 改善ステップのビフォー・アフター（全体まとめ）

| ステップ | やったこと | 解決した問題 |
|---|---|---|
| Step 1: コンストラクタ追加 | インスタンス変数を確実に初期化 | フィールドが未設定のまま使われるリスクを排除 |
| Step 2: バリデーション追加 | 不正値（マイナス・null）をコンストラクタで弾く | **生焼けオブジェクトが物理的に存在できない** 設計に |
| Step 3: `final` + 新インスタンス返却 | フィールドを不変にし、計算結果は新オブジェクトで返す | **副作用を排除** し、値の追跡が容易に |
| Step 4: 引数を `Money` 型に | `int` ではなく `Money` 型だけ受け付ける | **型の取り違え** がコンパイル時に検出される |
| Step 5: ビジネスルール検証 | 通貨単位の一致チェックを追加 | **現実にありえない操作** をコードレベルで禁止 |

---

### 🎯 まとめ（成熟したクラス設計における考え方）

#### 1. 目的（Why）
> **「貧血ドメインモデルが引き起こす6つの弊害を根絶するため」**

* **重複コード** — バリデーションや計算処理があちこちにコピペされる
* **修正漏れ** — 1箇所直してもコピペ先を直し忘れてバグになる
* **可読性低下** — ロジックが散らばって全体像が掴めなくなる
* **生焼けオブジェクト** — 不完全な状態のオブジェクトがシステムに混入する
* **不正値混入** — 不正な値がチェックされずに通ってしまう
* **副作用** — 意図しない変更が別の場所に波及する

#### 2. 目標（What）
> **「クラス自身がデータとルールを完全に管理している状態」**

* 外部が「この金額はマイナスじゃないよね？」と心配する必要がない。クラスが **自分で自分を守っている** 。

#### 3. 手段（How）
> **「データ + バリデーション + ロジック + 不変性を1つのクラスにまとめる」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| 初期化 | フィールドだけ持つ空クラス | コンストラクタでバリデーション（門番） |
| フィールド | `int amount;`（変更可能） | `final int amount;`（不変） |
| メソッド引数 | `add(int other)`（何でも渡せる） | `add(Money other)`（型で制限） |
| 戻り値 | `void add()`（元を書き換え） | `Money add()`（新しいインスタンスを返す） |
| メソッド追加 | 「便利そうだから」で追加 | 現実に即した操作だけを提供 |

> ※ 「データだけのクラスを見つけたら、そのデータを操作するロジックとバリデーションをクラスの中に引っ越しさせる」のが成熟化の第一歩です。
