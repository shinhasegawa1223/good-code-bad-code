## 何十にもネストしたロジック (Deeply Nested Logic)

**Java:**
```java
class DiscountCalculator {
    double calculateDiscount(User user, Item item) {
        if (user != null) {
            if (user.isActive()) {
                if (item != null) {
                    if (item.isAvailable()) {
                        if (user.getMembershipLevel() == Membership.PREMIUM) {
                            return item.getPrice() * 0.8;
                        } else {
                            return item.getPrice() * 0.95;
                        }
                    } else {
                        return 0.0;
                    }
                } else {
                    throw new IllegalArgumentException("Item is null");
                }
            } else {
                return 0.0;
            }
        } else {
            throw new IllegalArgumentException("User is null");
        }
    }
}
```

**Python:**
```python
class DiscountCalculator:
    def calculate_discount(self, user, item):
        if user is not None:
            if user.is_active():
                if item is not None:
                    if item.is_available():
                        if user.membership_level == "PREMIUM":
                            return item.price * 0.8
                        else:
                            return item.price * 0.95
                    else:
                        return 0.0
                else:
                    raise ValueError("Item is null")
            else:
                return 0.0
        else:
            raise ValueError("User is null")
```

**typescript:**
```typescript
class DiscountCalculator {
    calculateDiscount(user: User | null, item: Item | null): number {
        if (user !== null) {
            if (user.isActive()) {
                if (item !== null) {
                    if (item.isAvailable()) {
                        if (user.membershipLevel === 'PREMIUM') {
                            return item.price * 0.8;
                        } else {
                            return item.price * 0.95;
                        }
                    } else {
                        return 0.0;
                    }
                } else {
                    throw new Error("Item is null");
                }
            } else {
                return 0.0;
            }
        } else {
            throw new Error("User is null");
        }
    }
}
```

---

### 🚨 何がダメなのか？ (ネストしたロジックの問題点)

記載されているコードには、以下の問題があります。

#### 1. 可読性の著しい低下 (Arrow Anti-Pattern)
> 対象: `if` 文が連なっている箇所全体

何重にも `if` 文が入れ子（ネスト）になっているため、コードの形状が右に向かって矢印（Arrow）のようになっています。
**「どの `if` に対する `else` なのか」** を視覚的に追うのが非常に困難であり、少し目を離しただけで現在の条件分岐の前提が何だったか分からなくなってしまいます。

#### 2. 本質的な処理（正常系）が埋もれる
> 対象: `return item.getPrice() * 0.8;` などの計算部分

このメソッドが本当にやりたいこと（正常系である割引計算）が、何重ものエラーチェックや事前条件チェックの奥底に埋もれてしまっています。
コードを読む際、一番重要なロジックにたどり着くまでに多くの事前条件を頭のメモリに保持し続けなければならず、理解に時間がかかります。

---

### 🌟 改善版コード（早期リターン / ガード節）

本質的な処理（正常系）のジャマをする「異常系（エラーや事前条件の不一致）」を、メソッドの先頭で **早期リターン（Early Return / Guard Clause）** して弾き、ネストを浅くします。

**Java:**
```java
class DiscountCalculator {
    double calculateDiscount(User user, Item item) {
        if (user == null) throw new IllegalArgumentException("User is null");
        if (item == null) throw new IllegalArgumentException("Item is null");
        
        if (!user.isActive() || !item.isAvailable()) {
            return 0.0;
        }

        if (user.getMembershipLevel() == Membership.PREMIUM) {
            return item.getPrice() * 0.8;
        }

        return item.getPrice() * 0.95;
    }
}
```

**Python:**
```python
class DiscountCalculator:
    def calculate_discount(self, user, item):
        if user is None: raise ValueError("User is null")
        if item is None: raise ValueError("Item is null")
        
        if not user.is_active() or not item.is_available():
            return 0.0

        if user.membership_level == "PREMIUM":
            return item.price * 0.8

        return item.price * 0.95
```

**typescript:**
```typescript
class DiscountCalculator {
    calculateDiscount(user: User | null, item: Item | null): number {
        if (user === null) throw new Error("User is null");
        if (item === null) throw new Error("Item is null");
        
        if (!user.isActive() || !item.isAvailable()) {
            return 0.0;
        }

        if (user.membershipLevel === 'PREMIUM') {
            return item.price * 0.8;
        }

        return item.price * 0.95;
    }
}
```

#### 改善のポイント
- `if (user != null) { ... }` → **`if (user == null) throw ...`**: 条件を満たさない異常なケースを、波括弧の中に進ませるのではなく、先頭で即座にエラーとして弾くようにしました。
- `else` の徹底排除 → **早期リターンによるフラット化**: 全ての前提チェックが終わった状態で正常系のコードが最下層に書かれるため、一切ネスト深く潜らずに本質的な計算処理を読むことができます。

---

### 🎯 まとめ（ネスト排除における考え方）

深いネストから脱却し、読みやすいロジックを書くための考え方を「目的・目標・手段」の順に整理します。

#### 1. 目的（Why: なぜこれにこだわるのか？）
* **「認知負荷を下げるため」**
  * 人間の脳が同時に記憶できる前提条件の数には限界があります。
  * ネストが深いコードは「今どの条件を満たしているのか」を常に覚えながら読む必要があり、バグの温床になります。

#### 2. 目標（What: どのような状態を目指すのか？）
* **「正常系が一直線に読める状態」**
  * コードを上から下にスッと読んだ時、エラー処理や前提チェックなどに視界を遮られず、1番やりたいメイン処理が「インデントなし（浅いネスト）」で素直に書かれている状態を目指します。

#### 3. 手段（How: どうやって達成するのか？）
* **「ガード節（早期リターン）の活用」**
  * ❌ `if (条件OK) { 続く... }` → ⭕️ `if (条件NG) { return/throw }`
  * 処理の最初に「これ以上進めない条件」を見つけたらすぐにメソッドから脱出（`return`）させることで、後続のコードを `else` ブロックで囲む必要をなくし、ネストを浅く保つことができます。
