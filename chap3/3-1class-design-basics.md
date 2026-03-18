## クラス設計の基本 ― 自己防衛するクラスを作る

> 第3章のテーマは **「クラスをどう設計するか」** です。
> 第2章で学んだ「変数を整理する」「メソッドを抽出する」「関心を分離する」「値オブジェクトを作る」といった技法の根底にある **3つの設計原則** を整理します。

![カプセル化の概念図（データとロジックをクラスにまとめる）](image.png)

上の図のように、バラバラに存在していた **「データ（変数）」** と **「ロジック（処理）」** を、1つの **「クラス」** という箱にまとめるのが、クラス設計の出発点です。

---

### 📐 クラス設計における3つの柱

クラスを正しく設計するとは、以下の3つの原則を守ることです。
これらを満たしたクラスは **「クラス単体で正常に動作する」** ように設計されていると言えます。

> 🎮 **身近なたとえ:** ゲームのコントローラーを思い浮かべてください。
> ボタンを押せば技が出る。でも、コントローラーの中の基盤（データ）を直接触ることはできません。
> これが「カプセル化」です。同様に、コントローラーとテレビは別々の機器（＝関心の分離）で、コントローラーだけ別の物に差し替えてもゲームは動きます（＝多態性）。

<br>

#### 1. 🔒 カプセル化（Encapsulation）

> 💡 **用語: カプセル化（Encapsulation）とは？**
> データ（変数）と、そのデータを操作するロジック（メソッド）を **1つのクラスにまとめ** 、外部からデータを直接触れないように隠蔽すること。
> 風邪薬の「カプセル」のように、中の薬（データ）を包み込んで、外から中身を直接いじれないようにするイメージ。

**一言でいうと:** 「データとロジックを薬のカプセルのように包み込んで、外からは中身をいじれないようにする」こと。

<br>

**🤔 なぜカプセル化が必要なの？（初学者向け解説）**

たとえば、RPGゲームの「HP（ヒットポイント）」を考えてみましょう。
HPは「0〜999」の範囲でなければなりません。

もしHPがただの `int`（数値の変数）だったら、プログラムのどこからでも `hp = -500;` のように **ルール違反な値を入れられてしまいます** 。
全ての場所で「HPが0未満になっていないか？」をチェックするのは大変ですし、書き忘れたらバグになります。

カプセル化すれば、HPクラス自身が「0未満は許さない！」と **自動的にルールを守ってくれる** ので、使う側は何も心配する必要がなくなります。

<br>

| 観点 | ❌ カプセル化できていない | ✅ カプセル化できている |
|---|---|---|
| データの公開 | `public int hitPoint;`（誰でも書き換え可能） | `private int hitPoint;`（クラス内部でのみ操作） |
| 操作方法 | 外部が直接 `hp = hp - 30;` と計算する | `hp.damage(30);` とメソッド経由でのみ操作する |
| ルールの保証 | 「HPは0未満にならない」を **呼び出し側が毎回** チェックしなければならない | クラス内部で **常に自動的に** チェックされる |

**Java:**
```java
// ✅ カプセル化されたクラス
class HitPoint {
    private static final int MIN = 0;  // HPの最小値
    private static final int MAX = 999; // HPの最大値
    private final int value; // 外部から書き換え不可（private + final）

    // コンストラクタ：HPを作る時に、不正な値を弾く（門番の役割）
    HitPoint(final int value) {
        if (value < MIN) throw new IllegalArgumentException("HPは" + MIN + "以上");
        if (MAX < value) throw new IllegalArgumentException("HPは" + MAX + "以下");
        this.value = value;
    }

    // データを持つクラス自身にロジックを持たせる
    HitPoint damage(final int amount) {
        final int damaged = Math.max(MIN, this.value - amount);
        return new HitPoint(damaged); // 新しいHPオブジェクトを返す（元のHPは変えない）
    }
}
```

**Python:**
```python
# ✅ カプセル化されたクラス
class HitPoint:
    MIN: int = 0   # HPの最小値
    MAX: int = 999  # HPの最大値

    def __init__(self, value: int):
        # コンストラクタ：HPを作る時に、不正な値を弾く（門番の役割）
        if value < self.MIN:
            raise ValueError(f"HPは{self.MIN}以上")
        if self.MAX < value:
            raise ValueError(f"HPは{self.MAX}以下")
        self._value = value  # _をつけて「外から直接触らないでね」と表現する（Pythonの慣習）

    @property
    def value(self) -> int:
        """読み取り専用のプロパティ。hp.value で値を読めるが、hp.value = 100 とは書けない"""
        return self._value

    def damage(self, amount: int) -> 'HitPoint':
        damaged = max(self.MIN, self._value - amount)
        return HitPoint(damaged)
```

**TypeScript:**
```typescript
// ✅ カプセル化されたクラス
class HitPoint {
    private static readonly MIN = 0;   // HPの最小値
    private static readonly MAX = 999;  // HPの最大値
    readonly value: number; // readonly = 外部から書き換え不可

    // コンストラクタ：HPを作る時に、不正な値を弾く（門番の役割）
    constructor(value: number) {
        if (value < HitPoint.MIN) throw new Error(`HPは${HitPoint.MIN}以上`);
        if (HitPoint.MAX < value) throw new Error(`HPは${HitPoint.MAX}以下`);
        this.value = value;
    }

    damage(amount: number): HitPoint {
        const damaged = Math.max(HitPoint.MIN, this.value - amount);
        return new HitPoint(damaged);
    }
}
```

<br>

---

#### 2. 🧩 関心の分離（Separation of Concerns）

> 💡 **用語: 関心の分離（Separation of Concerns: SoC）とは？**
> プログラムを「それぞれ固有の役割を持つ独立した部品（クラス）」に分割すること。1つのクラスが複数の責務を持たないようにする。

**一言でいうと:** 「1つのクラスには、1つの仕事だけをさせる」こと。

<br>

**🤔 なぜ分離が必要なの？（初学者向け解説）**

たとえば、レストランを想像してください。
「料理を作る人」「料理を運ぶ人」「お金を計算する人」がそれぞれ担当を持っています。
もし1人で全部やっていたら、メニューが増えた時にパンクしますよね？

プログラムも同じで、`BattleService` という1つのクラスが「攻撃力の計算」「防御力の計算」「ダメージの適用」を全部やっていたら、どこかを変えたい時に **巨大なコードの中を探し回る羽目** になります。

「攻撃力のことは `Player` に任せる」「防御力のことは `Enemy` に任せる」と分けておけば、修正は **担当のクラスだけ** で済みます。

<br>

| 観点 | ❌ 関心が分離できていない | ✅ 関心が分離できている |
|---|---|---|
| クラスの責任 | `BattleService` が攻撃力の計算も防御力の計算もダメージの適用も全部やっている（Fat Class） | `Player` は自分の攻撃力を計算する、`Enemy` は自分の防御力を計算する、`BattleService` は司令塔に徹する |
| 変更時の影響 | 攻撃力の計算式を変えたいのに、`BattleService` の巨大なコードの中を探し回る必要がある | `Player` クラスだけを修正すれば良い |

**Java:**
```java
// ✅ 各クラスが自分の責務だけを持つ
class Player {
    // 自分のデータを使った計算は、自分でやる（レストランの「シェフ」）
    public int calculateTotalAttack() { ... }
}

class Enemy {
    // 自分の防御力は自分で計算する
    public int calculateDefense() { ... }
}

// BattleServiceは流れの調整だけ（レストランの「ホールマネージャー」）
class BattleService {
    void attackEnemy(Player player, Enemy enemy) {
        final int attack = player.calculateTotalAttack(); // Playerに聞く
        final int defense = enemy.calculateDefense();     // Enemyに聞く
        final int damage = Math.max(0, attack - defense);
        enemy.takeDamage(damage);
    }
}
```

**Python:**
```python
# ✅ 各クラスが自分の責務だけを持つ
class Player:
    def calculate_total_attack(self) -> int:
        # 自分のデータを使った計算は、自分でやる（レストランの「シェフ」）
        ...

class Enemy:
    def calculate_defense(self) -> int:
        ...

# BattleServiceは流れの調整だけ（レストランの「ホールマネージャー」）
class BattleService:
    def attack_enemy(self, player: Player, enemy: Enemy) -> None:
        attack: int = player.calculate_total_attack()  # Playerに聞く
        defense: int = enemy.calculate_defense()        # Enemyに聞く
        damage: int = max(0, attack - defense)
        enemy.take_damage(damage)
```

**TypeScript:**
```typescript
// ✅ 各クラスが自分の責務だけを持つ
class Player {
    // 自分のデータを使った計算は、自分でやる
    calculateTotalAttack(): number { ... }
}

class Enemy {
    calculateDefense(): number { ... }
}

// BattleServiceは流れの調整だけ
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

---

#### 3. 🔄 多態性による機能の取り換え（Polymorphism）

> 💡 **用語: 多態性（ポリモーフィズム / Polymorphism）とは？**
> 同じメソッド名で呼び出しても、実際の処理内容がクラスによって **自動的に切り替わる** 仕組み。
> たとえば、リモコンの「再生ボタン」を押すと、DVDプレイヤーならDVDが再生され、音楽プレイヤーなら音楽が流れる。ボタン（インターフェース）は同じだが、中身（実装）が異なる。

> 💡 **用語: インターフェース（Interface）とは？**
> 「このクラスは最低限こういうメソッドを持っていなければならない」というルールブック（契約書）のようなもの。
> 中身の実装は書かず、メソッドの「名前」と「引数」と「戻り値の型」だけを宣言する。

**一言でいうと:** 「呼び出し方は同じだが、中身を差し替えられる（USBのように、規格さえ合えば何でも挿せる）」こと。

<br>

**🤔 なぜ多態性が必要なの？（初学者向け解説）**

たとえば、ゲームで「攻撃の種類」が増えていく場合を考えてください。

*   最初は「物理攻撃」だけだった
*   次に「魔法攻撃」が追加された
*   さらに「アイテム攻撃」も追加された

多態性がなければ、攻撃する場所のコードに `if (type == "physical") { ... } else if (type == "magic") { ... } else if ...` と **どんどんif-elseが増えていきます** 。既存のif文を触るので、誤って壊してしまうリスクもあります。

多態性を使えば、新しい攻撃クラスを **1つ追加するだけ** で済み、既存のコードには一切手を入れる必要がありません。

> 💡 **用語: 開放閉鎖原則（Open-Closed Principle: OCP）とは？**
> 「機能の **追加** に対しては開いて（Open）いるが、既存コードの **修正** に対しては閉じて（Closed）いる」という設計原則。
> つまり、新しい機能を追加する時に、今動いているコードを書き換えなくて済む設計が理想。

<br>

| 観点 | ❌ 多態性がない | ✅ 多態性がある |
|---|---|---|
| 攻撃の種類を増やす場合 | `if-else` の嵐が長くなる | 新しいクラスを1つ追加するだけで済む |
| 変更の影響範囲 | 既存のif文を改修するため **既存コードのバグを誘発するリスク** がある | 既存コードは一切変更する必要がない（開放閉鎖原則） |

**Java:**
```java
// ✅ 共通のインターフェース（ルールブック）を定義
// 「calculateDamage というメソッドを必ず持ちなさい」という契約
interface AttackStrategy {
    int calculateDamage(int basePower);
}

// 物理攻撃の実装（ルールブックに従って中身を書く）
class PhysicalAttack implements AttackStrategy {
    public int calculateDamage(int basePower) {
        return basePower * 2; // 物理は2倍
    }
}

// 魔法攻撃の実装（新しい攻撃を追加しても既存コードは変わらない！）
class MagicAttack implements AttackStrategy {
    public int calculateDamage(int basePower) {
        return basePower * 3; // 魔法は3倍
    }
}

// 使う側は「どの攻撃か」を気にしない（USBポートのように、何が刺さっても動く）
class Player {
    private AttackStrategy attackStrategy; // どの攻撃を使うかは後から差し替え可能
    
    void attack(Enemy enemy) {
        // PhysicalAttackでもMagicAttackでも、同じ呼び方で動く！
        int damage = attackStrategy.calculateDamage(this.strength);
        enemy.takeDamage(damage);
    }
}
```

**Python:**
```python
from abc import ABC, abstractmethod

# ✅ 共通のインターフェース（ルールブック）を定義
# ABC = Abstract Base Class（抽象基底クラス）の略
# @abstractmethod = 「このメソッドは中身を書かず、子クラスで必ず実装しなさい」という指示
class AttackStrategy(ABC):
    @abstractmethod
    def calculate_damage(self, base_power: int) -> int:
        pass  # 中身はここでは書かない（子クラスに任せる）

class PhysicalAttack(AttackStrategy):
    def calculate_damage(self, base_power: int) -> int:
        return base_power * 2

class MagicAttack(AttackStrategy):
    def calculate_damage(self, base_power: int) -> int:
        return base_power * 3

# 使う側は「どの攻撃か」を気にしない
class Player:
    def __init__(self, strength: int, attack_strategy: AttackStrategy):
        self._strength = strength
        self._attack_strategy = attack_strategy  # 後から差し替え可能

    def attack(self, enemy: 'Enemy') -> None:
        # PhysicalAttackでもMagicAttackでも、同じ呼び方で動く！
        damage = self._attack_strategy.calculate_damage(self._strength)
        enemy.take_damage(damage)

# 使用例
warrior = Player(strength=10, attack_strategy=PhysicalAttack())  # 物理攻撃の戦士
mage = Player(strength=10, attack_strategy=MagicAttack())        # 魔法攻撃の魔法使い
```

**TypeScript:**
```typescript
// ✅ 共通のインターフェース（ルールブック）を定義
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

// 使う側は「どの攻撃か」を気にしない
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

// 使用例
const warrior = new Player(10, new PhysicalAttack());  // 物理攻撃の戦士
const mage = new Player(10, new MagicAttack());        // 魔法攻撃の魔法使い
```

---

### 🎯 まとめ（クラス設計における考え方）

> **「クラス単体で正常に動作する」** ように設計すること。それは以下の3原則を全て満たしたクラスです。

| 原則 | たとえ話 | 目的 | 手段 |
|---|---|---|---|
| 🔒 **カプセル化** | 薬のカプセル（中身を外から触れない） | 不正な状態を **物理的に作れなくする** | データを `private` にし、操作は必ずメソッド経由にする |
| 🧩 **関心の分離** | レストランの分業（シェフ・ウェイター・会計） | 変更箇所を **1つのクラスに局所化する** | 1つのクラスには1つの責務だけを持たせる |
| 🔄 **多態性** | USBポート（規格が合えば何でも挿せる） | 機能を追加しても **既存コードに手を入れない** | インターフェースと実装を分離し、振る舞いを差し替え可能にする |

> ※ この3原則は独立したものではなく、互いに補完し合っています。カプセル化でクラスの「堅さ」を作り、関心の分離で「整理」し、多態性で「柔軟性」を得る。この3つが揃って初めて、変更に強い堅牢なクラス設計が完成します。