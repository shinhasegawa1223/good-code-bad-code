## 設計パターン ― クラスをさらに堅牢にする6つの技法

> 🧰 **この章で学ぶこと:**
> 3-4で学んだ「成熟したクラス」をさらに強化するための **6つの設計パターン** を紹介します。
> どれも「バグを未然に防ぐ」「変更に強くする」ための実践的なテクニックです。

| # | パターン名 | 一言でいうと |
|---|---|---|
| 1 | 完全コンストラクタ | 生成時点で100%正しい状態を保証する |
| 2 | 値オブジェクト | プリミティブ型を専用クラスに置き換える |
| 3 | ストラテジパターン | 処理の切り替えを if-else からクラスに変える |
| 4 | ポリシーパターン | 複数の条件判定を柔軟に組み合わせる |
| 5 | ファーストクラスコレクション | コレクション操作を専用クラスに閉じ込める |
| 6 | スプラウトクラス | 既存コードを壊さず新機能を安全に追加する |

---

## 1. 完全コンストラクタ（Complete Constructor）

> 💡 **用語: 完全コンストラクタ（Complete Constructor）とは？**
> オブジェクトを生成する時点で **必要な値を全て受け取り、全てのバリデーションを完了** させる設計パターン。生成後のオブジェクトは「100%正しい状態」であることが保証される。

### ❌ 悪い例（不完全な初期化）

**Java:**
```java
// ❌ 生成後にセッターで値をセットする → セットし忘れの危険
class User {
    String name;
    String email;

    User() {} // 空のコンストラクタ

    void setName(String name) { this.name = name; }
    void setEmail(String email) { this.email = email; }
}
```

```java
// 使い方 — セットし忘れてもエラーにならない！
User user = new User();
user.setName("太郎");
// ← email をセットし忘れた！でもコンパイルエラーにならない
System.out.println(user.email); // → null（バグ!!）
```

**Python:**
```python
# ❌ 生成後にセッターで値をセットする → セットし忘れの危険
class User:
    def __init__(self) -> None:
        self.name: str = ""
        self.email: str = ""

    def set_name(self, name: str) -> None:
        self.name = name

    def set_email(self, email: str) -> None:
        self.email = email
```

**TypeScript:**
```typescript
// ❌ 生成後にセッターで値をセットする
class User {
    name: string = "";
    email: string = "";

    setName(name: string): void { this.name = name; }
    setEmail(email: string): void { this.email = email; }
}
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 生焼けオブジェクト | `new User()` で作った直後は `name` も `email` も空。 **セッターを呼ぶ順番や呼び忘れ** によって不正な状態のまま使われてしまう |
| 2 | 🔴 呼び出し側への依存 | 「正しく初期化する責任」がクラスではなく **使う側に押し付けられている** 。使う側がルールを知らないとバグになる |

---

### 🌟 改善版コード

**一言でいうと:** コンストラクタで **全ての値を受け取り、バリデーションも完了** させる。セッターは作らない。

**Java:**
```java
// ✅ 完全コンストラクタ：生成時点で100%正しい状態を保証
class User {
    private final String name;
    private final String email;

    User(final String name, final String email) {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("名前は必須です。");
        }
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("有効なメールアドレスを指定してください。");
        }
        this.name = name;
        this.email = email;
    }

    String getName() { return name; }
    String getEmail() { return email; }
    // ← セッターは存在しない！生成後の変更は不可能
}
```

📝 **使い方:**
```java
User user = new User("太郎", "taro@example.com"); // ✅ 生成と同時に完全な状態
// User bad = new User("", null);                  // ❌ 即座にエラー！
```

**Python:**
```python
# ✅ 完全コンストラクタ
class User:
    def __init__(self, name: str, email: str) -> None:
        if not name:
            raise ValueError("名前は必須です。")
        if not email or "@" not in email:
            raise ValueError("有効なメールアドレスを指定してください。")
        self._name: str = name
        self._email: str = email

    @property
    def name(self) -> str:
        return self._name

    @property
    def email(self) -> str:
        return self._email
```

**TypeScript:**
```typescript
// ✅ 完全コンストラクタ
class User {
    readonly name: string;
    readonly email: string;

    constructor(name: string, email: string) {
        if (!name) throw new Error("名前は必須です。");
        if (!email || !email.includes("@")) throw new Error("有効なメールアドレスを指定してください。");
        this.name = name;
        this.email = email;
    }
}
```

<br>

---

## 2. 値オブジェクト（Value Object）

> 💡 **用語: 値オブジェクト（Value Object）とは？**
> 「金額」「メールアドレス」「数量」など、 **概念そのものを専用のクラスとして定義** したオブジェクト。プリミティブ型（`int`, `string`）の代わりに使うことで、バリデーションやルールをクラスに閉じ込められる。

### ❌ 悪い例（プリミティブ型をそのまま使う）

**Java:**
```java
// ❌ 全部 String や int → 取り違えが起きやすい
class Order {
    String customerName;
    String customerEmail;
    String shippingAddress;
    int quantity;
    int price;

    // quantity と price を間違えて渡してもコンパイルエラーにならない！
    Order(String name, String email, String address, int quantity, int price) {
        this.customerName = name;
        this.customerEmail = email;
        this.shippingAddress = address;
        this.quantity = quantity;
        this.price = price;
    }
}
```

```java
// ❌ 引数の順番を間違えてもエラーにならない（全部同じ型だから）
Order order = new Order(
    "taro@example.com",  // ← name に email を入れてしまった！
    "太郎",               // ← email に name を入れてしまった！
    "東京都...",
    5000,                 // ← quantity に price を入れてしまった！
    3                     // ← price に quantity を入れてしまった！
);
```

**Python:**
```python
# ❌ 全部 str や int → 取り違えが起きやすい
class Order:
    def __init__(self, name: str, email: str, address: str, quantity: int, price: int) -> None:
        self.customer_name: str = name
        self.customer_email: str = email
        self.shipping_address: str = address
        self.quantity: int = quantity
        self.price: int = price
```

**TypeScript:**
```typescript
// ❌ 全部 string や number → 取り違えが起きやすい
class Order {
    constructor(
        public customerName: string,
        public customerEmail: string,
        public shippingAddress: string,
        public quantity: number,
        public price: number
    ) {}
}
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 型の取り違え | `name` と `email` はどちらも `String` なので、 **順番を間違えてもコンパイラが検出できない** |
| 2 | 🔴 バリデーションの散在 | 「メールアドレスに `@` が含まれるか」等のチェックが **使う側のあちこちにコピペ** される |
| 3 | 🔴 不正値の混入 | `quantity = -5` や `price = 0` が素通りしてしまう |

---

### 🌟 改善版コード

**一言でいうと:** `String` や `int` の代わりに **`Email`, `Quantity`, `Money`** などの専用クラスを作る。各クラスが自分のバリデーションを持つ。

**Java:**
```java
// ✅ 値オブジェクト：概念ごとに専用クラスを作る
class Email {
    private final String value;

    Email(final String value) {
        if (value == null || !value.contains("@")) {
            throw new IllegalArgumentException("有効なメールアドレスを指定してください。");
        }
        this.value = value;
    }

    String getValue() { return value; }
}

class Quantity {
    private final int value;

    Quantity(final int value) {
        if (value < 1) {
            throw new IllegalArgumentException("数量は1以上を指定してください。");
        }
        this.value = value;
    }

    int getValue() { return value; }
}

// ✅ 引数の型が全て異なるため、順番を間違えるとコンパイルエラー！
class Order {
    private final String customerName;
    private final Email customerEmail;       // ← String ではなく Email 型
    private final Quantity quantity;          // ← int ではなく Quantity 型
    private final Money price;               // ← int ではなく Money 型

    Order(final String name, final Email email, final Quantity qty, final Money price) {
        this.customerName = name;
        this.customerEmail = email;
        this.quantity = qty;
        this.price = price;
    }
}
```

📝 **使い方:**
```java
Email email   = new Email("taro@example.com");  // ✅ バリデーション済み
Quantity qty  = new Quantity(3);                  // ✅ 1以上が保証されている
Money price   = new Money(5000, jpy);            // ✅ 0以上 & 通貨必須

Order order = new Order("太郎", email, qty, price);
// Order bad = new Order(email, "太郎", price, qty); // ❌ 型が違うのでコンパイルエラー！
```

**Python:**
```python
# ✅ 値オブジェクト
class Email:
    def __init__(self, value: str) -> None:
        if not value or "@" not in value:
            raise ValueError("有効なメールアドレスを指定してください。")
        self._value: str = value

    @property
    def value(self) -> str:
        return self._value

class Quantity:
    def __init__(self, value: int) -> None:
        if value < 1:
            raise ValueError("数量は1以上を指定してください。")
        self._value: int = value

    @property
    def value(self) -> int:
        return self._value

class Order:
    def __init__(self, name: str, email: Email, quantity: Quantity, price: 'Money') -> None:
        self._customer_name: str = name
        self._customer_email: Email = email
        self._quantity: Quantity = quantity
        self._price: 'Money' = price
```

**TypeScript:**
```typescript
// ✅ 値オブジェクト
class Email {
    readonly value: string;
    constructor(value: string) {
        if (!value || !value.includes("@")) throw new Error("有効なメールアドレスを指定してください。");
        this.value = value;
    }
}

class Quantity {
    readonly value: number;
    constructor(value: number) {
        if (value < 1) throw new Error("数量は1以上を指定してください。");
        this.value = value;
    }
}

class Order {
    constructor(
        readonly customerName: string,
        readonly customerEmail: Email,
        readonly quantity: Quantity,
        readonly price: Money
    ) {}
}
```

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| メールアドレスの型 | `String` | `Email`（`@` チェック内蔵） |
| 数量の型 | `int` | `Quantity`（1以上を保証） |
| 引数の取り違え | 全部同じ型なのでエラーにならない | **型が異なるためコンパイルエラー** で即座に検出 |

<br>

---

## 3. ストラテジパターン（Strategy Pattern）

> 💡 **用語: ストラテジパターン（Strategy Pattern）とは？**
> 処理の「やり方（戦略）」をインターフェースとして切り出し、 **実行時に差し替えられる** ようにする設計パターン。`if-else` で分岐する代わりに、クラスを差し替えることで振る舞いを変える。

### ❌ 悪い例（if-else で配送料を分岐）

**Java:**
```java
// ❌ 配送方法が増えるたびに if-else が膨らむ
class ShippingCalculator {
    int calculate(String type, int weight) {
        if (type.equals("standard")) {
            return weight * 100;
        } else if (type.equals("express")) {
            return weight * 300 + 500;
        } else if (type.equals("overnight")) {
            return weight * 500 + 1000;
        }
        // 新しい配送方法 → ここに else if を追加... 既存コードに触る！
        return 0;
    }
}
```

**Python:**
```python
# ❌ 配送方法が増えるたびに if-else が膨らむ
class ShippingCalculator:
    def calculate(self, shipping_type: str, weight: int) -> int:
        if shipping_type == "standard":
            return weight * 100
        elif shipping_type == "express":
            return weight * 300 + 500
        elif shipping_type == "overnight":
            return weight * 500 + 1000
        return 0
```

**TypeScript:**
```typescript
// ❌ 配送方法が増えるたびに if-else が膨らむ
class ShippingCalculator {
    calculate(type: string, weight: number): number {
        if (type === "standard") return weight * 100;
        else if (type === "express") return weight * 300 + 500;
        else if (type === "overnight") return weight * 500 + 1000;
        return 0;
    }
}
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 開放閉鎖原則違反 | 新しい配送方法を追加するたびに **既存の `if-else` を修正** しなければならない |
| 2 | 🔴 文字列ミスに気づけない | `"standrd"` とタイプミスしても **コンパイルエラーにならない** |
| 3 | 🔴 テストしにくい | 全パターンが1メソッドに詰め込まれ、各配送方法を **独立してテストできない** |

> 💡 **用語: 開放閉鎖原則（Open-Closed Principle / OCP）とは？**
> 「機能の **追加** には開いて（Open）いるが、既存コードの **修正** には閉じて（Closed）いる」設計原則。新機能を追加するとき、既存コードを書き換えずに済む設計が理想。

---

### 🌟 改善版コード

**一言でいうと:** 配送方法ごとにクラスを作り、インターフェースで統一する。新しい配送方法は **クラスを1つ追加するだけ** 。

**Java:**
```java
// ✅ 配送料計算のルールブック（インターフェース）
interface ShippingStrategy {
    int calculate(int weight);
}

class StandardShipping implements ShippingStrategy {
    public int calculate(int weight) { return weight * 100; }
}

class ExpressShipping implements ShippingStrategy {
    public int calculate(int weight) { return weight * 300 + 500; }
}

// 使う側は「どの配送方法か」を気にしない
class Order {
    private final ShippingStrategy shipping;

    Order(final ShippingStrategy shipping) {
        this.shipping = shipping;
    }

    int shippingCost(int weight) {
        return shipping.calculate(weight);  // どの戦略でも同じコードで動く
    }
}
```

📝 **使い方:**
```java
Order standard = new Order(new StandardShipping());
standard.shippingCost(5); // 500円

Order express = new Order(new ExpressShipping());
express.shippingCost(5);  // 2000円

// 🎉 新しい配送方法の追加 → クラスを1つ作るだけ！既存コード変更なし！
class DroneShipping implements ShippingStrategy {
    public int calculate(int weight) { return weight * 800 + 2000; }
}
```

**Python:**
```python
from abc import ABC, abstractmethod

class ShippingStrategy(ABC):
    @abstractmethod
    def calculate(self, weight: int) -> int:
        pass

class StandardShipping(ShippingStrategy):
    def calculate(self, weight: int) -> int:
        return weight * 100

class ExpressShipping(ShippingStrategy):
    def calculate(self, weight: int) -> int:
        return weight * 300 + 500

class Order:
    def __init__(self, shipping: ShippingStrategy) -> None:
        self._shipping: ShippingStrategy = shipping

    def shipping_cost(self, weight: int) -> int:
        return self._shipping.calculate(weight)
```

**TypeScript:**
```typescript
interface ShippingStrategy {
    calculate(weight: number): number;
}

class StandardShipping implements ShippingStrategy {
    calculate(weight: number): number { return weight * 100; }
}

class ExpressShipping implements ShippingStrategy {
    calculate(weight: number): number { return weight * 300 + 500; }
}

class Order {
    constructor(private readonly shipping: ShippingStrategy) {}

    shippingCost(weight: number): number {
        return this.shipping.calculate(weight);
    }
}
```

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| 新しい配送方法の追加 | 既存の `if-else` を修正 | クラスを1つ追加するだけ |
| タイプミスのリスク | 文字列ミスに気づけない | クラス名のミスはコンパイルエラー |
| テスト | 全分岐を1メソッドでテスト | 各クラスを独立してテスト可能 |

<br>

---

## 4. ポリシーパターン（Policy Pattern）

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

---

## 5. ファーストクラスコレクション（First Class Collection）

> 💡 **用語: ファーストクラスコレクション（First Class Collection）とは？**
> コレクション（リストや配列）を **そのまま使わず、専用のクラスで包む** 設計パターン。コレクションに対する操作（追加・削除・検索・集計）を全てそのクラスの中に閉じ込める。

### ❌ 悪い例（コレクションをむき出しで使う）

**Java:**
```java
// ❌ List をむき出しで使っている → 操作ルールが散在する
class Party {
    List<Member> members = new ArrayList<>();
}
```

```java
// 使う側のコード（あちこちに同じチェックが散らばる）
Party party = new Party();

// パーティの上限チェック → 使う側が毎回書く
if (party.members.size() < 4) {
    party.members.add(newMember);
}

// 別の場所でも同じチェック... 書き忘れたらバグ！
if (party.members.size() < 4) {
    party.members.add(anotherMember);
}

// ❌ 外部から直接操作できてしまう
party.members.clear();  // パーティ全員消去！ルール無視！
```

**Python:**
```python
# ❌ リストをむき出しで使っている
class Party:
    def __init__(self) -> None:
        self.members: list['Member'] = []

# 使う側が毎回上限チェックする必要がある
party = Party()
if len(party.members) < 4:
    party.members.append(new_member)

party.members.clear()  # ❌ ルール無視で全消去可能
```

**TypeScript:**
```typescript
// ❌ 配列をむき出しで使っている
class Party {
    members: Member[] = [];
}

const party = new Party();
if (party.members.length < 4) {
    party.members.push(newMember);
}
party.members.length = 0; // ❌ ルール無視で全消去可能
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 ルールの散在 | 「パーティは最大4人」等のルールが **使う側のあちこちにコピペ** される。書き忘れたら上限を超えた不正状態に |
| 2 | 🔴 外部からの不正操作 | `members.clear()` や `members.remove()` が自由にできてしまい、 **クラスが知らないうちに中身が変わる** |
| 3 | 🔴 集計ロジックの重複 | 「HPが最も低いメンバーを探す」等の処理が、使う箇所ごとに重複して書かれる |

---

### 🌟 改善版コード

**一言でいうと:** `List<Member>` を直接公開せず、 **`Party` クラスの中にコレクションを隠蔽** し、操作は全てメソッド経由にする。

**Java:**
```java
// ✅ ファーストクラスコレクション：コレクションを専用クラスで包む
class Party {
    private static final int MAX_SIZE = 4;
    private final List<Member> members;

    Party() {
        this.members = new ArrayList<>();
    }

    // ✅ 追加ルールをクラスの中に閉じ込める
    void add(final Member member) {
        if (members.size() >= MAX_SIZE) {
            throw new IllegalStateException("パーティは最大" + MAX_SIZE + "人です。");
        }
        members.add(member);
    }

    // ✅ 読み取り専用のコピーを返す（外部からの直接操作を防ぐ）
    List<Member> getMembers() {
        return Collections.unmodifiableList(members);
    }

    // ✅ 集計ロジックもクラス内に閉じ込める
    Member lowestHpMember() {
        return members.stream()
            .min(Comparator.comparingInt(Member::getHp))
            .orElseThrow();
    }

    int size() { return members.size(); }
    boolean isEmpty() { return members.isEmpty(); }
}
```

📝 **使い方:**
```java
Party party = new Party();
party.add(warrior);    // ✅ ルールチェック済みで追加
party.add(mage);
party.add(healer);
party.add(thief);
// party.add(extra);   // ❌ IllegalStateException: パーティは最大4人です。

Member weakest = party.lowestHpMember(); // ✅ 集計もクラスに任せる

// party.getMembers().clear();  // ❌ UnsupportedOperationException（変更不可！）
```

**Python:**
```python
# ✅ ファーストクラスコレクション
class Party:
    MAX_SIZE: int = 4

    def __init__(self) -> None:
        self._members: list['Member'] = []

    def add(self, member: 'Member') -> None:
        if len(self._members) >= self.MAX_SIZE:
            raise ValueError(f"パーティは最大{self.MAX_SIZE}人です。")
        self._members.append(member)

    @property
    def members(self) -> tuple['Member', ...]:
        """読み取り専用（タプルで返すことで外部からの変更を防ぐ）"""
        return tuple(self._members)

    def lowest_hp_member(self) -> 'Member':
        if not self._members:
            raise ValueError("パーティが空です。")
        return min(self._members, key=lambda m: m.hp)

    def size(self) -> int:
        return len(self._members)
```

**TypeScript:**
```typescript
// ✅ ファーストクラスコレクション
class Party {
    private static readonly MAX_SIZE = 4;
    private readonly members: Member[] = [];

    add(member: Member): void {
        if (this.members.length >= Party.MAX_SIZE) {
            throw new Error(`パーティは最大${Party.MAX_SIZE}人です。`);
        }
        this.members.push(member);
    }

    getMembers(): readonly Member[] {
        return [...this.members]; // コピーを返す
    }

    lowestHpMember(): Member {
        if (this.members.length === 0) throw new Error("パーティが空です。");
        return this.members.reduce((min, m) => m.hp < min.hp ? m : min);
    }

    get size(): number { return this.members.length; }
}
```

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| 上限チェック | 使う側が毎回 `if (size < 4)` を書く | `add()` メソッドが自動でチェック |
| 外部からの操作 | `members.clear()` で勝手に消去可能 | 読み取り専用コピーを返すので変更不可 |
| 集計ロジック | 使う箇所ごとにコピペ | `lowestHpMember()` 等のメソッドに集約 |

<br>

---

## 6. スプラウトクラス（Sprout Class）

> 💡 **用語: スプラウトクラス（Sprout Class）とは？**
> 既存の巨大なクラスに新機能を追加したい時、既存クラスを直接修正するのではなく、 **新しい小さなクラス（芽＝スプラウト）を別に作って** そこに新機能を実装する手法。既存コードを壊すリスクをゼロにしながら機能追加できる。

### ❌ 悪い例（既存の巨大クラスに直接追加）

**Java:**
```java
// ❌ 既に500行以上ある巨大クラスに新機能を直接追加
class OrderService {
    // ... 既存のコード500行 ...

    void processOrder(Order order) { /* 既存の複雑な処理 */ }
    void cancelOrder(Order order) { /* 既存の複雑な処理 */ }

    // ↓ 新しい「ギフトラッピング機能」を直接追加してしまう
    void applyGiftWrapping(Order order, String message) {
        // ギフト用バリデーション
        if (message == null || message.length() > 200) {
            throw new IllegalArgumentException("メッセージは200文字以内");
        }
        // ラッピング料金計算
        int wrappingFee = order.getItemCount() * 300;
        order.addFee(wrappingFee);
        order.setGiftMessage(message);
        // ← 既存の500行のコードと混ざり合い、どんどん肥大化する
    }
}
```

**Python:**
```python
# ❌ 巨大クラスに直接追加
class OrderService:
    # ... 既存コード500行 ...

    def apply_gift_wrapping(self, order: 'Order', message: str) -> None:
        if not message or len(message) > 200:
            raise ValueError("メッセージは200文字以内")
        wrapping_fee: int = order.item_count * 300
        order.add_fee(wrapping_fee)
        order.gift_message = message
```

**TypeScript:**
```typescript
// ❌ 巨大クラスに直接追加
class OrderService {
    // ... 既存コード500行 ...

    applyGiftWrapping(order: Order, message: string): void {
        if (!message || message.length > 200) throw new Error("メッセージは200文字以内");
        const wrappingFee = order.itemCount * 300;
        order.addFee(wrappingFee);
        order.giftMessage = message;
    }
}
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 既存コードを壊すリスク | 500行のクラスに手を入れると、 **既に動いている処理に予期しない影響** を与える可能性がある |
| 2 | 🔴 クラスの肥大化 | 機能が追加されるたびにクラスが膨らみ、 **理解・テスト・修正が困難** になっていく |
| 3 | 🔴 単一責任原則の違反 | 「注文処理」と「ギフトラッピング」は別の関心事なのに **1つのクラスに混在** している |

> 💡 **用語: 単一責任原則（Single Responsibility Principle / SRP）とは？**
> 1つのクラスは **1つの責任（理由）だけ** を持つべきという設計原則。「変更する理由が2つ以上あるクラスは分割すべき」とも言い換えられる。

---

### 🌟 改善版コード

**一言でいうと:** 新機能を **独立した小さなクラス（スプラウトクラス）** として作り、既存クラスから呼び出す。既存クラスのコードには最小限の変更（1〜2行の呼び出し追加）だけで済む。

**Java:**
```java
// ✅ スプラウトクラス：ギフトラッピングの責任だけを持つ新クラス
class GiftWrapping {
    private static final int MAX_MESSAGE_LENGTH = 200;
    private static final int FEE_PER_ITEM = 300;

    private final String message;
    private final int fee;

    GiftWrapping(final String message, final int itemCount) {
        if (message == null || message.length() > MAX_MESSAGE_LENGTH) {
            throw new IllegalArgumentException(
                "メッセージは" + MAX_MESSAGE_LENGTH + "文字以内です。"
            );
        }
        if (itemCount < 1) {
            throw new IllegalArgumentException("商品数は1以上です。");
        }
        this.message = message;
        this.fee = itemCount * FEE_PER_ITEM;
    }

    String getMessage() { return message; }
    int getFee() { return fee; }
}

// ✅ 既存クラスには最小限の変更だけ（1行追加）
class OrderService {
    // ... 既存の500行はそのまま触らない ...

    void applyGiftWrapping(Order order, String message) {
        // 新クラスに処理を委譲する（既存コードへの影響はこの1行だけ）
        GiftWrapping gift = new GiftWrapping(message, order.getItemCount());
        order.addFee(gift.getFee());
        order.setGiftMessage(gift.getMessage());
    }
}
```

📝 **使い方:**
```java
// ✅ GiftWrapping は独立してテスト可能！
GiftWrapping gift = new GiftWrapping("お誕生日おめでとう！", 3);
System.out.println(gift.getFee());     // 900円
System.out.println(gift.getMessage()); // "お誕生日おめでとう！"

// ❌ バリデーションも GiftWrapping が担当
// new GiftWrapping(null, 0);  // → 例外！OrderService を巻き込まない
```

**Python:**
```python
# ✅ スプラウトクラス
class GiftWrapping:
    MAX_MESSAGE_LENGTH: int = 200
    FEE_PER_ITEM: int = 300

    def __init__(self, message: str, item_count: int) -> None:
        if not message or len(message) > self.MAX_MESSAGE_LENGTH:
            raise ValueError(f"メッセージは{self.MAX_MESSAGE_LENGTH}文字以内です。")
        if item_count < 1:
            raise ValueError("商品数は1以上です。")
        self._message: str = message
        self._fee: int = item_count * self.FEE_PER_ITEM

    @property
    def message(self) -> str:
        return self._message

    @property
    def fee(self) -> int:
        return self._fee

# ✅ 既存クラスからは委譲するだけ
class OrderService:
    # ... 既存コードはそのまま ...
    def apply_gift_wrapping(self, order: 'Order', message: str) -> None:
        gift = GiftWrapping(message, order.item_count)
        order.add_fee(gift.fee)
        order.gift_message = gift.message
```

**TypeScript:**
```typescript
// ✅ スプラウトクラス
class GiftWrapping {
    private static readonly MAX_MESSAGE_LENGTH = 200;
    private static readonly FEE_PER_ITEM = 300;

    readonly message: string;
    readonly fee: number;

    constructor(message: string, itemCount: number) {
        if (!message || message.length > GiftWrapping.MAX_MESSAGE_LENGTH) {
            throw new Error(`メッセージは${GiftWrapping.MAX_MESSAGE_LENGTH}文字以内です。`);
        }
        if (itemCount < 1) throw new Error("商品数は1以上です。");
        this.message = message;
        this.fee = itemCount * GiftWrapping.FEE_PER_ITEM;
    }
}

// ✅ 既存クラスからは委譲するだけ
class OrderService {
    // ... 既存コードはそのまま ...
    applyGiftWrapping(order: Order, message: string): void {
        const gift = new GiftWrapping(message, order.itemCount);
        order.addFee(gift.fee);
        order.giftMessage = gift.message;
    }
}
```

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| 既存コードへの影響 | 500行のクラスに直接追加 | 既存クラスは1〜2行の呼び出し追加のみ |
| テスト | 巨大クラス全体をテスト対象にする必要がある | `GiftWrapping` だけを独立してテスト可能 |
| 責任の分離 | 「注文処理」+「ギフト」が混在 | 各クラスが1つの責任だけを持つ |

<br>

---

### 🎯 まとめ（6つの設計パターン一覧）

| パターン | 解決する問題 | 核心 |
|---|---|---|
| 完全コンストラクタ | 生焼けオブジェクト | 生成時に **全チェック完了** 。セッター不要 |
| 値オブジェクト | プリミティブ型の取り違え | 概念ごとに **専用クラス** を作る |
| ストラテジ | `if-else` の肥大化 | 処理を **クラスに切り出して差し替え** 可能に |
| ポリシー | 条件判定の固定化 | 条件を **独立クラスにして組み合わせ** 自由 |
| ファーストクラスコレクション | コレクション操作の散在 | コレクションを **専用クラスで包んで隠蔽** |
| スプラウトクラス | 巨大クラスの肥大化 | 新機能は **別クラスとして芽吹かせる** |

> ※ これらのパターンは **組み合わせて使う** のが前提です。例えば「完全コンストラクタ + 値オブジェクト」は常にセットで使います。
