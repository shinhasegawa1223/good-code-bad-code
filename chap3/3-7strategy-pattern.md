## ストラテジパターン（Strategy Pattern）

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
