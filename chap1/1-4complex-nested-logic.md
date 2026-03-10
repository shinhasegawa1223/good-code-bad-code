## 複雑に絡み合ったネスト (Complex Nested Logic)

**Java:**
```java
if (conditionA) {
    if (conditionB) {
        if (conditionC) {
            // 処理1
        }
        if (conditionD) {
            // 処理2
        }
    }
    if (conditionE) {
        if (conditionF) {
            // 処理3
        }
        if (conditionG) {
            // 処理4
        }
    }
}
```

**Python:**
```python
if condition_a:
    if condition_b:
        if condition_c:
            pass # 処理1
        if condition_d:
            pass # 処理2
    if condition_e:
        if condition_f:
            pass # 処理3
        if condition_g:
            pass # 処理4
```

**TypeScript:**
```typescript
if (conditionA) {
    if (conditionB) {
        if (conditionC) {
            // 処理1
        }
        if (conditionD) {
            // 処理2
        }
    }
    if (conditionE) {
        if (conditionF) {
            // 処理3
        }
        if (conditionG) {
            // 処理4
        }
    }
}
```

---

### 🚨 何がダメなのか？ (複雑に絡み合ったネストの問題点)

<br>

#### 1. 🔴 条件の組み合わせが爆発 → 全ルートを網羅できない
> 対象: `if` の層が繰り返されている全体

| 疑問 | 答えられる？ |
|---|---|
| 処理1が実行される条件は？ | `A && B && C` …多分… 🤔 |
| 処理3が実行されるのは？ | `A && E && F` …合ってる？ 😰 |
| 全パターンのテストは？ | **頭の中だけでは不可能** 💀 |

→ 意図しない条件の組み合わせで **バグが発生しやすい** 。

<br>

#### 2. 🔴 スコープや前提が視覚的に分かりづらい
> 対象: `conditionE` のブロック周辺

`conditionB` や `conditionE` には、外側の `conditionA` が真という **暗黙の前提** がある。

→ コードを少し読み進めるだけで **「今は何の前提の中？」を見失う** 。

---

### 🌟 改善版コード（条件の隠蔽 / メソッドへの抽出）

> 💡 **用語: Extract Method（メソッドの抽出）とは？**
> 複雑な条件式を、意味のある名前をつけた別のメソッド（関数）としてくくり出す手法。
> 可読性・再利用性・テスト容易性が **大幅に向上** する。

**一言でいうと:** 条件を **「名前付きメソッド」に変換** → `if` 文が英文のように読める。

**Java:**
```java
// 例えばログインや権限確認などのバラバラの条件があるとする
boolean hasValidId = ...;
boolean hasCorrectPassword = ...;
boolean isNotBanned = ...;
boolean hasPaidSubscription = ...;

// 🧩 条件をメソッドにまとめ、「ビジネスルール」を名前で語らせる
if (isValidUser(hasValidId, hasCorrectPassword, isNotBanned)) {
    // 処理1（例：ログイン成功処理）
}

if (isPremiumMember(hasValidId, hasCorrectPassword, hasPaidSubscription)) {
    // 処理2（例：優先クーポンの表示処理）
}

// 📦 抽出されたメソッド
// 「IDが正しくて、パスワードも正しくて、BANされてない」→「正当なユーザー」
private boolean isValidUser(boolean hasValidId, boolean hasCorrectPassword, boolean isNotBanned) {
    return hasValidId && hasCorrectPassword && isNotBanned;
}
```

**Python:**
```python
# 例えばログインや権限確認などのバラバラの条件があるとする
has_valid_id: bool = ...
has_correct_password: bool = ...
is_not_banned: bool = ...
has_paid_subscription: bool = ...

# 🧩 条件をメソッドにまとめ、「ビジネスルール」を名前で語らせる
if is_valid_user(has_valid_id, has_correct_password, is_not_banned):
    pass # 処理1（例：ログイン成功処理）

if is_premium_member(has_valid_id, has_correct_password, has_paid_subscription):
    pass # 処理2（例：優先クーポンの表示処理）

# 📦 抽出されたメソッド
# 「IDが正しくて、パスワードも正しくて、BANされてない」→「正当なユーザー」
def is_valid_user(has_valid_id: bool, has_correct_password: bool, is_not_banned: bool) -> bool:
    return has_valid_id and has_correct_password and is_not_banned
```

**TypeScript:**
```typescript
// 例えばログインや権限確認などのバラバラの条件があるとする
let hasValidId = ...;
let hasCorrectPassword = ...;
let isNotBanned = ...;
let hasPaidSubscription = ...;

// 🧩 条件をメソッドにまとめ、「ビジネスルール」を名前で語らせる
if (isValidUser(hasValidId, hasCorrectPassword, isNotBanned)) {
    // 処理1（例：ログイン成功処理）
}

if (isPremiumMember(hasValidId, hasCorrectPassword, hasPaidSubscription)) {
    // 処理2（例：優先クーポンの表示処理）
}

// 📦 抽出されたメソッド
// 「IDが正しくて、パスワードも正しくて、BANされてない」→「正当なユーザー」
function isValidUser(hasValidId: boolean, hasCorrectPassword: boolean, isNotBanned: boolean): boolean {
    return hasValidId && hasCorrectPassword && isNotBanned;
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| **条件式** | `if (A) { if (B) { if (C) {...}}}` | `if (isValidUser(...))` |
| **読み方** | `&&` の中身を1つずつ解読 | メソッド名を読むだけで **意図が分かる** |
| **前提の追跡** | 外側の `if` を常に覚えておく必要 | 各 `if` が **独立** している |

---

### 🎯 まとめ（条件式カプセル化における考え方）

#### 1. 目的（Why）
> **「状態の意図（ビジネスルール）を明確にするため」**

* `&&` で繋ぐだけでは **「なぜこの組み合わせが必要なのか」** という理由（仕様）がコードから消える

#### 2. 目標（What）
> **「`if` 文が自然言語（英文）のように読める状態」**

* ❌ 「フラグがtrueで、数値が0以上なら…」（仕組みの解読）
* ⭕️ 「プレミアム会員なら…」（ **意味** が即座に分かる）

#### 3. 手段（How）
> **「名前のない条件式」に「名前」を与える**

| アプローチ | 使いどき | 例 |
|:---:|---|---|
| **① 説明変数** | 条件が1箇所で使われる場合 | `boolean isAdult = (age >= 20);` `if (isAdult)` |
| **② メソッド抽出** | 条件が複数箇所で使われる場合 | `if (isPremiumUser(user))` |