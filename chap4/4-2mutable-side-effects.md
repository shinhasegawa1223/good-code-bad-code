## 可変がもたらす意図せぬ影響 ― インスタンスの使い回しと副作用

> ⚡ **身近なたとえ:** 1つのリモコンを2台のテレビに接続している想像をしてください。
> 片方のテレビの音量を上げたつもりが、 **もう片方のテレビの音量も一緒に上がってしまう** 。
> プログラムでも同じで、 **可変なインスタンスを複数の場所で共有** すると、一方の変更がもう一方に意図せず波及します。

### ケース1: 可変インスタンスの使い回し

**Java:**
```java
// ❌ value に final が付いていない → 外部から自由に書き換え可能
class AttackPower {
    static final int MIN = 0;
    int value;  // ← final なし！変更し放題

    AttackPower(int value) {
        if (value < MIN) {
            throw new IllegalArgumentException();
        }
        this.value = value;
    }
}

class Weapon {
    final AttackPower attackPower;

    Weapon(AttackPower attackPower) {
        this.attackPower = attackPower;
    }
}
```

**Python:**
```python
# ❌ value が変更可能 → 外部から自由に書き換え可能
class AttackPower:
    MIN: int = 0

    def __init__(self, value: int) -> None:
        if value < self.MIN:
            raise ValueError()
        self.value: int = value  # ← 変更可能！

class Weapon:
    def __init__(self, attack_power: AttackPower) -> None:
        self.attack_power: AttackPower = attack_power
```

**TypeScript:**
```typescript
// ❌ value が変更可能 → 外部から自由に書き換え可能
class AttackPower {
    static readonly MIN = 0;
    value: number; // ← readonly なし！変更し放題

    constructor(value: number) {
        if (value < AttackPower.MIN) throw new Error();
        this.value = value;
    }
}

class Weapon {
    readonly attackPower: AttackPower;

    constructor(attackPower: AttackPower) {
        this.attackPower = attackPower;
    }
}
```

---

### 🚨 何がダメなのか？ (可変インスタンス共有の問題点)

> 💡 **用語: 副作用（Side Effect）とは？**
> 関数やメソッドが **引数を受け取って戻り値を返す以外に、外部の状態を変更する** こと。インスタンス変数の書き換え、グローバル変数の変更、ファイルの読み書きなどが副作用にあたる。

<br>

#### 1. 🔴 1つのインスタンスを共有すると、変更が全てに波及する

同じ `AttackPower` インスタンスを剣と斧で共有してしまった場合：

**Java:**
```java
// ❌ 同じ AttackPower インスタンスを使い回す
AttackPower attackPower = new AttackPower(30);
Weapon sword = new Weapon(attackPower);  // 剣
Weapon axe   = new Weapon(attackPower);  // 斧 ← 同じインスタンスを参照！

// 剣だけ強化したつもりが...
sword.attackPower.value += 30;

System.out.println(sword.attackPower.value); // → 60 ✅ 期待通り
System.out.println(axe.attackPower.value);   // → 60 ❌ 斧も60になった！
```

> 対象: `AttackPower attackPower` を2つの `Weapon` で共有

| 名前 | 問題 |
|---|---|
| インスタンスの共有 | `sword` と `axe` が **同じ `AttackPower` オブジェクト** を参照しているため、一方を変更すると **もう一方も変わってしまう** 。これは参照型の特性で、コード上からは見抜きにくいバグになる。 |

<br>

#### 2. 🔴 メソッドによる予期せぬ状態変更

**Java:**
```java
// ❌ メソッドがインスタンスの状態を直接変更する
class AttackPower {
    static final int MIN = 0;
    int value;

    AttackPower(int value) {
        if (value < MIN) throw new IllegalArgumentException();
        this.value = value;
    }

    // ❌ 自分自身の value を直接書き換える（副作用！）
    void enhance(int increase) {
        value += increase;
    }

    // ❌ 自分自身の value を直接書き換える（副作用！）
    void disable() {
        value = MIN;
    }
}
```

```java
AttackPower attackPower = new AttackPower(10);

// スレッドAで強化
attackPower.enhance(10);  // value = 20 になる

// 別のスレッドBで無力化
attackPower.disable();    // value = 0 になる！

// スレッドAは value が20のつもりで処理を続行 → バグ！
System.out.println(attackPower.value); // → 0（予期しない値）
```

| 名前 | 問題 |
|---|---|
| `enhance()` / `disable()` | メソッドが **自分自身の `value` を直接書き換える** 。複数のスレッドや複数の呼び出し元から同じインスタンスを操作した場合、 **誰がいつ値を変えたか** の追跡が極めて困難になる。 |

> 💡 **用語: 主作用と副作用**
>
> | 種類 | 意味 | 例 |
> |---|---|---|
> | **主作用** | 関数が引数を受け取り、戻り値を返すこと | `int add(int a, int b) { return a + b; }` |
> | **副作用** | 主作用以外に **外部の状態を変更** すること | インスタンス変数の変更、グローバル変数の変更、ファイルI/O、参照型引数の変更 |

---

### 🌟 改善版コード（不変にして副作用を排除する）

> 💡 **関数の影響範囲を限定する3つのルール:**
> 1. データ・状態は **引数で受け取る**
> 2. 受け取った状態を **変更しない**
> 3. 値は **関数の戻り値として返却する**

**一言でいうと:** `value` を `final` にして直接変更を禁止。`enhance()` や `disable()` は元のオブジェクトを変更せず、 **新しい `AttackPower` を生成して返す** 。

**Java:**
```java
// ✅ 不変な AttackPower クラス
class AttackPower {
    static final int MIN = 0;
    final int value;  // ← final で不変！

    AttackPower(final int value) {
        if (value < MIN) {
            throw new IllegalArgumentException();
        }
        this.value = value;
    }

    // ✅ 元のオブジェクトは変更せず、新しいインスタンスを返す
    AttackPower reinForce(final AttackPower increment) {
        return new AttackPower(this.value + increment.value);
    }

    // ✅ 元のオブジェクトは変更せず、新しいインスタンスを返す
    AttackPower disable() {
        return new AttackPower(MIN);
    }
}
```

📝 **使い方:**
```java
final AttackPower attackPower = new AttackPower(20);

// ✅ 強化しても元の attackPower は変わらない（新しいインスタンスが返る）
final AttackPower reinforced = attackPower.reinForce(new AttackPower(15));
System.out.println(attackPower.value);  // → 20（変わらない！）
System.out.println(reinforced.value);   // → 35（新しいインスタンス）

// ✅ 無力化しても元は変わらない
final AttackPower disabled = attackPower.disable();
System.out.println(attackPower.value);  // → 20（変わらない！）
System.out.println(disabled.value);     // → 0（新しいインスタンス）
```

**Python:**
```python
# ✅ 不変な AttackPower クラス
class AttackPower:
    MIN: int = 0

    def __init__(self, value: int) -> None:
        if value < self.MIN:
            raise ValueError()
        self._value: int = value

    @property
    def value(self) -> int:
        return self._value

    # ✅ 新しいインスタンスを返す（元は変更しない）
    def reinforce(self, increment: 'AttackPower') -> 'AttackPower':
        return AttackPower(self._value + increment.value)

    # ✅ 新しいインスタンスを返す
    def disable(self) -> 'AttackPower':
        return AttackPower(self.MIN)
```

📝 **使い方:**
```python
attack_power = AttackPower(20)

reinforced = attack_power.reinforce(AttackPower(15))
print(attack_power.value)  # → 20（変わらない！）
print(reinforced.value)    # → 35

disabled = attack_power.disable()
print(attack_power.value)  # → 20（変わらない！）
print(disabled.value)      # → 0
```

**TypeScript:**
```typescript
// ✅ 不変な AttackPower クラス
class AttackPower {
    static readonly MIN = 0;
    readonly value: number; // ← readonly で不変！

    constructor(value: number) {
        if (value < AttackPower.MIN) throw new Error();
        this.value = value;
    }

    reinforce(increment: AttackPower): AttackPower {
        return new AttackPower(this.value + increment.value);
    }

    disable(): AttackPower {
        return new AttackPower(AttackPower.MIN);
    }
}
```

<br>

---

### 🗡️ Weapon クラスも不変にする

`AttackPower` が不変になったので、 `Weapon` クラスも同様に **武器の強化は新しい `Weapon` を返す** ように改善します。

**Java:**
```java
// ✅ 不変な Weapon クラス
class Weapon {
    final AttackPower attackPower;

    Weapon(final AttackPower attackPower) {
        this.attackPower = attackPower;
    }

    // ✅ 武器を強化 → 元の Weapon は変わらず、新しい Weapon を返す
    Weapon reinForce(final AttackPower increment) {
        AttackPower reinforced = attackPower.reinForce(increment);
        return new Weapon(reinforced);
    }
}
```

📝 **使い方:**
```java
final AttackPower powerA = new AttackPower(20);
final AttackPower powerB = new AttackPower(20);

final Weapon weaponA = new Weapon(powerA);
final Weapon weaponB = new Weapon(powerB);

// ✅ 武器Aだけ強化しても、武器Bには一切影響しない！
final AttackPower increment = new AttackPower(5);
final Weapon reinforcedWeaponA = weaponA.reinForce(increment);

System.out.println(weaponA.attackPower.value);           // → 20（元のまま）
System.out.println(reinforcedWeaponA.attackPower.value);  // → 25（強化版）
System.out.println(weaponB.attackPower.value);            // → 20（影響なし！）
```

**Python:**
```python
# ✅ 不変な Weapon クラス
class Weapon:
    def __init__(self, attack_power: AttackPower) -> None:
        self._attack_power: AttackPower = attack_power

    @property
    def attack_power(self) -> AttackPower:
        return self._attack_power

    def reinforce(self, increment: AttackPower) -> 'Weapon':
        reinforced: AttackPower = self._attack_power.reinforce(increment)
        return Weapon(reinforced)
```

**TypeScript:**
```typescript
// ✅ 不変な Weapon クラス
class Weapon {
    readonly attackPower: AttackPower;

    constructor(attackPower: AttackPower) {
        this.attackPower = attackPower;
    }

    reinforce(increment: AttackPower): Weapon {
        const reinforced = this.attackPower.reinforce(increment);
        return new Weapon(reinforced);
    }
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| フィールド | `int value`（変更可能） | `final int value`（不変） | **外部からの書き換えが物理的に不可能** に |
| `enhance()` | `value += increase`（自身を変更） | `return new AttackPower(...)`（新インスタンスを返す） | **元のオブジェクトが変わらない** ので副作用ゼロ |
| インスタンス共有 | 同じオブジェクトを複数で共有 → 波及 | 各武器が独立したインスタンスを持つ | **一方の変更がもう一方に影響しない** |
| スレッド安全性 | 複数スレッドから変更 → 競合 | 不変なので **スレッド間で安全に共有可能** | データ競合のリスクがゼロに |

---

### 🎯 まとめ（可変がもたらす影響における考え方）

#### 1. 目的（Why）
> **「可変インスタンスの共有による意図しない影響を根絶するため」**

* 可変なオブジェクトを複数箇所で共有すると、一方の変更がもう一方に波及する。
* メソッドが内部状態を直接変更する（副作用がある）と、特にマルチスレッド環境で予測不能な動作を引き起こす。

#### 2. 目標（What）
> **「どのオブジェクトも、生成後は状態が変わらないことが保証されている状態」**

* オブジェクトの値がコード上の宣言箇所を見るだけで確定する。「いつの間にか変わっていた」がない。

#### 3. 手段（How）
> **「フィールドを `final` にし、変更が必要な場合は新しいインスタンスを返す」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| フィールド | `int value;` | `final int value;` |
| 状態変更メソッド | `void enhance() { value += x; }` | `AttackPower reinForce() { return new AttackPower(...); }` |
| インスタンス生成 | 1つのインスタンスを使い回す | 各用途ごとに独立したインスタンスを生成 |

> ※ 不変にするとオブジェクト生成が増えますが、現代のJVMやランタイムは **短寿命のオブジェクト生成を非常に効率的に処理** するため、パフォーマンスの心配は不要です。
