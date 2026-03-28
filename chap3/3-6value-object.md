## 値オブジェクト（Value Object）

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
