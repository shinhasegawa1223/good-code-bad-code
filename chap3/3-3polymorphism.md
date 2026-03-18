## 多態性 ― 機能をプラグインのように取り換える

> 🔌 **身近なたとえ:** USBポートを想像してください。
> USBポートには、マウスでもキーボードでもUSBメモリでも **規格さえ合えば何でも挿せます** 。
> パソコン側は「何が刺さっているか」を知らなくても動作する。
> プログラムでも同じように、 **中身を自由に差し替えられる仕組み** を作れます。それが「多態性」です。

**Java:**
```java
// 攻撃の種類をif-elseで分岐させている
class Player {
    String attackType; // "physical" or "magic" or "item"

    void attack(Enemy enemy) {
        int damage;

        if (attackType.equals("physical")) {
            // 物理攻撃の計算
            damage = this.strength * 2;
        } else if (attackType.equals("magic")) {
            // 魔法攻撃の計算
            damage = this.intelligence * 3;
        } else if (attackType.equals("item")) {
            // アイテム攻撃の計算
            damage = 50;
        } else {
            damage = 0;
        }

        enemy.takeDamage(damage);
    }
}
```

**Python:**
```python
# 攻撃の種類をif-elseで分岐させている
class Player:
    def __init__(self, attack_type: str, strength: int, intelligence: int):
        self.attack_type = attack_type  # "physical" or "magic" or "item"
        self.strength = strength
        self.intelligence = intelligence

    def attack(self, enemy: 'Enemy') -> None:
        if self.attack_type == "physical":
            damage = self.strength * 2
        elif self.attack_type == "magic":
            damage = self.intelligence * 3
        elif self.attack_type == "item":
            damage = 50
        else:
            damage = 0

        enemy.take_damage(damage)
```

**TypeScript:**
```typescript
// 攻撃の種類をif-elseで分岐させている
class Player {
    attackType: string; // "physical" or "magic" or "item"
    strength: number;
    intelligence: number;

    attack(enemy: Enemy): void {
        let damage: number;

        if (this.attackType === "physical") {
            damage = this.strength * 2;
        } else if (this.attackType === "magic") {
            damage = this.intelligence * 3;
        } else if (this.attackType === "item") {
            damage = 50;
        } else {
            damage = 0;
        }

        enemy.takeDamage(damage);
    }
}
```

---

### 🚨 何がダメなのか？ (if-else分岐の問題点)

> 💡 **用語: 多態性（ポリモーフィズム / Polymorphism）とは？**
> 同じメソッド名で呼び出しても、実際の処理内容がクラスによって **自動的に切り替わる** 仕組み。
> リモコンの「再生ボタン」を押すと、DVDプレイヤーならDVDが再生され、音楽プレイヤーなら音楽が流れる。ボタン（インターフェース）は同じだが、中身（実装）が異なる。

<br>

#### 1. 🔴 新しい攻撃を追加するたびに既存コードを修正しなければならない
> 対象: `if-else` の分岐

| 名前 | 問題 |
|---|---|
| `if-else` の連鎖 | 「氷魔法攻撃」「毒攻撃」「召喚攻撃」…と攻撃の種類が増えるたびに、 **この `if-else` にどんどん分岐を追加** しなければならない。既存の `if` 文の中を触るため、 **今まで動いていた物理攻撃や魔法攻撃のコードを壊してしまうリスク** がある。 |

> 💡 **用語: 開放閉鎖原則（Open-Closed Principle: OCP）とは？**
> 「機能の **追加** に対しては開いて（Open）いるが、既存コードの **修正** に対しては閉じて（Closed）いる」という設計原則。
> つまり、 **新しい機能を追加する時に、今動いているコードを書き換えなくて済む** 設計が理想。

<br>

#### 2. 🔴 文字列（"physical"等）のタイプミスに気づけない
> 対象: `attackType.equals("physical")`

| 名前 | 問題 |
|---|---|
| 文字列による分岐 | `"physcial"`（typo）と書いてしまっても **コンパイルエラーにならず実行されてしまう** 。`else` に入って `damage = 0` になるバグは、テストで見つけるのが非常に難しい。 |

---

### 🌟 改善版コード（インターフェースで振る舞いを差し替え可能にする）

> 💡 **用語: インターフェース（Interface）とは？**
> 「このクラスは最低限こういうメソッドを持っていなければならない」というルールブック（契約書）のようなもの。
> 中身の実装は書かず、メソッドの「名前」と「引数」と「戻り値の型」だけを宣言する。

**一言でいうと:** `if-else` で分岐する代わりに、 **「攻撃の計算方法」をインターフェースとして定義** し、物理攻撃・魔法攻撃などを別々のクラスで実装する。新しい攻撃を追加する時は **クラスを1つ追加するだけ** で、既存コードは一切触らない。

**Java:**
```java
// ✅ 「攻撃の計算方法」のルールブック（インターフェース）
// 「calculateDamage というメソッドを必ず持ちなさい」という契約
interface AttackStrategy {
    int calculateDamage(int basePower);
}

// 物理攻撃（ルールブックに従って中身を書く）
class PhysicalAttack implements AttackStrategy {
    public int calculateDamage(int basePower) {
        return basePower * 2; // 物理攻撃は基本攻撃力の2倍
    }
}

// 魔法攻撃（新しい攻撃を追加しても、PhysicalAttackのコードは一切変わらない！）
class MagicAttack implements AttackStrategy {
    public int calculateDamage(int basePower) {
        return basePower * 3; // 魔法攻撃は基本攻撃力の3倍
    }
}

// 使う側は「どの攻撃か」を全く気にしない（USBポートのように、何が刺さっても動く）
class Player {
    private int strength;
    private AttackStrategy attackStrategy; // ← ここに好きな攻撃をセットできる

    void attack(Enemy enemy) {
        // PhysicalAttackでもMagicAttackでも、全く同じコードで動く！
        int damage = attackStrategy.calculateDamage(this.strength);
        enemy.takeDamage(damage);
    }
}
```

**Python:**
```python
from abc import ABC, abstractmethod

# ✅ 「攻撃の計算方法」のルールブック（抽象基底クラス）
# ABC = Abstract Base Class の略。「このクラスを直接使うな、必ず継承しろ」という意味
# @abstractmethod = 「このメソッドの中身はここでは書かない。子クラスで必ず実装しなさい」
class AttackStrategy(ABC):
    @abstractmethod
    def calculate_damage(self, base_power: int) -> int:
        pass  # 中身はここでは書かない（子クラスに任せる）

# 物理攻撃
class PhysicalAttack(AttackStrategy):
    def calculate_damage(self, base_power: int) -> int:
        return base_power * 2

# 魔法攻撃
class MagicAttack(AttackStrategy):
    def calculate_damage(self, base_power: int) -> int:
        return base_power * 3

# 使う側は「どの攻撃か」を全く気にしない
class Player:
    def __init__(self, strength: int, attack_strategy: AttackStrategy):
        self._strength = strength
        self._attack_strategy = attack_strategy

    def attack(self, enemy: 'Enemy') -> None:
        damage: int = self._attack_strategy.calculate_damage(self._strength)
        enemy.take_damage(damage)
```

**TypeScript:**
```typescript
// ✅ 「攻撃の計算方法」のルールブック（インターフェース）
interface AttackStrategy {
    calculateDamage(basePower: number): number;
}

class PhysicalAttack implements AttackStrategy {
    calculateDamage(basePower: number): number {
        return basePower * 2;
    }
}

class MagicAttack implements AttackStrategy {
    calculateDamage(basePower: number): number {
        return basePower * 3;
    }
}

// 使う側は「どの攻撃か」を全く気にしない
class Player {
    private attackStrategy: AttackStrategy;

    constructor(private strength: number, attackStrategy: AttackStrategy) {
        this.attackStrategy = attackStrategy;
    }

    attack(enemy: Enemy): void {
        const damage = this.attackStrategy.calculateDamage(this.strength);
        enemy.takeDamage(damage);
    }
}
```

#### 📝 外部からの使い方（使用例）

**Java:**
```java
// 物理攻撃の戦士を作る
Player warrior = new Player(10, new PhysicalAttack());
warrior.attack(enemy); // ダメージ = 10 * 2 = 20

// 魔法攻撃の魔法使いを作る
Player mage = new Player(10, new MagicAttack());
mage.attack(enemy); // ダメージ = 10 * 3 = 30

// 🎉 新しい攻撃を追加したくなったら？→ クラスを1つ作るだけ！
class PoisonAttack implements AttackStrategy {
    public int calculateDamage(int basePower) {
        return basePower + 100; // 毒攻撃は固定ダメージ追加
    }
}
// 既存の Player, PhysicalAttack, MagicAttack のコードは一切変更なし！
```

**Python:**
```python
warrior = Player(strength=10, attack_strategy=PhysicalAttack())
warrior.attack(enemy)  # ダメージ = 10 * 2 = 20

mage = Player(strength=10, attack_strategy=MagicAttack())
mage.attack(enemy)     # ダメージ = 10 * 3 = 30

# 🎉 新しい攻撃 → クラスを1つ作るだけ！既存コード変更なし！
class PoisonAttack(AttackStrategy):
    def calculate_damage(self, base_power: int) -> int:
        return base_power + 100
```

**TypeScript:**
```typescript
const warrior = new Player(10, new PhysicalAttack());
warrior.attack(enemy); // 20

const mage = new Player(10, new MagicAttack());
mage.attack(enemy);    // 30

// 🎉 新しい攻撃 → クラスを1つ作るだけ！
class PoisonAttack implements AttackStrategy {
    calculateDamage(basePower: number): number {
        return basePower + 100;
    }
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| 新しい攻撃の追加 | `if-else` に分岐を追加する（既存コードを修正） | 新しいクラスを1つ作るだけ（既存コードに触らない） | **開放閉鎖原則（OCP）** が守られ、既存機能を壊すリスクがゼロになった |
| タイプミスのリスク | `"physcial"` と書いてもエラーにならない | クラス名 `PhysicalAttack` のタイプミスは **コンパイラが即座に検出** してくれる | バグの発見が「テスト時」から「コーディング時」に前倒しされた |
| 責務の分離 | `Player` クラスが全攻撃の計算ロジックを抱え込んでいる | 各攻撃クラスが自分のダメージ計算だけを担当 | `Player` がスリムになり、各攻撃のロジック変更が **独立して** 行えるようになった |

---

### 🎯 まとめ（多態性における考え方）

#### 1. 目的（Why）
> **「新機能を追加しても、既存コードに手を入れずに済むようにするため」**

* `if-else` の嵐は、機能が増えるたびに既存コードを書き換える必要があり、バグの温床になる。
* 多態性を使えば、 **「追加」は新クラスの作成だけで完結** し、既存コードは安全に保たれる。

#### 2. 目標（What）
> **「呼び出し側が『何が来るか』を知らなくても正しく動く状態」**

* `player.attack(enemy)` と書くだけで、物理でも魔法でも毒でも正しく処理される。

#### 3. 手段（How）
> **「if-else の分岐を見つけたら、インターフェース + クラス分離に置き換える」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| 分岐の方法 | `if (type == "A") { ... } else if (type == "B") { ... }` | `interface Strategy` + `class A implements Strategy` + `class B implements Strategy` |

> ※ 「`if-else` や `switch` が3つ以上の分岐を持ち、今後さらに増える可能性がある」と感じたら、それが **多態性を導入すべきサイン** です。
