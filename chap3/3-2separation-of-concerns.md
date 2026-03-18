## 関心の分離 ― 1つのクラスには1つの仕事だけ

> 🍽️ **身近なたとえ:** レストランを想像してください。
> 「料理を作る人（シェフ）」「料理を運ぶ人（ウェイター）」「お金を計算する人（レジ係）」がいます。
> もし1人で全部やっていたら、メニューが増えた時にパンクしますよね？
> プログラムのクラスも同じで、 **「1つのクラスに1つの仕事だけ」** を任せるのが正しい設計です。

**Java:**
```java
// 1つのクラスが全部やっている（Fat Class: 太ったクラス）
class BattleService {

    void attackEnemy(Player player, Enemy enemy) {
        // ① プレイヤーの攻撃力を計算する（Playerの仕事のはず）
        int baseAttack = player.strength + (player.level * 2);
        int weaponAttack = player.weapon.attackPower;
        if (player.hasAttackBuff) {
            weaponAttack = (int)(weaponAttack * 1.5);
        }
        int totalAttack = baseAttack + weaponAttack;

        // ② 敵の防御力を計算する（Enemyの仕事のはず）
        int defense = enemy.vitality + (enemy.level * 1);
        if (enemy.hasDefenseBuff) {
            defense = (int)(defense * 1.5);
        }

        // ③ ダメージを計算して適用する
        int damage = Math.max(0, totalAttack - defense);
        enemy.hp -= damage;
    }
}
```

**Python:**
```python
# 1つのクラスが全部やっている（Fat Class: 太ったクラス）
class BattleService:

    def attack_enemy(self, player: Player, enemy: Enemy) -> None:
        # ① プレイヤーの攻撃力を計算する（Playerの仕事のはず）
        base_attack: int = player.strength + (player.level * 2)
        weapon_attack: int = player.weapon.attack_power
        if player.has_attack_buff:
            weapon_attack = int(weapon_attack * 1.5)
        total_attack: int = base_attack + weapon_attack

        # ② 敵の防御力を計算する（Enemyの仕事のはず）
        defense: int = enemy.vitality + (enemy.level * 1)
        if enemy.has_defense_buff:
            defense = int(defense * 1.5)

        # ③ ダメージを計算して適用する
        damage: int = max(0, total_attack - defense)
        enemy.hp -= damage
```

**TypeScript:**
```typescript
// 1つのクラスが全部やっている（Fat Class: 太ったクラス）
class BattleService {

    attackEnemy(player: Player, enemy: Enemy): void {
        // ① プレイヤーの攻撃力を計算する（Playerの仕事のはず）
        const baseAttack = player.strength + (player.level * 2);
        let weaponAttack = player.weapon.attackPower;
        if (player.hasAttackBuff) {
            weaponAttack = Math.floor(weaponAttack * 1.5);
        }
        const totalAttack = baseAttack + weaponAttack;

        // ② 敵の防御力を計算する（Enemyの仕事のはず）
        let defense = enemy.vitality + (enemy.level * 1);
        if (enemy.hasDefenseBuff) {
            defense = Math.floor(defense * 1.5);
        }

        // ③ ダメージを計算して適用する
        const damage = Math.max(0, totalAttack - defense);
        enemy.hp -= damage;
    }
}
```

---

### 🚨 何がダメなのか？ (1クラスに全部詰め込む問題点)

> 💡 **用語: 関心の分離（Separation of Concerns: SoC）とは？**
> プログラムを「それぞれ固有の役割を持つ独立した部品（クラス）」に分割すること。
> レストランの分業のように、各クラスが **自分の担当だけ** を責任持ってこなす設計。

> 💡 **用語: Fat Class（太ったクラス）とは？**
> あれもこれもと機能を詰め込みすぎて肥大化したクラスのこと。コードが長く、変更するたびにバグが生まれやすい。別名「God Class（神クラス）」とも呼ばれる。

<br>

#### 1. 🔴 他人のデータを引っ張り出して計算している（Tell, Don't Ask 違反）
> 対象: `player.strength`, `player.weapon.attackPower` などを `BattleService` が直接参照

| 名前 | 問題 |
|---|---|
| `BattleService` | Playerのデータ（`strength`, `weapon`, `hasAttackBuff`）を **外から引っ張り出して** 自分で計算している。これでは `Player` クラスはただの「データの入れ物」に過ぎず、何の仕事もしていない。 |

> 💡 **用語: Tell, Don't Ask（求めるな、命じよ）とは？**
> 他のオブジェクトのデータを「教えて（Ask）」もらって自分で計算するのではなく、そのオブジェクトに「やって（Tell）」と命じるだけにすべきという原則。

<br>

#### 2. 🔴 変更箇所を探すのが難しい
> 対象: `BattleService` 全体

| 名前 | 問題 |
|---|---|
| 変更の影響範囲 | 「攻撃力の計算式を変更したい」↔「防御力の計算式を変更したい」↔「ダメージの計算を変更したい」　これら全てが **1つのメソッドの中に混在** しているため、探しづらく、関係ない箇所を誤って壊すリスクがある。 |

---

### 🌟 改善版コード（各クラスに自分の仕事を委譲する）

**一言でいうと:** 「プレイヤーのことはプレイヤーに聞く」「敵のことは敵に聞く」。`BattleService` は **「誰に何を頼むか」の流れ（What）だけを管理する司令塔** に徹する。

**Java:**
```java
// ✅ Playerは自分の攻撃力を自分で計算する（シェフが自分の担当料理を作る）
class Player {
    private int strength;
    private int level;
    private Weapon weapon;
    private boolean hasAttackBuff;

    public int calculateTotalAttack() {
        int baseAttack = this.strength + (this.level * 2);
        int weaponAttack = this.weapon.getAttackPower();
        if (this.hasAttackBuff) {
            weaponAttack = (int)(weaponAttack * 1.5);
        }
        return baseAttack + weaponAttack;
    }
}

// ✅ Enemyは自分の防御力を自分で計算する
class Enemy {
    private int vitality;
    private int level;
    private boolean hasDefenseBuff;
    private HitPoint hp;

    public int calculateDefense() {
        int defense = this.vitality + (this.level * 1);
        if (this.hasDefenseBuff) {
            defense = (int)(defense * 1.5);
        }
        return defense;
    }

    public void takeDamage(int damage) {
        this.hp = this.hp.damage(damage);
    }
}

// ✅ BattleServiceは流れの調整だけ（ホールマネージャー）
class BattleService {
    void attackEnemy(Player player, Enemy enemy) {
        final int attack = player.calculateTotalAttack(); // Playerに「攻撃力出して」と命じる
        final int defense = enemy.calculateDefense();     // Enemyに「防御力出して」と命じる
        final int damage = Math.max(0, attack - defense);
        enemy.takeDamage(damage);                          // Enemyに「ダメージ受けて」と命じる
    }
}
```

**Python:**
```python
# ✅ Playerは自分の攻撃力を自分で計算する
class Player:
    def __init__(self, strength: int, level: int, weapon: Weapon, has_attack_buff: bool):
        self._strength = strength
        self._level = level
        self._weapon = weapon
        self._has_attack_buff = has_attack_buff

    def calculate_total_attack(self) -> int:
        base_attack: int = self._strength + (self._level * 2)
        weapon_attack: int = self._weapon.attack_power
        if self._has_attack_buff:
            weapon_attack = int(weapon_attack * 1.5)
        return base_attack + weapon_attack

# ✅ BattleServiceは流れの調整だけ
class BattleService:
    def attack_enemy(self, player: Player, enemy: Enemy) -> None:
        attack: int = player.calculate_total_attack()
        defense: int = enemy.calculate_defense()
        damage: int = max(0, attack - defense)
        enemy.take_damage(damage)
```

**TypeScript:**
```typescript
// ✅ Playerは自分の攻撃力を自分で計算する
class Player {
    constructor(
        private strength: number,
        private level: number,
        private weapon: Weapon,
        private hasAttackBuff: boolean
    ) {}

    calculateTotalAttack(): number {
        const baseAttack = this.strength + (this.level * 2);
        let weaponAttack = this.weapon.attackPower;
        if (this.hasAttackBuff) {
            weaponAttack = Math.floor(weaponAttack * 1.5);
        }
        return baseAttack + weaponAttack;
    }
}

// ✅ BattleServiceは流れの調整だけ
class BattleService {
    attackEnemy(player: Player, enemy: Enemy): void {
        const attack = player.calculateTotalAttack();
        const defense = enemy.calculateDefense();
        const damage = Math.max(0, attack - defense);
        enemy.takeDamage(damage);
    }
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| ロジックの配置 | `BattleService` が他人のデータを引っ張り出して全部計算 | `Player` が自分の攻撃力を計算し、結果だけを返す | 「自分のことは自分でやる」が守られ、各クラスが自立した |
| 修正時の探しやすさ | 攻撃力の修正のために巨大な `BattleService` の中を探す | `Player` クラスだけを見ればいい | 修正箇所が **担当クラスに局所化** された |
| `BattleService` の役割 | シェフ兼ウェイター兼レジ係（Fat Class） | ホールマネージャー（司令塔）に徹する | 読みやすく、バグが生まれにくくなった |

---

### 🎯 まとめ（関心の分離における考え方）

#### 1. 目的（Why）
> **「変更に強く、どこを直せばいいか一瞬でわかるコードにするため」**

* 1つのクラスに機能を詰め込みすぎると、修正のたびに無関係な箇所を壊すリスクが高まる。

#### 2. 目標（What）
> **「クラス名を見ただけで、何の責務を持つか一瞬でわかる状態」**

* `Player` → プレイヤーのステータス計算、`Enemy` → 敵のステータス計算、`BattleService` → 戦闘の進行管理。

#### 3. 手段（How）
> **「他人のデータを Askするのではなく、持ち主に Tell（命令）する」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| データの操作 | `player.strength + player.level` を外部で計算 | `player.calculateTotalAttack()` と持ち主に聞く |

> ※ 「コメントで処理のまとまりを区切りたくなった時」は、そのまとまりを **別のクラスのメソッドに移動させるべきサイン** です。
