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

### 🌟 改善版コード（条件の集約によるフラット化 / 関心の分離）

複雑に絡み合った複数の条件を論理演算子（AND, OR）を使って**条件を集約（結合）**するか、あるいは意味のある単位に**メソッド抽出（関心の分離）**してネストを解消します。今回は条件の集約によるフラット化を行います。

**Java:**
```java
// 条件を結合し、ネストを排除してフラットに並べる
if (conditionA && conditionB && conditionC) {
    // 処理1
}

if (conditionA && conditionB && conditionD) {
    // 処理2
}

if (conditionA && conditionE && conditionF) {
    // 処理3
}

if (conditionA && conditionE && conditionG) {
    // 処理4
}
```

**Python:**
```python
# 条件を結合し、ネストを排除してフラットに並べる
if condition_a and condition_b and condition_c:
    pass # 処理1

if condition_a and condition_b and condition_d:
    pass # 処理2

if condition_a and condition_e and condition_f:
    pass # 処理3

if condition_a and condition_e and condition_g:
    pass # 処理4
```

**typescript:**
```typescript
// 条件を結合し、ネストを排除してフラットに並べる
if (conditionA && conditionB && conditionC) {
    // 処理1
}

if (conditionA && conditionB && conditionD) {
    // 処理2
}

if (conditionA && conditionE && conditionF) {
    // 処理3
}

if (conditionA && conditionE && conditionG) {
    // 処理4
}
```

#### 改善のポイント
- `if(A){ if(B){ if(C){} } }` → **`if(A && B && C)`**: 複数の条件が合致したときにだけ実行されるなら、それをネストではなく「論理積（AND）」で表すことで一行の条件に結合しました。
- 独立したブロック化: それぞれの「処理」を実行するための条件がコード上で明確に分断（独立）されるため、「他の状態を気にせずにその処理の条件だけを読めばよい」という状態に改善されました。

---

### 🎯 まとめ（条件集約における考え方）

複雑なネストから脱却し、正しいコードを書くための考え方を「目的・目標・手段」の順に整理します。

#### 1. 目的（Why: なぜこれにこだわるのか？）
* **「状態爆発（バグの発生源）を防ぐため」**
  * `if` 文の中にさらに別の判定をいくつも重ねると、掛け算式で考慮すべきパターンが増え、テストも修正もできなくなります。

#### 2. 目標（What: どのような状態を目指すのか？）
* **「それぞれの処理の実行条件が独立している状態」**
  * ある記述を読むとき、その「スコープの外側」にある情報をいちいち頭に留めておかなくても、その一行を読むだけで「いつ実行されるか」が明らかな状態を目指します。

#### 3. 手段（How: どうやって達成するのか？）
* **「ネストを論理演算子（AND/OR）に変換する」**
  * ❌ 外側の `if` に内包させて前提条件を引き継ぐ → ⭕️ 必要な条件を `&&` で連結して、処理ごとに独立させる。
  * `if` 文の中に別の `if` 文を書きたくなったら、「それらを `&&` で繋いで並列に書けないか？」「別のメソッドに切り出して整理できないか？」を疑う習慣をつけることが大切です。