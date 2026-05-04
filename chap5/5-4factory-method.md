## 初期化ロジックの分散とファクトリメソッド

**Java:**
```java
class GiftPoint {
    private static final int MIN_POINT = 0;
    final int value;

    // ❌ コンストラクタが公開されているため、どこからでも自由に生成できてしまう
    GiftPoint(final int point) {
        if (point < MIN_POINT) {
            throw new IllegalArgumentException("ポイントが0以上ではありません。");
        }
        this.value = point;
    }

    GiftPoint add(final GiftPoint other) {
        return new GiftPoint(this.value + other.value);
    }
}
```

```java
// ❌ 呼び出し側で具体的な数値を指定して初期化（初期化ロジックが分散する）
GiftPoint standardMembershipPoint = new GiftPoint(3000);
GiftPoint premiumMembershipPoint = new GiftPoint(5000);
```

**Python:**
```python
class GiftPoint:
    MIN_POINT: int = 0

    # ❌ どこからでも自由に生成できてしまう
    def __init__(self, point: int) -> None:
        if point < self.MIN_POINT:
            raise ValueError("ポイントが0以上ではありません。")
        self._value: int = point

    @property
    def value(self) -> int:
        return self._value

    def add(self, other: "GiftPoint") -> "GiftPoint":
        return GiftPoint(self._value + other.value)
```

```python
# ❌ 呼び出し側で具体的な数値を指定して初期化
standard_membership_point = GiftPoint(3000)
premium_membership_point = GiftPoint(5000)
```

**TypeScript:**
```typescript
class GiftPoint {
    private static readonly MIN_POINT: number = 0;
    readonly value: number;

    // ❌ コンストラクタが公開されているため、どこからでも自由に生成できてしまう
    constructor(point: number) {
        if (point < GiftPoint.MIN_POINT) {
            throw new Error("ポイントが0以上ではありません。");
        }
        this.value = point;
    }

    add(other: GiftPoint): GiftPoint {
        return new GiftPoint(this.value + other.value);
    }
}
```

```typescript
// ❌ 呼び出し側で具体的な数値を指定して初期化
const standardMembershipPoint = new GiftPoint(3000);
const premiumMembershipPoint = new GiftPoint(5000);
```

---

### 🚨 何がダメなのか？ (初期化ロジックの分散の問題点)

> 💡 **用語: 初期化ロジックの分散 とは？**
> オブジェクトを生成する際、「どんな値で初期化するか（例: 標準会員は3000ポイント）」というルールが、クラスの内部ではなく、 **呼び出し側のコードに散らばってしまう** 状態のこと。

<br>

#### 1. 🔴 コンストラクタの公開による用途の無秩序化
> 対象: `public`（公開された）コンストラクタ

| 名前 | 問題 |
|---|---|
| コンストラクタの公開 | コンストラクタを公開（`public`）すると、様々な場所で任意の数値を使って自由にインスタンスが生成されてしまう。「標準会員は3000ポイント」というルールがクラス外のあちこちに散らばり、保守性が低下する。 |

<br>

#### 2. 🔴 仕様変更時の修正漏れリスク
> 対象: `new GiftPoint(3000)` のような生成ロジック

| 名前 | 問題 |
|---|---|
| マジックナンバーの散在 | 「入会ポイントを3000から4000に変更したい」となった場合、ソースコード全体から `new GiftPoint(3000)` と書かれた箇所をすべて探し出して修正しなければならない。修正漏れが発生しやすく、バグの原因になる。 |

---

### 🌟 改善版コード（ファクトリメソッドの導入）

> 💡 **用語: ファクトリメソッド（Factory Method）とは？**
> インスタンスの生成（`new`）を専門に行うメソッドのこと。コンストラクタを `private` にして外部から隠し、代わりに **用途が明確な名前のメソッド**（例: `forStandardMembership()`）を用意することで、生成のルールを1箇所に集中させる。

**一言でいうと:** コンストラクタを `private` にして直接の `new` を禁止し、代わりに目的別の **ファクトリメソッド** を用意することで、生成ロジックをクラス内に集約する。

**Java:**
```java
class GiftPoint {
    private static final int MIN_POINT = 0;
    private static final int STANDARD_MEMBERSHIP_POINT = 3000;
    private static final int PREMIUM_MEMBERSHIP_POINT = 10000;
    
    final int value;

    // 🔒 外部からは new できない（クラス内部でのみ生成可能）
    private GiftPoint(final int point) {
        if (point < MIN_POINT) {
            throw new IllegalArgumentException("ポイントが0以上ではありません。");
        }
        this.value = point;
    }

    // ✅ 目的が明確なファクトリメソッドを用意
    static GiftPoint forStandardMembership() {
        return new GiftPoint(STANDARD_MEMBERSHIP_POINT);
    }

    static GiftPoint forPremiumMembership() {
        return new GiftPoint(PREMIUM_MEMBERSHIP_POINT);
    }
}
```

```java
// ✅ ファクトリメソッド経由で生成（数値を知らなくてよい）
GiftPoint standardMembershipPoint = GiftPoint.forStandardMembership();
GiftPoint premiumMembershipPoint = GiftPoint.forPremiumMembership();
```

**Python:**
```python
class GiftPoint:
    MIN_POINT: int = 0
    STANDARD_MEMBERSHIP_POINT: int = 3000
    PREMIUM_MEMBERSHIP_POINT: int = 10000

    # 🔒 Pythonには完全なprivateコンストラクタはないが、直接呼ばない規約とする（_を付けるなど）
    def __init__(self, point: int) -> None:
        if point < self.MIN_POINT:
            raise ValueError("ポイントが0以上ではありません。")
        self._value: int = point

    @property
    def value(self) -> int:
        return self._value

    # ✅ クラスメソッドをファクトリとして使用
    @classmethod
    def for_standard_membership(cls) -> "GiftPoint":
        return cls(cls.STANDARD_MEMBERSHIP_POINT)

    @classmethod
    def for_premium_membership(cls) -> "GiftPoint":
        return cls(cls.PREMIUM_MEMBERSHIP_POINT)
```

```python
# ✅ ファクトリメソッド経由で生成
standard_membership_point = GiftPoint.for_standard_membership()
premium_membership_point = GiftPoint.for_premium_membership()
```

**TypeScript:**
```typescript
class GiftPoint {
    private static readonly MIN_POINT: number = 0;
    private static readonly STANDARD_MEMBERSHIP_POINT: number = 3000;
    private static readonly PREMIUM_MEMBERSHIP_POINT: number = 10000;
    
    readonly value: number;

    // 🔒 外部からは new できない
    private constructor(point: number) {
        if (point < GiftPoint.MIN_POINT) {
            throw new Error("ポイントが0以上ではありません。");
        }
        this.value = point;
    }

    // ✅ 目的が明確なファクトリメソッドを用意
    static forStandardMembership(): GiftPoint {
        return new GiftPoint(GiftPoint.STANDARD_MEMBERSHIP_POINT);
    }

    static forPremiumMembership(): GiftPoint {
        return new GiftPoint(GiftPoint.PREMIUM_MEMBERSHIP_POINT);
    }
}
```

```typescript
// ✅ ファクトリメソッド経由で生成
const standardMembershipPoint = GiftPoint.forStandardMembership();
const premiumMembershipPoint = GiftPoint.forPremiumMembership();
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| インスタンス生成 | `new GiftPoint(3000)` | `GiftPoint.forStandardMembership()` | 呼び出し側が **「具体的な数値」** ではなく **「目的」** を指定して生成するようになった |
| コンストラクタ | `public`（誰でも呼べる） | `private`（内部からのみ呼べる） | 想定外の数値で不正にインスタンスが作られるのを防げるようになった |
| ロジックの場所 | 呼び出し側の各所に分散 | `GiftPoint` クラスの内部に集約 | 仕様変更時（入会ポイントの変更など）の修正箇所が1箇所で済むようになった |

---

### 🎯 まとめ（ファクトリメソッドにおける考え方）

#### 1. 目的（Why）
> **「初期化ロジックの分散を防ぎ、メンテナンス性を向上させるため」**

* コンストラクタを公開すると、様々な場所で直接 `new` されてしまい、初期化ルールの管理ができなくなる。
* 仕様変更時に修正漏れが発生し、バグの温床となる。

#### 2. 目標（What）
> **「生成ルールがクラス内部に閉じ込められ、呼び出し側は目的だけを伝える状態」**

* インスタンスの生成がクラス自身の責任として管理されている。
* 呼び出し側のコードを読んだとき、そのオブジェクトが **「何のために」** 生成されたのかが一目でわかる。

#### 3. 手段（How）
> **「コンストラクタを private にし、目的別のファクトリメソッドを用意する」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| コンストラクタ | `GiftPoint(int point)` | `private GiftPoint(int point)`（外部からの生成を禁止） |
| 生成方法の提供 | 各自が `new` で自由に生成 | `static GiftPoint forStandardMembership()`（目的別のメソッド） |

> ※ 生成に関するロジック（例: ポイントの初期値や付与の条件）が増えそうだと感じたら、迷わずコンストラクタを `private` にしてファクトリメソッドを検討しよう。
