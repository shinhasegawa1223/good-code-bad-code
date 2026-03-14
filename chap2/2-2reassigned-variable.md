## 再代入による変数の使い回し（Variable Reassignment）

**Java:**
```java
int damageAmount = playerDamage + partyBonus;

// 敵の防御力を引く（再代入）
damageAmount = damageAmount - enemyDefense;

if (damageAmount < 0) {
    // 0未満なら0にする（再代入）
    damageAmount = 0;
}

// 最後に装備ボーナスを追加する（再代入）
damageAmount = damageAmount + equipmentBonus;
```

**Python:**
```python
damage_amount: int = player_damage + party_bonus

# 敵の防御力を引く（再代入）
damage_amount = damage_amount - enemy_defense

if damage_amount < 0:
    # 0未満なら0にする（再代入）
    damage_amount = 0

# 最後に装備ボーナスを追加する（再代入）
damage_amount = damage_amount + equipment_bonus
```

**TypeScript:**
```typescript
let damageAmount: number = playerDamage + partyBonus;

// 敵の防御力を引く（再代入）
damageAmount = damageAmount - enemyDefense;

if (damageAmount < 0) {
    // 0未満なら0にする（再代入）
    damageAmount = 0;
}

// 最後に装備ボーナスを追加する（再代入）
damageAmount = damageAmount + equipmentBonus;
```

---

### 🚨 何がダメなのか？ (再代入による使い回しの問題点)

> 💡 **用語: 変数の再代入（Variable Reassignment）とは？**
> 一度値を入れた変数に対して、後から別の値を入れ直すこと（上書き）。状態が変わるため、コードを読む際に「今のこの変数はどんな値が入っているか？」を常に意識し続けなければならなくなる。

<br>

#### 1. 🔴 途中で値の意味（目的）が変わってしまう
> 対象: `damageAmount`

| 名前 | 問題 |
|---|---|
| `damageAmount` | 最初は「基本ダメージ」、次は「防御力を引いたダメージ」、最後は「最終ダメージ」と、**同じ変数なのに中身の「意味」がコロコロ変わっている**。読む人は「どの時点の `damageAmount` か？」を頭の中でトラッキングしなければならず、バグの温床になる。 |

<br>

#### 2. 🔴 変更不可（immutable）にできないため、予期せぬ変更を防げない
> 対象: 変数の定義（`int` や `let`）

| 名前 | 問題 |
|---|---|
| 変数定義全体 | 何度も上書きすることを前提としているため、**定数化（`final` や `const`）ができない**。これによって、後から間違った計算式を誤って代入してしまっても、コンパイラがエラーを出してくれない。 |

---

### 🌟 改善版コード（目的別の変数を用意する）

> 💡 **用語: 不変（Immutable）な変数とは？**
> 一度代入したら二度と値を変更できない（再代入不可）変数のこと。値が途中で変わらないことが保証されるため、推測が不要になり安全性が極めて高まる。

**一言でいうと:** 変数を使い回さず、計算の「途中結果」や「目的」ごとに新しい不変な変数（`final`, `const`）を用意する。

**Java:**
```java
// 1. 基本となるダメージを計算
final int baseDamage = playerDamage + partyBonus;

// 2. 防御力を引いたダメージを計算（0未満にならないようにする）
final int damageAfterDefense = Math.max(0, baseDamage - enemyDefense);

// 3. 最終的なダメージを計算
final int finalDamageAmount = damageAfterDefense + equipmentBonus;
```

**Python:**
```python
# Pythonには定数の言語仕様はないが、再代入を避けて目的別に変数を分ける
base_damage: int = player_damage + party_bonus

damage_after_defense: int = max(0, base_damage - enemy_defense)

final_damage_amount: int = damage_after_defense + equipment_bonus
```

**TypeScript:**
```typescript
// constを使って再代入を防ぐ
const baseDamage: number = playerDamage + partyBonus;

const damageAfterDefense: number = Math.max(0, baseDamage - enemyDefense);

const finalDamageAmount: number = damageAfterDefense + equipmentBonus;
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| 変数の数 | `damageAmount` （1つ使い回し） | `baseDamage`<br>`damageAfterDefense`<br>`finalDamageAmount` | それぞれのフェーズでの「状態」に固有の名前がついたため、各計算の**目的**が一目でわかるようになった |
| 変数の状態 | 変更可能（ミュータブル） | 不変（イミュータブル） | `final` や `const` を使えるようになり、不用意な上書きによるバグをコンパイラレベルで防げるようになった |

---

### 🎯 まとめ（再代入と変数の分割における考え方）

#### 1. 目的（Why）
> **「途中で値が変わるという『状態の複雑さ』を排除するため」**

* 変数の値がコロコロ変わるコードは、どこで値がおかしくなったのかを追うのが非常に困難（デバッグが難しい）。
* 「1つの変数は1つの目的にのみ使う」（Single-Responsibility Principle）ことで、コードの予測可能性を高める。

#### 2. 目標（What）
> **「全ての変数が定数（Immutable）として定義されている状態」**

* 処理の中で本当に変更が必要な状態（ループカウンタなど）を除き、すべての変数を `final` や `const` で宣言できる状態を目指す。

#### 3. 手段（How）
> **「途中の計算結果には、その意味を表す新しい名前の変数を用意する」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| 使い回し | `d = d - defense` | `final int damageAfterDefense = ...`（新しい変数を定義する） |

> ※ 「メモリがもったいないから変数を使い回す」というのは、現代のコンピュータにおいては全くの杞憂。メモリの節約よりも、**「人間の読みやすさとバグの出にくさ」** を常に優先すること。
