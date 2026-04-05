## 不変と可変の使い分け ― デフォルトは不変、可変は正しく設計する

> 🎯 **この章のポイント:**
> 4-1、4-2で「不変が大切」と学びましたが、 **全てを不変にできるわけではありません** 。
> HPのように **値が変わること自体がゲームの仕様** である場合、可変にする必要があります。
> 大切なのは「デフォルトは不変」「可変にする場合は正しく設計する」という使い分けです。

### なぜデフォルトは不変にすべきなのか？

| メリット | 説明 |
|---|---|
| 📌 変数の意味が変化しない | 一度セットした値がコードのどこで読んでも同じ。「今いくつ？」と悩まない |
| 📌 挙動が安定し、結果を予測しやすい | 副作用がないため、メソッドの入出力だけ見れば動作が分かる |
| 📌 影響範囲が限定的で保守しやすい | 変更が波及しないので、修正時に「他に影響ないかな？」と心配する必要がない |

> 💡 **原則:** 迷ったら **不変** にする。可変が必要だと明確に判断できた場合のみ、可変にする。

---

### 可変が必要なケース ― ミューテーターの正しい設計

> 💡 **用語: ミューテーター（Mutator）とは？**
> オブジェクトの **状態を変更するメソッド** のこと。例えば `damage()` でHPを減らすメソッドがミューテーター。可変にする場合は、ミューテーターの設計を正しく行い、 **不正な状態にならないよう** クラス内でルールを守る必要がある。

RPGのHPのように「ダメージを受けたらHPが減る」場合、HPは可変にせざるを得ません。しかし、 **「可変にする＝何でもアリ」ではありません** 。

<br>

#### ❌ 悪い例（ルールなしの可変）

**Java:**
```java
// ❌ HP は可変だが、ルールが一切ない
class HitPoint {
    int amount;  // ← 誰でも自由に書き換え可能
}

class Member {
    final HitPoint hitPoint;
    final States states;

    void damage(int damageAmount) {
        hitPoint.amount -= damageAmount;
        // ❌ HP がマイナスになってもチェックしない！
        // ❌ HP が 0 になっても「死亡」状態にならない！
    }
}
```

**Python:**
```python
# ❌ HP は可変だが、ルールが一切ない
class HitPoint:
    def __init__(self) -> None:
        self.amount: int = 0  # ← 誰でも自由に書き換え可能

class Member:
    def __init__(self) -> None:
        self.hit_point: HitPoint = HitPoint()
        self.states: States = States()

    def damage(self, damage_amount: int) -> None:
        self.hit_point.amount -= damage_amount
        # ❌ HP がマイナスになってもチェックしない！
```

**TypeScript:**
```typescript
// ❌ HP は可変だが、ルールが一切ない
class HitPoint {
    amount: number = 0; // ← 誰でも自由に書き換え可能
}

class Member {
    readonly hitPoint = new HitPoint();
    readonly states = new States();

    damage(damageAmount: number): void {
        this.hitPoint.amount -= damageAmount;
        // ❌ HP がマイナスになってもチェックしない！
    }
}
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 HPがマイナスになる | `hitPoint.amount -= damageAmount` だけでは、HPが **0を下回ってマイナス** になる。HP -50 のキャラクターは意味不明 |
| 2 | 🔴 状態遷移が漏れる | HPが0になっても **「死亡」状態に遷移する処理がない** 。HPゼロなのに動き続けるゾンビバグが発生する |
| 3 | 🔴 外部から直接書き換え可能 | `hitPoint.amount = 99999` のように外部から自由にHPを操作できてしまう |

---

### 🌟 改善版コード（ミューテーターを正しく設計する）

**一言でいうと:** HPを可変にする場合でも、① **バリデーション（0未満にならない）** 、② **状態遷移（0なら死亡）** をクラス内に閉じ込める。外部からは `damage()` メソッド経由でしか操作できないようにする。

**Java:**
```java
// ✅ 可変だが、ルールをクラス内に閉じ込めた HitPoint
class HitPoint {
    private static final int MIN = 0;
    int amount;  // ← 可変（HPは変わる必要がある）

    HitPoint(final int amount) {
        if (amount < MIN) {
            throw new IllegalArgumentException();
        }
        this.amount = amount;
    }

    // ✅ ミューテーター：HPを減らすが、0未満にはならないようガード
    void damage(final int damageAmount) {
        final int nextAmount = amount - damageAmount;
        amount = Math.max(MIN, nextAmount);  // ← 最低でも0で止まる
    }

    // ✅ HPがゼロかどうかの判定もクラス内に持つ
    boolean isZero() {
        return amount == MIN;
    }
}

// ✅ Member は HitPoint の damage() を呼ぶだけ。ルールは HitPoint が守る
class Member {
    final HitPoint hitPoint;
    final States states;

    Member() {
        hitPoint = new HitPoint(100);
        states = new States();
    }

    // ✅ ダメージ処理 + 状態遷移を1メソッドに集約
    void damage(final int damageAmount) {
        hitPoint.damage(damageAmount);       // HP を減らす
        if (hitPoint.isZero()) {
            states.add(StateType.dead);      // HP が 0 なら死亡状態に遷移
        }
    }
}
```

📝 **使い方:**
```java
Member hero = new Member();                  // HP = 100
hero.damage(30);                              // HP = 70
System.out.println(hero.hitPoint.amount);     // → 70

hero.damage(80);                              // HP = 0（-10 にはならず 0 で止まる）
System.out.println(hero.hitPoint.amount);     // → 0
System.out.println(hero.states.contains(StateType.dead)); // → true（死亡状態！）
```

**Python:**
```python
# ✅ 可変だが、ルールをクラス内に閉じ込めた HitPoint
class HitPoint:
    MIN: int = 0

    def __init__(self, amount: int) -> None:
        if amount < self.MIN:
            raise ValueError()
        self._amount: int = amount

    @property
    def amount(self) -> int:
        return self._amount

    # ✅ ミューテーター：0未満にはならないようガード
    def damage(self, damage_amount: int) -> None:
        next_amount: int = self._amount - damage_amount
        self._amount = max(self.MIN, next_amount)

    def is_zero(self) -> bool:
        return self._amount == self.MIN

class Member:
    def __init__(self) -> None:
        self.hit_point: HitPoint = HitPoint(100)
        self.states: States = States()

    def damage(self, damage_amount: int) -> None:
        self.hit_point.damage(damage_amount)
        if self.hit_point.is_zero():
            self.states.add(StateType.DEAD)
```

📝 **使い方:**
```python
hero = Member()              # HP = 100
hero.damage(30)              # HP = 70
print(hero.hit_point.amount) # → 70

hero.damage(80)              # HP = 0（マイナスにならない）
print(hero.hit_point.amount) # → 0
print(hero.states.contains(StateType.DEAD))  # → True
```

**TypeScript:**
```typescript
// ✅ 可変だが、ルールをクラス内に閉じ込めた HitPoint
class HitPoint {
    private static readonly MIN = 0;
    private _amount: number; // ← 可変（private で外部からの直接変更を防止）

    constructor(amount: number) {
        if (amount < HitPoint.MIN) throw new Error();
        this._amount = amount;
    }

    get amount(): number { return this._amount; }

    // ✅ ミューテーター：0未満にはならないようガード
    damage(damageAmount: number): void {
        const nextAmount = this._amount - damageAmount;
        this._amount = Math.max(HitPoint.MIN, nextAmount);
    }

    isZero(): boolean {
        return this._amount === HitPoint.MIN;
    }
}

class Member {
    readonly hitPoint = new HitPoint(100);
    readonly states = new States();

    damage(damageAmount: number): void {
        this.hitPoint.damage(damageAmount);
        if (this.hitPoint.isZero()) {
            this.states.add(StateType.Dead);
        }
    }
}
```

📝 **使い方:**
```typescript
const hero = new Member();           // HP = 100
hero.damage(30);                     // HP = 70
console.log(hero.hitPoint.amount);   // → 70

hero.damage(80);                     // HP = 0（マイナスにならない）
console.log(hero.hitPoint.amount);   // → 0
console.log(hero.states.contains(StateType.Dead)); // → true
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| HPの下限 | マイナスになっても放置 | `Math.max(0, ...)` で **0未満にならない** | ゲーム上ありえないHPが防がれた |
| 状態遷移 | HP0でも何も起きない | `isZero()` チェックで **自動的に死亡状態** へ | 状態遷移の漏れがなくなった |
| 外部からの操作 | `hitPoint.amount = 99999` が可能 | `damage()` メソッド経由のみ | **不正な操作がブロック** された |

---

### 🗄️ コード外とのやり取りは局所化する ― リポジトリパターン

> 💡 **用語: リポジトリパターン（Repository Pattern）とは？**
> **データベースやファイルなどの外部ストレージへのアクセス** を、専用のクラス（リポジトリ）に閉じ込める設計パターン。ビジネスロジック側は「データの保存先がDBなのかファイルなのかAPI なのか」を知る必要がなくなる。

不変にできない典型的な例として、 **データベースへの読み書き（I/O操作）** があります。I/O操作は副作用そのものですが、 **影響範囲を局所化** することでリスクを最小限に抑えられます。

#### ❌ 悪い例（ビジネスロジックにDB操作が混在）

**Java:**
```java
// ❌ ビジネスロジックの中にSQL文が直接書かれている
class UserService {
    void updateUserName(int userId, String newName) {
        // ビジネスロジックとDB操作がごちゃ混ぜ
        if (newName == null || newName.isEmpty()) {
            throw new IllegalArgumentException("名前は必須です。");
        }
        // ❌ SQL が直接ここに書かれている
        String sql = "UPDATE users SET name = ? WHERE id = ?";
        database.execute(sql, newName, userId);
        // ❌ さらにログをファイルに書く処理まで混在
        logger.write("ユーザー名を更新: " + userId);
    }
}
```

**Python:**
```python
# ❌ ビジネスロジックの中にDB操作が混在
class UserService:
    def update_user_name(self, user_id: int, new_name: str) -> None:
        if not new_name:
            raise ValueError("名前は必須です。")
        # ❌ SQL が直接ここに
        sql: str = "UPDATE users SET name = %s WHERE id = %s"
        self.database.execute(sql, new_name, user_id)
```

**TypeScript:**
```typescript
// ❌ ビジネスロジックの中にDB操作が混在
class UserService {
    async updateUserName(userId: number, newName: string): Promise<void> {
        if (!newName) throw new Error("名前は必須です。");
        // ❌ SQL が直接ここに
        await this.database.query("UPDATE users SET name = $1 WHERE id = $2", [newName, userId]);
    }
}
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 テストが困難 | ビジネスロジックをテストしたいだけなのに、 **本物のDBが必要** になる |
| 2 | 🔴 変更が困難 | DBをPostgreSQLからMongoDBに変えたい場合、 **全メソッドのSQL文を書き換える** 必要がある |
| 3 | 🔴 副作用の影響範囲が不明 | どのメソッドがDBを読み書きするか **コード全体を調べないと分からない** |

---

### 🌟 改善版コード（リポジトリパターンでI/Oを局所化）

**一言でいうと:** DB操作を **リポジトリクラスに閉じ込め** 、ビジネスロジックは「データの出し入れ」だけをリポジトリに依頼する。ビジネスロジックからSQLは完全に消える。

**Java:**
```java
// ✅ リポジトリ：DB操作だけを担当する専用クラス
interface UserRepository {
    User findById(int userId);
    void save(User user);
}

// ✅ 実際のDB操作はここだけに閉じ込める
class PostgresUserRepository implements UserRepository {
    public User findById(int userId) {
        String sql = "SELECT * FROM users WHERE id = ?";
        return database.query(sql, userId);  // SQL はここだけ！
    }

    public void save(User user) {
        String sql = "UPDATE users SET name = ? WHERE id = ?";
        database.execute(sql, user.getName(), user.getId());
    }
}

// ✅ ビジネスロジック：SQLを一切知らない。純粋なロジックだけ
class UserService {
    private final UserRepository repository;

    UserService(final UserRepository repository) {
        this.repository = repository;
    }

    void updateUserName(final int userId, final String newName) {
        if (newName == null || newName.isEmpty()) {
            throw new IllegalArgumentException("名前は必須です。");
        }
        User user = repository.findById(userId);  // リポジトリに依頼
        User updated = user.changeName(newName);   // ビジネスロジック
        repository.save(updated);                   // リポジトリに依頼
    }
}
```

📝 **使い方:**
```java
// 本番環境：実際のDBを使う
UserRepository repo = new PostgresUserRepository(database);
UserService service = new UserService(repo);
service.updateUserName(1, "太郎");

// テスト環境：DB不要！メモリ上のダミーを使える
UserRepository fakeRepo = new InMemoryUserRepository();
UserService testService = new UserService(fakeRepo);
testService.updateUserName(1, "テスト太郎"); // ✅ DBなしでテスト可能！
```

**Python:**
```python
from abc import ABC, abstractmethod

# ✅ リポジトリのインターフェース
class UserRepository(ABC):
    @abstractmethod
    def find_by_id(self, user_id: int) -> 'User':
        pass

    @abstractmethod
    def save(self, user: 'User') -> None:
        pass

# ✅ DB操作はここだけに閉じ込める
class PostgresUserRepository(UserRepository):
    def find_by_id(self, user_id: int) -> 'User':
        sql: str = "SELECT * FROM users WHERE id = %s"
        return self.database.query(sql, user_id)

    def save(self, user: 'User') -> None:
        sql: str = "UPDATE users SET name = %s WHERE id = %s"
        self.database.execute(sql, user.name, user.id)

# ✅ ビジネスロジック：SQLを一切知らない
class UserService:
    def __init__(self, repository: UserRepository) -> None:
        self._repository: UserRepository = repository

    def update_user_name(self, user_id: int, new_name: str) -> None:
        if not new_name:
            raise ValueError("名前は必須です。")
        user: 'User' = self._repository.find_by_id(user_id)
        updated: 'User' = user.change_name(new_name)
        self._repository.save(updated)
```

**TypeScript:**
```typescript
// ✅ リポジトリのインターフェース
interface UserRepository {
    findById(userId: number): Promise<User>;
    save(user: User): Promise<void>;
}

// ✅ DB操作はここだけに閉じ込める
class PostgresUserRepository implements UserRepository {
    async findById(userId: number): Promise<User> {
        return await this.db.query("SELECT * FROM users WHERE id = $1", [userId]);
    }
    async save(user: User): Promise<void> {
        await this.db.query("UPDATE users SET name = $1 WHERE id = $2", [user.name, user.id]);
    }
}

// ✅ ビジネスロジック：SQLを一切知らない
class UserService {
    constructor(private readonly repository: UserRepository) {}

    async updateUserName(userId: number, newName: string): Promise<void> {
        if (!newName) throw new Error("名前は必須です。");
        const user = await this.repository.findById(userId);
        const updated = user.changeName(newName);
        await this.repository.save(updated);
    }
}
```

📝 **使い方:**
```typescript
// 本番環境
const repo = new PostgresUserRepository(db);
const service = new UserService(repo);
await service.updateUserName(1, "太郎");

// テスト環境：DB不要！
const fakeRepo = new InMemoryUserRepository();
const testService = new UserService(fakeRepo);
await testService.updateUserName(1, "テスト太郎"); // ✅ DBなしでテスト可能！
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| SQL の場所 | ビジネスロジック内に直接記述 | `Repository` クラスに **完全に閉じ込め** | ビジネスロジックがDBの種類を知らなくて済む |
| テスト | 本物のDBが必要 | ダミーの `Repository` を注入するだけ | **DBなしで高速テスト** が可能に |
| DB変更 | 全メソッドのSQLを書き換え | `Repository` の実装クラスを差し替えるだけ | **既存のビジネスロジックは一切変更不要** |

---

### 🎯 まとめ（不変と可変の使い分け）

#### 1. 目的（Why）
> **「変更による影響を最小限に抑え、バグを防ぐため」**

* デフォルトは不変にすることで、副作用やインスタンス共有の問題を根本から排除する。
* 可変が必要な場合は、クラス内にルールを閉じ込めた **正しいミューテーター** を設計する。

#### 2. 目標（What）
> **「不変と可変が明確に区別され、可変部分が最小限に局所化されている状態」**

* 可変なのはどのクラスのどのフィールドか、が一目瞭然。それ以外は全て不変。

#### 3. 手段（How）
> **「デフォルト不変 → 可変は正しく設計 → I/Oは局所化」**

| ステップ | やること |
|:---:|---|
| まず不変 | 変数・フィールドはデフォルトで `final` / `readonly` / `const` にする |
| 可変が必要なら | HPのようにミューテーターを正しく設計（バリデーション + 状態遷移をクラス内に） |
| I/O操作は | リポジトリパターンで副作用を局所化し、ビジネスロジックから分離する |

> ※ 「不変にすべきか可変にすべきか迷ったら、 **まず不変にする** 。あとから可変に変えるのは簡単だが、可変を不変に直すのは大変」です。
