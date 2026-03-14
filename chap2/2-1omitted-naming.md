
## 省略された意図不明な命名

**Java:**
```java
int d = 0;
d = p1 + p2 + p3 + p4;

if (d < 0) {
    d = 0;
}
```

**Python:**
```python
d: int = 0
d = p1 + p2 + p3 + p4

if d < 0:
    d = 0
```

**TypeScript:**
```typescript
let d: number = 0;
d = p1 + p2 + p3 + p4;

if (d < 0) {
    d = 0;
}
```

---

### 🚨 何がダメなのか？ (省略命名の問題点)

> 💡 **用語: 省略命名（Omitted Naming）とは？**
> 変数名やメソッド名を極端に短くしたり、連番（1, 2, 3...）にしたりすること。書く時は早いが、読む時に「何を表すデータなのか」文脈から推測する作業が発生し、可読性を著しく下げる。

<br>

#### 1. 🔴 短すぎる変数名で意図が伝わらない
> 対象: `d`

| 名前 | 問題 |
|---|---|
| `d` | **「ダメージ？ 距離？ 日数？」** 一文字では何を表す数値なのか全く分からない。毎回文脈から推論するコストがかかる。 |

<br>

#### 2. 🔴 連番の変数名で対象が不明瞭
> 対象: `p1`, `p2`, `p3`, `p4`

| 名前 | 問題 |
|---|---|
| `p1` 〜 `p4` | **「何のパラメータ？」** プレイヤー(Player)なのか、単なるパラメータ(Parameter)なのか、それ以外の何かなのかが不明。 |

---

### 🌟 改善版コード（目的語を用いた具体的な命名）

> 💡 **用語: 自己文書化（Self-Documenting Code）とは？**
> コメントで説明しなくても、変数名やメソッド名を読むだけでコードの意図や動きが自然と理解できる状態にすること。

**一言でいうと:** 安易に省略せず、その変数が「何のデータを入れるものなのか」が明確に伝わる具体的な英単語で命名する。

**Java:**
```java
int damageAmount = 0;
damageAmount = player1Damage + player2Damage + player3Damage + player4Damage;

if (damageAmount < 0) {
    damageAmount = 0;
}
```

**Python:**
```python
damage_amount: int = 0
damage_amount = player1_damage + player2_damage + player3_damage + player4_damage

if damage_amount < 0:
    damage_amount = 0
```

**TypeScript:**
```typescript
let damageAmount: number = 0;
damageAmount = player1Damage + player2Damage + player3Damage + player4Damage;

if (damageAmount < 0) {
    damageAmount = 0;
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| 合計変数 | `d` | `damageAmount` | 単なる「d」から「ダメージ量」であることが明確になった |
| 各項目変数 | `p1` | `player1Damage` | 推測できなかった「p」が「各プレイヤーのダメージ」だと分かるようになった |

---

### 🎯 まとめ（変数命名における考え方）

#### 1. 目的（Why）
> **「後からコードを読む人の解読コストを劇的に下げるため」**

* プログラムは書く時間よりも **「読まれる時間」** の方が圧倒的に長い。
* 他人（または未来の自分）がコードを読む際に、名前から意図を推測させる手間を省き、誤読によるバグの混入を防ぐ。

#### 2. 目標（What）
> **「変数名を読むだけで、その役割と中身が直感的に予測できる状態」**

* コメントに頼らずとも、コード自体が意図を語る（自己文書化された）状態を目指す。

#### 3. 手段（How）
> **「安易な省略語を避け、具体的な英単語を使う」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| 合計値 | `d` | `damageAmount`（何の量？） |
| 個別値 | `p1` | `player1Damage`（何のデータ？） |

> ※ タイピングの手間（名前の短さ）よりも、 **「意味の明確さ」** を常に優先して命名することが重要。
