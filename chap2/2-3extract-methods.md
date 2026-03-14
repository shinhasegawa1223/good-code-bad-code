## 目的ごとのまとまりでメソッド化する（Extract Method）

**Java:**
```java
// プレイヤーステータスから基本攻撃力を算出する処理
int baseAttack = player.strength + (player.level * 2);
int weaponAttack = player.weapon.attackPower;
if (player.hasAttackBuff) {
    weaponAttack = (int)(weaponAttack * 1.5);
}
final int totalAttack = baseAttack + weaponAttack;

// 敵のステータスから防御力を算出する処理
int enemyDefense = enemy.vitality + (enemy.level * 1);
if (enemy.hasDefenseBuff) {
    enemyDefense = (int)(enemyDefense * 1.5);
}

// 最終ダメージを算出する処理
final int baseDamage = totalAttack;
final int damageAfterDefense = Math.max(0, baseDamage - enemyDefense);
final int finalDamageAmount = damageAfterDefense + player.equipmentBonus;

// 敵にダメージを適用する処理
enemy.hp -= finalDamageAmount;
```

**Python:**
```python
# プレイヤーステータスから基本攻撃力を算出する処理
base_attack: int = player.strength + (player.level * 2)
weapon_attack: int = player.weapon.attack_power
if player.has_attack_buff:
    weapon_attack = int(weapon_attack * 1.5)
total_attack: int = base_attack + weapon_attack

# 敵のステータスから防御力を算出する処理
enemy_defense: int = enemy.vitality + (enemy.level * 1)
if enemy.has_defense_buff:
    enemy_defense = int(enemy_defense * 1.5)

# 最終ダメージを算出する処理
base_damage: int = total_attack
damage_after_defense: int = max(0, base_damage - enemy_defense)
final_damage_amount: int = damage_after_defense + player.equipment_bonus

# 敵にダメージを適用する処理
enemy.hp -= final_damage_amount
```

**TypeScript:**
```typescript
// プレイヤーステータスから基本攻撃力を算出する処理
const baseAttack: number = player.strength + (player.level * 2);
let weaponAttack: number = player.weapon.attackPower;
if (player.hasAttackBuff) {
    weaponAttack = Math.floor(weaponAttack * 1.5);
}
const totalAttack: number = baseAttack + weaponAttack;

// 敵のステータスから防御力を算出する処理
let enemyDefense: number = enemy.vitality + (enemy.level * 1);
if (enemy.hasDefenseBuff) {
    enemyDefense = Math.floor(enemyDefense * 1.5);
}

// 最終ダメージを算出する処理
const baseDamage: number = totalAttack;
const damageAfterDefense: number = Math.max(0, baseDamage - enemyDefense);
const finalDamageAmount: number = damageAfterDefense + player.equipmentBonus;

// 敵にダメージを適用する処理
enemy.hp -= finalDamageAmount;
```

---

### 🚨 何がダメなのか？ (処理のベタ書きの問題点)

> 💡 **用語: ベタ書き（Procedural / Spaghetti Code）とは？**
> メソッドや関数に分割せず、1つの長いブロックの中に様々な目的の計算ロジックを上から下までズラズラと書き連ねること。

<br>

#### 1. 🔴 どこからどこまでが「何の処理」なのか分かりにくい
> 対象: `一連の長い計算ロジック`

| 名前 | 問題 |
|---|---|
| メソッド全体 | 変数を分けて役割を明確化しても、**異なる目的の処理（攻撃力の計算、防御力の計算、ダメージ計算）が全て同じ場所に混在している**ため、全体の見通しが悪く、読む人は「今はどの計算をしているのか」をコメントを頼りに脳内で区切って読む必要がある。 |

<br>

#### 2. 🔴 ロジックが複雑になった時に破綻する
> 対象: `計算ロジックの追加`

| 名前 | 問題 |
|---|---|
| ロジックの複雑化 | 「クリティカル判定」「属性相性計算」などの新しい仕様が追加された場合、**この長いコードのどこかに無理やり処理を差し込む**ことになり、さらに長大でメンテナンス不可能なコードになりやすい。 |

---

### 🌟 改善版コード（目的ごとにメソッドを抽出して分離する）

> 💡 **用語: メソッド抽出（Extract Method）とは？**
> 長いコードブロックの中から「意味のある塊（目的のまとまり）」を見つけ出し、それぞれを独立した機能（メソッド）として切り出すリファクタリング手法。

**一言でいうと:** ダラダラと続く一連の処理の流れを、**「何をするか（What）」が分かる名前をつけた小さなメソッド** に分割して呼び出す形にする。

**Java:**
```java
// 処理の「流れ（What）」だけが一目でわかるメインメソッド
void attackEnemy(Player player, Enemy enemy) {
    final int totalAttack = calculateTotalAttack(player);
    final int enemyDefense = calculateEnemyDefense(enemy);
    final int finalDamageAmount = calculateDamage(totalAttack, enemyDefense, player.equipmentBonus);
    
    enemy.takeDamage(finalDamageAmount);
}

// 1. 攻撃力を算出するメソッド（Howの詳細を隠蔽）
private int calculateTotalAttack(Player player) {
    int baseAttack = player.strength + (player.level * 2);
    int weaponAttack = player.weapon.attackPower;
    if (player.hasAttackBuff) {
        weaponAttack = (int)(weaponAttack * 1.5);
    }
    return baseAttack + weaponAttack;
}

// 2. 防御力を算出するメソッド
private int calculateEnemyDefense(Enemy enemy) {
    int defense = enemy.vitality + (enemy.level * 1);
    if (enemy.hasDefenseBuff) {
        defense = (int)(defense * 1.5);
    }
    return defense;
}

// 3. 最終ダメージを算出するメソッド
private int calculateDamage(int attack, int defense, int equipmentBonus) {
    final int damageAfterDefense = Math.max(0, attack - defense);
    return damageAfterDefense + equipmentBonus;
}
```

**Python:**
```python
# 処理の「流れ（What）」だけが一目でわかるメインメソッド
def attack_enemy(player: Player, enemy: Enemy) -> None:
    total_attack: int = calculate_total_attack(player)
    enemy_defense: int = calculate_enemy_defense(enemy)
    final_damage_amount: int = calculate_damage(total_attack, enemy_defense, player.equipment_bonus)
    
    enemy.take_damage(final_damage_amount)

# 1. 攻撃力を算出するメソッド（Howの詳細を隠蔽）
def calculate_total_attack(player: Player) -> int:
    base_attack: int = player.strength + (player.level * 2)
    weapon_attack: int = player.weapon.attack_power
    if player.has_attack_buff:
        weapon_attack = int(weapon_attack * 1.5)
    return base_attack + weapon_attack

# 2. 防御力を算出するメソッド
def calculate_enemy_defense(enemy: Enemy) -> int:
    defense: int = enemy.vitality + (enemy.level * 1)
    if enemy.has_defense_buff:
        defense = int(defense * 1.5)
    return defense

# 3. 最終ダメージを算出するメソッド
def calculate_damage(attack: int, defense: int, equipment_bonus: int) -> int:
    damage_after_defense: int = max(0, attack - defense)
    return damage_after_defense + equipment_bonus
```

**TypeScript:**
```typescript
// 処理の「流れ（What）」だけが一目でわかるメインメソッド
function attackEnemy(player: Player, enemy: Enemy): void {
    const totalAttack: number = calculateTotalAttack(player);
    const enemyDefense: number = calculateEnemyDefense(enemy);
    const finalDamageAmount: number = calculateDamage(totalAttack, enemyDefense, player.equipmentBonus);
    
    enemy.takeDamage(finalDamageAmount);
}

// 1. 攻撃力を算出するメソッド（Howの詳細を隠蔽）
function calculateTotalAttack(player: Player): number {
    const baseAttack: number = player.strength + (player.level * 2);
    let weaponAttack: number = player.weapon.attackPower;
    if (player.hasAttackBuff) {
        weaponAttack = Math.floor(weaponAttack * 1.5);
    }
    return baseAttack + weaponAttack;
}

// 2. 防御力を算出するメソッド
function calculateEnemyDefense(enemy: Enemy): number {
    let defense: number = enemy.vitality + (enemy.level * 1);
    if (enemy.hasDefenseBuff) {
        defense = Math.floor(defense * 1.5);
    }
    return defense;
}

// 3. 最終ダメージを算出するメソッド
function calculateDamage(attack: number, defense: number, equipmentBonus: number): number {
    const damageAfterDefense: number = Math.max(0, attack - defense);
    return damageAfterDefense + equipmentBonus;
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| メイン処理の見た目 | **How（どうやってか）** が延々と書かれている | **What（何をするか）** だけが書かれている | 計算の複雑な詳細コードがいきなり目に入らなくなり、大まかな「シナリオ」だけを読めるようになった |
| 目的の分離 | 1つの巨大なブロック | 3つの独立したメソッド | 「攻撃力の計算」などの目的が明確に分離され、後から安全に変更（機能拡張など）を加えやすくなった |
| コメントへの依存 | コメントで処理の区切りを説明していた | メソッド名自体が処理の区切りの説明になる | コメントがなくても、メソッド名を読むだけで自己文書化（Self-Documenting Code）されるようになった |

---

### 🎯 まとめ（関心事の分離における考え方）

#### 1. 目的（Why）
> **「コードを読む際の『認知負荷（Cognitive Load）』を下げるため」**

* 全てのロジックが1箇所にまとまっていると、読む人は全ての変数や計算式を同時に頭のメモリに保持しなければならない。
* 独立した小さなメソッドに分割することで、「今はこの計算だけ気にすればいい」と人間の脳の負担を減らせる。

#### 2. 目標（What）
> **「メイン処理が『目次（シナリオ）』のように読める状態」**

* メインの処理（呼び出し元）を見た時、複雑な数式やif文ではなく、「攻撃力を出す → 防御力を出す → ダメージを計算する → HPを減らす」という **「業務の流れ」** が一目で把握できる状態を目指す。

#### 3. 手段（How）
> **「『目的が異なるまとまり』を検知したら、すぐに別のメソッドとして切り出す」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| ロジックの書き方 | 全てを同じメソッド内にベタ書きする | `calculateTotalAttack()` 等の「目的」を持ったメソッドに切り出して呼び出す |

> ※ **「コメントで処理のまとまりを区切ろうと思った時」** が、メソッドを抽出する絶好のタイミング（Code Smell）である。
