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

**typescript:**
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

記載されているコードには、以下の問題があります。

#### 1. 条件の組み合わせが爆発し、すべてのルートを網羅できない
> 対象: `if` の層が繰り返されている全体

`if` の中に複数の `if` が配置され、それが連鎖しているため「どの条件とどの条件が重なった時に処理1や処理3が実行されるのか」を全て把握することが不可能です。
頭の中だけでパターンの組み合わせ（状態空間）をテスト・想像しきれないため、意図しない条件の組み合わせでバグが発生しやすくなります。

#### 2. スコープや前提が視覚的に極めて分かりづらい
> 対象: `conditionE` のブロック周辺など 

`conditionB` や `conditionE` のブロックが、一番外側の `conditionA` が真であるという前提の上に成り立っていますが、コードを少し読み進めただけで「今は何の前提の中のif文なのか？」を見失ってしまいます。

---

### 🌟 改善版コード（条件の隠蔽 / メソッドへの抽出）

> 💡 **用語: Extract Method（メソッドの抽出）とは？**
> 複雑な条件式や処理の一部を、意味のある名前をつけた別のメソッド（関数）としてくくり出すリファクタリング手法。コードの可読性・再利用性・テスト容易性が大幅に向上する。

論理演算子（AND, OR）で長々と繋げるだけの `if (A && B && C)` もネストよりはマシですが、条件が複雑になると「なぜこの組み合わせで実行されるのか（ビジネスルール）」が分かりにくくなります。真のベストプラクティスは、**複雑な条件式そのものを意味のある名前のメソッドとして抽出（カプセル化）する**ことです。

**Java:**
```java
// 例えばログインや権限確認などのバラバラの条件（変数）があるとする
boolean hasValidId = ...;
boolean hasCorrectPassword = ...;
boolean isNotBanned = ...;
boolean hasPaidSubscription = ...;

// バラバラの複雑な条件をメソッド（関数）にまとめ、「何のための条件か（ビジネスルール）」を名前で語らせる
if (isValidUser(hasValidId, hasCorrectPassword, isNotBanned)) {
    // 処理1（例：ログイン成功処理）
}

if (isPremiumMember(hasValidId, hasCorrectPassword, hasPaidSubscription)) {
    // 処理2（例：優先クーポンの表示処理）
}

// 抽出されたメソッド
// （「IDが正しくて、パスワードも正しくて、BANされてない」なら「正当なユーザー」というルールを定義する）
private boolean isValidUser(boolean hasValidId, boolean hasCorrectPassword, boolean isNotBanned) {
    return hasValidId && hasCorrectPassword && isNotBanned;
}
```

**Python:**
```python
# 例えばログインや権限確認などのバラバラの条件（変数）があるとする
has_valid_id: bool = ...
has_correct_password: bool = ...
is_not_banned: bool = ...
has_paid_subscription: bool = ...

# バラバラの複雑な条件をメソッド（関数）にまとめ、「何のための条件か（ビジネスルール）」を名前で語らせる
if is_valid_user(has_valid_id, has_correct_password, is_not_banned):
    pass # 処理1（例：ログイン成功処理）

if is_premium_member(has_valid_id, has_correct_password, has_paid_subscription):
    pass # 処理2（例：優先クーポンの表示処理）

# 抽出されたメソッド
# （「IDが正しくて、パスワードも正しくて、BANされてない」なら「正当なユーザー」というルールを定義する）
def is_valid_user(has_valid_id: bool, has_correct_password: bool, is_not_banned: bool) -> bool:
    return has_valid_id and has_correct_password and is_not_banned
```

**TypeScript:**
```typescript
// 例えばログインや権限確認などのバラバラの条件（変数）があるとする
let hasValidId = ...;
let hasCorrectPassword = ...;
let isNotBanned = ...;
let hasPaidSubscription = ...;

// バラバラの複雑な条件をメソッド（関数）にまとめ、「何のための条件か（ビジネスルール）」を名前で語らせる
if (isValidUser(hasValidId, hasCorrectPassword, isNotBanned)) {
    // 処理1（例：ログイン成功処理）
}

if (isPremiumMember(hasValidId, hasCorrectPassword, hasPaidSubscription)) {
    // 処理2（例：優先クーポンの表示処理）
}

// 抽出されたメソッド
// （「IDが正しくて、パスワードも正しくて、BANされてない」なら「正当なユーザー」というルールを定義する）
function isValidUser(hasValidId: boolean, hasCorrectPassword: boolean, isNotBanned: boolean): boolean {
    return hasValidId && hasCorrectPassword && isNotBanned;
}
```

#### 改善のポイント
- `if (A && B && C)` → **`if (isValidUser(...))`**: ただ条件演算子で繋ぐだけでなく、その条件群が**「ビジネスとして（仕様上）どういう状態なのか」**を英語のメソッド名として命名（カプセル化）しました。
- ロジックとルールの分離: `if` 文を読むときに、わざわざ `&&` の中身を一つ一つ解読しなくても「この条件名を満たせば、この処理が行われる」という意図が自然言語的（英文のよう）に読めるようになりました。

---

### 🎯 まとめ（条件式カプセル化における考え方）

複雑なネストから脱却し、正しいコードを書くための考え方を「目的・目標・手段」の順に整理します。

#### 1. 目的（Why: なぜこれにこだわるのか？）
* **「状態の意図（ビジネスルール）を明確にするため」**
  * 条件が複雑になればなるほど、AND(`&&`)やOR(`||`)で繋ぐだけでは「なぜその条件の組み合わせが必要なのか」という理由（仕様）がコードから失われてしまうため。

#### 2. 目標（What: どのような状態を目指すのか？）
* **「`if`文が自然言語（英語の文章）のように読める状態」**
  * ある記述を読むとき、「このフラグがtrueで、かつあの数値が0以上なら」と条件の"仕組み"を解読するのではなく、「これがプレミアム会員なら」と条件の"意味"が即座に理解できる状態を目指します。

#### 3. 手段（How: どうやって達成するのか？）
* **「名前のない条件式」に「名前」を与える**
  * `if (A && B && C)` のような生の条件式は、「それが何を意味するのか」という意図が欠落しています。これを防ぐために、以下の2つのアプローチで**条件式に名前を与えます（カプセル化）**。

  1. **説明変数（変数の抽出）を使う**
     * ❌ `if (user.age >= 20 && user.hasLicense)` 
     * ⭕️ `boolean isAdultDriver = (user.age >= 20 && user.hasLicense);`  
       `if (isAdultDriver)`
       * *→ 条件が変わらない一部の処理であれば、一時変数に名前を見出しとして付けるだけで劇的に読みやすくなります。*

  2. **メソッドに抽出する（Extract Method）**
     * ❌ `if (A && B && ...)` 
     * ⭕️ `if (isPremiumUser(user))`
       * *→ 条件式が2つ以上になった場合や、アプリのあちこちで同じ判定をする場合は、条件そのものを別メソッドにくくり出し、「その条件が意味する名前（例：プレミアム会員か？）」をメソッド名として命名します。*