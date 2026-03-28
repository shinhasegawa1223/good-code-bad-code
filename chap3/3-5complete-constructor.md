## 完全コンストラクタ（Complete Constructor）

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
