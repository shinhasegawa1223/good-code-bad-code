## 成熟したクラス設計 ― 貧血ドメインモデルを卒業する

> 🏗️ **身近なたとえ:** 「名前と住所だけが書いてある空っぽのダンボール箱」を想像してください。
> 中身の管理ルール（壊れ物注意・上限重量）は箱に書かれておらず、運ぶ人が毎回自分で気をつけなければなりません。
> クラスも同じで、 **データだけ持ってルールが外にある状態** は危険です。データとルールを一体化させて **「自分で自分を守れるクラス」** に育てることが大切です。

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

### 🌟 改善版コード（成熟したクラスに育てる）

> 💡 **用語: 不変オブジェクト（Immutable Object）とは？**
> 一度生成したら **中身を変更できない** オブジェクトのこと。値を変えたい場合は、新しいオブジェクトを生成して返す。これにより副作用を防ぎ、コードの予測可能性が格段に上がる。

> 💡 **用語: 値オブジェクト（Value Object）とは？**
> 「金額」「通貨」などの概念を **専用のクラスとして定義** し、バリデーションやロジックを内包させたオブジェクト。プリミティブ型（`int`, `string` 等）の代わりに使うことで、型の取り違えを防げる。

**一言でいうと:** ①コンストラクタで不正値をブロック、②フィールドを `final`（不変）にする、③計算メソッドは自身を変更せず新しいインスタンスを返す、④引数は `Money` 型に限定する — これを全て適用して「自分で自分を守れるクラス」にする。

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

#### 🚫 現実にそぐわないメソッドを追加しない

成熟したクラスを作る際、 **「便利そうだから」** と現実にはあり得ない操作を追加してはいけません。

```java
// ❌ 金額どうしの「掛け算」は現実にはありえない
class Money {
    Money multiply(Money other) {
        final int multiplied = amount * other.amount;
        return new Money(multiplied, currency);
    }
}
// 「1000円 × 500円 = 500000円」？ 意味不明！
```

> 💡 **ポイント:** メソッドを追加する前に「現実世界でその操作は意味があるか？」を問いかけること。善意で作った不要なメソッドが、将来バグや混乱の原因になる。

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| 不正値の防止 | `new Money(-100, null)` が普通に作れる | コンストラクタが即座にエラーを出す | **生焼けオブジェクトが物理的に存在できない** 設計になった |
| データの変更 | `money.amount += 100` で直接書き換え可能 | `final` / `readonly` で変更不可。新しいインスタンスを返す | **副作用が排除** され、コードの予測可能性が向上 |
| 型安全性 | `add(int other)` ⇒ 個数でも何でも渡せる | `add(Money other)` ⇒ `Money` 型だけ受け付ける | **意図しない型の取り違え** がコンパイル時に検出される |
| ロジックの場所 | 外部にバリデーションや計算が散在 | クラス内にロジックが集約 | **修正漏れのリスクがゼロ** になった |

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
