## マジックナンバー（意図不明な数値）

**Java:**
```java
int hitPoint;
int damageAmount;
int recoveryAmount;

// ... どこかで実行される処理 ...

// ダメージ処理
hitPoint = hitPoint - damageAmount;
if (hitPoint < 0) {
  hitPoint = 0;
}

// ... 別の場所で実行される処理 ...

// 回復処理
hitPoint = hitPoint + recoveryAmount;
if (999 < hitPoint) {
  hitPoint = 999;
}
```

**Python:**
```python
hit_point: int
damage_amount: int
recovery_amount: int

# ... どこかで実行される処理 ...

# ダメージ処理
hit_point = hit_point - damage_amount
if hit_point < 0:
    hit_point = 0

# ... 別の場所で実行される処理 ...

# 回復処理
hit_point = hit_point + recovery_amount
if 999 < hit_point:
    hit_point = 999
```

**TypeScript:**
```typescript
let hitPoint: number;
let damageAmount: number;
let recoveryAmount: number;

// ... どこかで実行される処理 ...

// ダメージ処理
hitPoint = hitPoint - damageAmount;
if (hitPoint < 0) {
  hitPoint = 0;
}

// ... 別の場所で実行される処理 ...

// 回復処理
hitPoint = hitPoint + recoveryAmount;
if (999 < hitPoint) {
  hitPoint = 999;
}
```

---

### 🚨 何がダメなのか？ (直書きされた数値の問題点)

> 💡 **用語: マジックナンバー（Magic Number）とは？**
> ソースコードの中に直接書き込まれた、意図や根拠が不明な数値（または文字列）のこと。「マジック（魔法）」のように、書いた本人にしか分からない数字という意味が込められている。

<br>

#### 1. 🔴 数値が「何を意味しているのか」分からない
> 対象: `0`, `999`

| 名前 | 問題 |
|---|---|
| `0` | 文脈から「HPの最小値（死亡）」と推測はできるが、初見の人は「なぜ0なのか？」を一瞬立ち止まって考えなければならない。 |
| `999` | 「HPの最大値」だと推測はできるが、**「なぜ1000ではなく999なのか？」「他の場所でも999という数字が使われている場合、それも同じ意味なのか？」** をコード上から証明する手段がない。 |

<br>

#### 2. 🔴 仕様変更時の修正漏れ（バグ）のリスクが極めて高い
> 対象: コード全体に散らばった `999`

| 名前 | 問題 |
|---|---|
| 修正漏れ | 例えば「レベルキャップ解放で最大HPを9999にする」という仕様変更が来た場合、開発者はソースコード全体から `999` と書かれた箇所を検索することになる。しかし、アイテムの最大所持数の `999` など**「意味の違う別の999」まで間違って一括置換してしまう**リスクがある。あるいは、修正し忘れた箇所があってバグになる。 |

---

### 🌟 改善版コード（目的を名前にした定数にする）

> 💡 **用語: 定数化（Constant Extraction）とは？**
> 変化しない固定の値を、大文字とアンダースコア（SNAKE_CASE）で命名した、再代入不可能な変数として定義すること。

**一言でいうと:** ソースコード中には絶対に生の数値を書かず、必ず「意味を表す名前」をつけた定数として定義して使う。

**Java:**
```java
// 意味に名前をつける（定数化）
private static final int MIN_HIT_POINT = 0;
private static final int MAX_HIT_POINT = 999;

int hitPoint;
int damageAmount;
int recoveryAmount;

// ...

// ダメージ処理
hitPoint = hitPoint - damageAmount;
if (hitPoint < MIN_HIT_POINT) {
  hitPoint = MIN_HIT_POINT;
}

// 回復処理
hitPoint = hitPoint + recoveryAmount;
if (MAX_HIT_POINT < hitPoint) {
  hitPoint = MAX_HIT_POINT;
}
```

**Python:**
```python
# 意味に名前をつける（定数化：Pythonでの大文字スネークケースによる慣習）
MIN_HIT_POINT: int = 0
MAX_HIT_POINT: int = 999

hit_point: int
damage_amount: int
recovery_amount: int

# ...

# ダメージ処理
hit_point = hit_point - damage_amount
if hit_point < MIN_HIT_POINT:
    hit_point = MIN_HIT_POINT

# 回復処理
hit_point = hit_point + recovery_amount
if MAX_HIT_POINT < hit_point:
    hit_point = MAX_HIT_POINT
```

**TypeScript:**
```typescript
// 意味に名前をつける（定数化）
const MIN_HIT_POINT: number = 0;
const MAX_HIT_POINT: number = 999;

let hitPoint: number;
let damageAmount: number;
let recoveryAmount: number;

// ...

// ダメージ処理
hitPoint -= damageAmount;
if (hitPoint < MIN_HIT_POINT) {
  hitPoint = MIN_HIT_POINT;
}

// 回復処理
hitPoint += recoveryAmount;
if (MAX_HIT_POINT < hitPoint) {
  hitPoint = MAX_HIT_POINT;
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| 数値の表現 | `if (999 < hitPoint) { hitPoint = 999 }` | `if (MAX_HIT_POINT < hitPoint)` | 生の数値が消え、「最大HPを超えたら〜」という **日本語の文章のように自然に読める** ようになった（自己文書化） |
| 仕様変更時の対応 | ソースコード全体を `999` で文字列検索して置換 | 定数 `MAX_HIT_POINT = 9999;` の1箇所を書き換えるだけ | 1箇所直せばこの定数を使っている全てのコードに反映されるため、**修正漏れや誤爆（他の `999` を書き換えてしまう事故）が絶対に起きなくなった** |

---

### 🎯 まとめ（定数化における考え方）

#### 1. 目的（Why）
> **「マジックナンバーによる属人化とバグの混入を防ぐため」**

* 数値の「意図」は、書いた本人には自明でも、他の人（半年後の自分を含む）には伝わらない。
* 「どこでどの数字を使っているか」を暗記させないことで、チーム開発でのミスを減らす。

#### 2. 目標（What）
> **「コード内に `0` や `1` などの極めて自明な数字以外の『生の数値』が存在しない状態」**

* 全ての数値や固定文字列には、その利用目的を表す「名前（定数名）」がついている状態を目指す。

#### 3. 手段（How）
> **「数値を書く前に、必ず定数を先に宣言してそれを使う癖をつける」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| 数値の扱い | `hp = 999` | `final int MAX_HP = 999;` <br> `hp = MAX_HP;` |

> ※ `0`（最小値）や `1`（インクリメント時の+1）などは定数化しなくても通じることが多いが、`0` が「未設定」や「デフォルト」など単なるゼロ以外の意味を持つ場合は定数化したほうが安全である。
