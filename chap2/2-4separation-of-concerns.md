## 関心の分離とクラスへの委譲（Separation of Concerns）

**Java:**
```java
// GameManager や BattleService のような巨大な進行クラス
class BattleService {
    void attackEnemy(Player player, Enemy enemy) {
        final int totalAttack = calculateTotalAttack(player);
        final int enemyDefense = calculateEnemyDefense(enemy);
        final int damage = calculateDamage(totalAttack, enemyDefense, player.equipmentBonus);
        
        enemy.takeDamage(damage);
    }

    // プレイヤーの内部データ（strengthやweapon）を直接引っ張り出して計算している
    private int calculateTotalAttack(Player player) {
        int baseAttack = player.strength + (player.level * 2);
        int weaponAttack = player.weapon.attackPower;
        if (player.hasAttackBuff) {
            weaponAttack = (int)(weaponAttack * 1.5);
        }
        return baseAttack + weaponAttack;
    }

    private int calculateEnemyDefense(Enemy enemy) { ... }
    private int calculateDamage(int attack, int defense, int equipmentBonus) { ... }
}
```

**Python:**
```python
class BattleService:
    def attack_enemy(self, player: Player, enemy: Enemy) -> None:
        total_attack: int = self._calculate_total_attack(player)
        enemy_defense: int = self._calculate_enemy_defense(enemy)
        damage: int = self._calculate_damage(total_attack, enemy_defense, player.equipment_bonus)
        
        enemy.take_damage(damage)

    # プレイヤーの内部データ（strengthやweapon）を直接引っ張り出して計算している
    def _calculate_total_attack(self, player: Player) -> int:
        base_attack: int = player.strength + (player.level * 2)
        weapon_attack: int = player.weapon.attack_power
        if player.has_attack_buff:
            weapon_attack = int(weapon_attack * 1.5)
        return base_attack + weapon_attack
    
    # ... 省略 ...
```

**TypeScript:**
```typescript
class BattleService {
    attackEnemy(player: Player, enemy: Enemy): void {
        const totalAttack = this.calculateTotalAttack(player);
        const enemyDefense = this.calculateEnemyDefense(enemy);
        const damage = this.calculateDamage(totalAttack, enemyDefense, player.equipmentBonus);
        
        enemy.takeDamage(damage);
    }

    // プレイヤーの内部データ（strengthやweapon）を直接引っ張り出して計算している
    private calculateTotalAttack(player: Player): number {
        const baseAttack = player.strength + (player.level * 2);
        let weaponAttack = player.weapon.attackPower;
        if (player.hasAttackBuff) {
            weaponAttack = Math.floor(weaponAttack * 1.5);
        }
        return baseAttack + weaponAttack;
    }
    
    // ... 省略 ...
}
```

---

### 🚨 何がダメなのか？ (ロジックの集中とデータクラス化の問題点)

> 💡 **用語: 関心の分離（Separation of Concerns: SoC）とは？**
> ソフトウェアの設計において、プログラムを「それぞれ固有の役割を持つ独立した部品」に分割すること。プレイヤーステータスの計算は、バトルシステムではなく「プレイヤー自身」が関心を持つべき領域である。

<br>

#### 1. 🔴 データを持つ者とロジックを持つ者が乖離している（貧血ドメインモデル）
> 対象: `calculateTotalAttack(Player player)` メソッド

| 名前 | 問題 |
|---|---|
| メソッドの所属 | メソッド自体は抽出して綺麗になったが、**プレイヤーのデータ（力、レベル、武器）を使って計算するロジックが、プレイヤー自身ではなくバトル側のクラスに書かれている**。この状態では、Playerクラスは単なる「データの入れ物（データクラス）」になってしまう。 |

<br>

#### 2. 🔴 ロジックがあちこちに分散するリスクがある
> 対象: `BattleService` 全体

| 名前 | 問題 |
|---|---|
| ロジックの再利用性 | 「攻撃力を計算する」という処理がバトルクラスに閉じ込められているため、例えば「ステータス画面で攻撃力を表示したい」となった時に、**同じ計算ロジックを別の場所にも書いてしまう（DRY原則違反）** リスクが高まる。 |

---

### 🌟 改善版コード（データを持つクラス自身にロジックを委譲する）

> 💡 **用語: カプセル化（Encapsulation）とは？**
> データと、そのデータを操作する手続き（メソッド）を1つのオブジェクトにまとめ、外部から直接データを触れないように隠蔽すること。

**一言でいうと:** 詳細なロジックを抽出するだけでなく、**その計算に必要なデータを持っているクラス（Player や Enemy）自身に計算メソッドを持たせる**。

**Java:**
```java
// プレイヤー自身に「自分の攻撃力」を計算させる
class Player {
    // データ（ステータス）はカプセル化（private）する
    private int level;
    private int strength;
    private Weapon weapon;
    private boolean hasAttackBuff;

    // 自分自身のデータを使って計算するロジックをここに書く
    public int calculateTotalAttack() {
        int baseAttack = this.strength + (this.level * 2);
        int weaponAttack = this.weapon.attackPower;
        if (this.hasAttackBuff) {
            weaponAttack = (int)(weaponAttack * 1.5);
        }
        return baseAttack + weaponAttack;
    }
}

// バトル側のクラスは、ロジックを持たず「司令塔」に徹する
class BattleService {
    void attackEnemy(Player player, Enemy enemy) {
        // プレイヤーに攻撃力を聞く（ロジックの委譲）
        final int attack = player.calculateTotalAttack();
        final int defense = enemy.calculateDefense();
        
        final int damage = calculateDamage(attack, defense);
        enemy.takeDamage(damage);
    }
}
```

**Python:**
```python
# プレイヤー自身に「自分の攻撃力」を計算させる
class Player:
    def __init__(self, level: int, strength: int, weapon: Weapon, has_attack_buff: bool):
        self._level = level
        self._strength = strength
        self._weapon = weapon
        self._has_attack_buff = has_attack_buff

    # 自分自身のデータを使って計算するロジックをここに書く
    def calculate_total_attack(self) -> int:
        base_attack: int = self._strength + (self._level * 2)
        weapon_attack: int = self._weapon.attack_power
        if self._has_attack_buff:
            weapon_attack = int(weapon_attack * 1.5)
        return base_attack + weapon_attack

# バトル側のクラスは、ロジックを持たず「司令塔」に徹する
class BattleService:
    def attack_enemy(self, player: Player, enemy: Enemy) -> None:
        # プレイヤーに攻撃力を聞く（ロジックの委譲）
        attack: int = player.calculate_total_attack()
        defense: int = enemy.calculate_defense()
        
        damage: int = self._calculate_damage(attack, defense)
        enemy.take_damage(damage)
```

**TypeScript:**
```typescript
// プレイヤー自身に「自分の攻撃力」を計算させる
class Player {
    private level: number;
    private strength: number;
    private weapon: Weapon;
    private hasAttackBuff: boolean;

    // 自分自身のデータを使って計算するロジックをここに書く
    public calculateTotalAttack(): number {
        const baseAttack = this.strength + (this.level * 2);
        let weaponAttack = this.weapon.attackPower;
        if (this.hasAttackBuff) {
            weaponAttack = Math.floor(weaponAttack * 1.5);
        }
        return baseAttack + weaponAttack;
    }
}

// バトル側のクラスは、ロジックを持たず「司令塔」に徹する
class BattleService {
    attackEnemy(player: Player, enemy: Enemy): void {
        // プレイヤーに攻撃力を聞く（ロジックの委譲）
        const attack = player.calculateTotalAttack();
        const defense = enemy.calculateDefense();
        
        const damage = this.calculateDamage(attack, defense);
        enemy.takeDamage(damage);
    }
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| ロジックの配置先 | `BattleService` が他人のデータ（Player）を奪い取って計算していた | `Player` 自身が自分のデータを使って計算し、結果だけを返すようになった | 「自分のことは自分でやる」というオブジェクト指向の基本単位（カプセル化）が守られるようになった |
| 構造の明確さ | BattleServiceが「太ったクラス(Fat Class)」になっていた | 複数のクラスに正しく処理が分散し（委譲）、BattleServiceは調整役に徹することができた | 「プレイヤーに関する修正」はPlayerクラスだけを見れば良くなり、メンテナンス箇所の特定が容易になった |

---

### 🎯 まとめ（関心の分離とオブジェクト指向における考え方）

#### 1. 目的（Why）
> **「変更に強く、再利用可能なコードを作るため」**

* ロジックがデータを所持するクラス自身にカプセル化されていれば、ステータス画面でもバトル画面でも**同じメソッドを安全に呼ぶだけ**で済む（DRY原則）。
* 他のクラスの仕様変更（例：ステータス計算式の変更）に巻き込まれるリスクが減る。

#### 2. 目標（What）
> **「データとロジックが一つにまとまり（高凝集）、クラス間の依存度が低い（疎結合）状態」**

* `BattleService` は詳細な計算式を知らず、ただ命令だけを出す状態。
* 主役である `Player` や `Enemy` は、ただのデータの入れ物ではなく、「振る舞い（メソッド）」を持った豊かなオブジェクトであること。

#### 3. 手段（How）
> **「『どのデータを使って計算しているか』に注目し、データを持つクラスにメソッドごと移動（委譲）する」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| メソッドの作成場所 | どこかの「Manager」や「Service」クラス内にメソッドを無限に作り続ける | `A.b + A.c` のように、ある特定クラスのプロパティばかり使うロジックを見つけたら、**クラスの持ち主自身に計算させる**ようにメソッドを移動させる |

> ※ 「**Tell, Don't Ask（求めるな、命じよ）**」の原則。他人のデータを聞き出して計算するのではなく、ただ「〜しろ（計算結果をよこせ）」と命じるだけの設計が美しいとされる。
