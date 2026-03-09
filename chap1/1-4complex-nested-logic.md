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

論理演算子（AND, OR）で長々と繋げるだけの `if (A && B && C)` もネストよりはマシですが、条件が複雑になると「なぜこの組み合わせで実行されるのか（ビジネスルール）」が分かりにくくなります。真のベストプラクティスは、**複雑な条件式そのものを意味のある名前のメソッドとして抽出（カプセル化）する**ことです。

**Java:**
```java
// 条件をメソッドに抽出し、「何のための条件か（ビジネスルール）」を名前で語らせる
if (isValidUser(conditionA, conditionB, conditionC)) {
    // 処理1（例：ログイン成功処理）
}

if (isPremiumMember(conditionA, conditionB, conditionD)) {
    // 処理2（例：優先クーポンの表示処理）
}

if (needsPasswordReset(conditionA, conditionE, conditionF)) {
    // 処理3（例：警告メッセージの表示処理）
}

if (isAccountLocked(conditionA, conditionE, conditionG)) {
    // 処理4（例：ロック画面への遷移）
}

// 抽出されたメソッド（例）
private boolean isValidUser(boolean a, boolean b, boolean c) {
    return a && b && c;
}
```

**Python:**
```python
# 条件をメソッドに抽出し、「何のための条件か（ビジネスルール）」を名前で語らせる
if is_valid_user(condition_a, condition_b, condition_c):
    pass # 処理1（例：ログイン成功処理）

if is_premium_member(condition_a, condition_b, condition_d):
    pass # 処理2（例：優先クーポンの表示処理）

if needs_password_reset(condition_a, condition_e, condition_f):
    pass # 処理3（例：警告メッセージの表示処理）

if is_account_locked(condition_a, condition_e, condition_g):
    pass # 処理4（例：ロック画面への遷移）

# 抽出されたメソッド（例）
def is_valid_user(a, b, c):
    return a and b and c
```

**TypeScript:**
```typescript
// 条件をメソッドに抽出し、「何のための条件か（ビジネスルール）」を名前で語らせる
if (isValidUser(conditionA, conditionB, conditionC)) {
    // 処理1（例：ログイン成功処理）
}

if (isPremiumMember(conditionA, conditionB, conditionD)) {
    // 処理2（例：優先クーポンの表示処理）
}

if (needsPasswordReset(conditionA, conditionE, conditionF)) {
    // 処理3（例：警告メッセージの表示処理）
}

if (isAccountLocked(conditionA, conditionE, conditionG)) {
    // 処理4（例：ロック画面への遷移）
}

// 抽出されたメソッド（例）
function isValidUser(a: boolean, b: boolean, c: boolean): boolean {
    return a && b && c;
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