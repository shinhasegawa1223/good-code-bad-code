## 値オブジェクト（Value Object）とカプセル化

**Java:**
```java
// HPを変数として扱う（プリミティブ型への執着）
int hitPoint = 100;

// ダメージ処理（どこかにベタ書きされている）
hitPoint = hitPoint - 30;
if (hitPoint < 0) {
    hitPoint = 0;
}

// ... 別の場所 ...
// 誤って不正な値（マイナス）が代入されるのを防げない
hitPoint = -500; 
```

**Python:**
```python
# HPを変数として扱う（プリミティブ型への執着）
hit_point: int = 100

# ダメージ処理（どこかにベタ書きされている）
hit_point = hit_point - 30
if hit_point < 0:
    hit_point = 0

# ... 別の場所 ...
# 誤って不正な値（マイナス）が代入されるのを防げない
hit_point = -500
```

**TypeScript:**
```typescript
// HPを変数として扱う（プリミティブ型への執着）
let hitPoint: number = 100;

// ダメージ処理（どこかにベタ書きされている）
hitPoint = hitPoint - 30;
if (hitPoint < 0) {
    hitPoint = 0;
}

// ... 別の場所 ...
// 誤って不正な値（マイナス）が代入されるのを防げない
hitPoint = -500; 
```

---

### 🚨 何がダメなのか？ (変数のむき出しとプリミティブ型への執着)

> 💡 **用語: プリミティブ型への執着（Primitive Obsession）とは？**
> 「電話番号」「金額」「ヒットポイント」などの、本来「独自のルール（マイナスにならない等）」を持った概念を、単なる `int` や `String` といった言語標準の基本データ型（プリミティブ型）だけで表現しようとすること。

<br>

#### 1. 🔴 不正な値の混入をシステム的に防げない
> 対象: `hitPoint = -500;` などの自由な代入

| 名前 | 問題 |
|---|---|
| `int` (プリミティブ型) | 単なる `int` 型である以上、**「-9999」や「10億」といったゲームの仕様上あり得ない数値でも、コンパイラはエラーを出さずに受け入れてしまう**。 |

<br>

#### 2. 🔴 ロジックがあちこちに散乱する（低凝集）
> 対象: `if (hitPoint < 0) { ... }`

| 名前 | 問題 |
|---|---|
| バリデーション処理 | 「HPは0未満にならない」「HPは最大値を超えない」といったルール（不変条件）を守るための `if` 文が、ダメージを受ける箇所や回復する箇所の**すべてに重複して書かれる**ことになり、DRY原則に違反する。 |

---

### 🌟 改善版コード（専用のクラスを作る：値オブジェクト）

> 💡 **用語: 値オブジェクト（Value Object）とは？**
> システム固有の「値」を表現するための小さなクラス。生成時に不正な値を弾き（バリデーション）、一度作られたら中身が変わらない（不変性）特徴を持つ。

**一言でいうと:** ただの `int` ではなく、**「HP専用の型（クラス）」** を作り、ルールや計算ロジックをすべてそのクラスの中に閉じ込める（カプセル化）。

**Java:**
```java
// "HP" という概念を表現する専用のクラス（値オブジェクト）
class HitPoint {
    private static final int MIN = 0;
    private static final int MAX = 999;
    private static final int MIN_DAMAGE_AMOUNT = 0;
    private static final int MIN_RECOVERY_AMOUNT = 0;
    
    // 値は不変（final）にし、外から勝手に書き換えられないようにする
    final int value;

    // コンストラクタで「不正なHP」が生まれるのを絶対に防ぐ（不変条件の維持）
    HitPoint(final int value) {
        if (value < MIN) {
            throw new IllegalArgumentException("ヒットポイントは" + MIN + "以上を指定してください");
        }
        if (MAX < value) {
            throw new IllegalArgumentException("ヒットポイントは" + MAX + "以下を指定してください");
        }
        this.value = value;
    }

    // HPダメージを受ける（自分の内部状態は変えず、新しいHitPointを返す）
    HitPoint damage(final int damageAmount) {
        if (damageAmount < MIN_DAMAGE_AMOUNT) {
            throw new IllegalArgumentException("ダメージ量は" + MIN_DAMAGE_AMOUNT + "以上を指定してください");
        }
        final int damaged = value - damageAmount;
        final int corrected = damaged < MIN ? MIN : damaged;
        return new HitPoint(corrected); // 新しい状態のオブジェクトを返す（不変性を保つ）
    }

    // HPを回復する
    HitPoint recover(final int recoveryAmount) {
        if (recoveryAmount < MIN_RECOVERY_AMOUNT) {
            throw new IllegalArgumentException("回復量は" + MIN_RECOVERY_AMOUNT + "以上を指定してください");
        }
        final int recovered = value + recoveryAmount;
        final int corrected = MAX < recovered ? MAX : recovered;
        return new HitPoint(corrected);
    }
}
```

#### 📝 外部からの使い方（使用例）

```java
// 使い方：使う側は詳細なルールを知らなくても、絶対に安全に操作できる
HitPoint hp = new HitPoint(100);

// ダメージを受ける（HPがマイナスになることは設計上あり得ない）
hp = hp.damage(30);

// 回復する（最大値を超えても自動で補正される）
hp = hp.recover(500);

// ❌ コンパイルは通るが、実行時に即座にエラーになってバグを教えてくれる（フェイルファスト）
HitPoint errorHp = new HitPoint(-10); // IllegalArgumentException!
```

**Python:**
```python
# 使い方
hp = HitPoint(100)

# ダメージを受ける
hp = hp.damage(30)

# 値を取得する（@property のおかげで、メソッド呼び出し()ではなくプロパティのようにアクセスできる）
print(hp.value) # 70

# ❌ 外部から書き換えようとするとエラーになる（セッターがないため）
# hp.value = 500 # AttributeError!

# ❌ 不正な値での初期化はフェイルファスト
# error_hp = HitPoint(-10) # ValueError!
```

> 💡 **補足: Pythonの `@property`（プロパティデコレータ）とは？**
> メソッド（この場合は `def value(self)`）を、あたかも「ただの変数（プロパティ）」のようにドットアクセス（`hp.value`）で読み取れるようにする機能です。これにより、内部変数 `_value` を隠蔽し、読み取り専用（Read-only）の変数を簡潔に作ることができます。

**TypeScript:**
```typescript
// 使い方
let hp = new HitPoint(100);

// ダメージを受ける
hp = hp.damage(30);

// 値を取得する
console.log(hp.value); // 70

// ❌ TypeScriptの readonly がついているため、コンパイル時にエラーを出してくれる
// hp.value = 500; // Cannot assign to 'value' because it is a read-only property.

// ❌ 不正な値での初期化はフェイルファスト
// const errorHp = new HitPoint(-10); // Error!
```

**Python:**
```python
class HitPoint:
    MIN: int = 0
    MAX: int = 999
    MIN_DAMAGE_AMOUNT: int = 0
    MIN_RECOVERY_AMOUNT: int = 0

    def __init__(self, value: int):
        if value < self.MIN:
            raise ValueError(f"ヒットポイントは{self.MIN}以上を指定してください")
        if self.MAX < value:
            raise ValueError(f"ヒットポイントは{self.MAX}以下を指定してください")
        self._value = value # 慣習として変更不可を示す

    @property
    def value(self) -> int:
        return self._value

    def damage(self, damage_amount: int) -> 'HitPoint':
        if damage_amount < self.MIN_DAMAGE_AMOUNT:
            raise ValueError(f"ダメージ量は{self.MIN_DAMAGE_AMOUNT}以上を指定してください")
        
        damaged = self._value - damage_amount
        corrected = self.MIN if damaged < self.MIN else damaged
        return HitPoint(corrected)

    def recover(self, recovery_amount: int) -> 'HitPoint':
        if recovery_amount < self.MIN_RECOVERY_AMOUNT:
            raise ValueError(f"回復量は{self.MIN_RECOVERY_AMOUNT}以上を指定してください")
            
        recovered = self._value + recovery_amount
        corrected = self.MAX if self.MAX < recovered else recovered
        return HitPoint(corrected)
```

**TypeScript:**
```typescript
class HitPoint {
    private static readonly MIN: number = 0;
    private static readonly MAX: number = 999;
    private static readonly MIN_DAMAGE_AMOUNT: number = 0;
    private static readonly MIN_RECOVERY_AMOUNT: number = 0;

    readonly value: number;

    constructor(value: number) {
        if (value < HitPoint.MIN) {
            throw new Error(`ヒットポイントは${HitPoint.MIN}以上を指定してください`);
        }
        if (HitPoint.MAX < value) {
            throw new Error(`ヒットポイントは${HitPoint.MAX}以下を指定してください`);
        }
        this.value = value;
    }

    damage(damageAmount: number): HitPoint {
        if (damageAmount < HitPoint.MIN_DAMAGE_AMOUNT) {
            throw new Error(`ダメージ量は${HitPoint.MIN_DAMAGE_AMOUNT}以上を指定してください`);
        }
        const damaged = this.value - damageAmount;
        const corrected = damaged < HitPoint.MIN ? HitPoint.MIN : damaged;
        return new HitPoint(corrected);
    }

    recover(recoveryAmount: number): HitPoint {
        if (recoveryAmount < HitPoint.MIN_RECOVERY_AMOUNT) {
            throw new Error(`回復量は${HitPoint.MIN_RECOVERY_AMOUNT}以上を指定してください`);
        }
        const recovered = this.value + recoveryAmount;
        const corrected = HitPoint.MAX < recovered ? HitPoint.MAX : recovered;
        return new HitPoint(corrected);
    }
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| 型の安全性 | `int hitPoint = -500;`（不正な値が入る） | `HitPoint` クラス（コンストラクタで弾く） | 不正な値を持ったHPがシステム内に誕生すること自体を **「絶対不可能」な設計** にできた。 |
| ロジックの置き場所 | ダメージ計算や補正処理が、呼び出し元のあちこちに散乱していた | `HitPoint` クラスの中に全て集まった（高凝集） | 「HPの計算方法」が変わった時、この `HitPoint` クラス **1箇所だけを修正すれば済む** ようになった。 |

---

### 🎯 まとめ（値オブジェクトにおける考え方）

#### 1. 目的（Why）
> **「不正なデータがシステムに入り込むのを根絶し、バグを防ぐため」**

* プリミティブ型（`int`や`String`）は柔軟すぎるため、「あり得ない値」を簡単に許容してしまう。
* 万が一バグでマイナスの値を入れようとしても、クラス側が **フェイルファスト（Fail Fast: エラーを即座に検知して落とす）** することで、被害の拡大を防ぐ。

#### 2. 目標（What）
> **「『HP』という概念が、自らのルールを自ら守っている状態（完全コンストラクタ）」**

* オブジェクトが生成された時点で、「私の中身は100%正しい値です」と保証されている状態。

#### 3. 手段（How）
> **「単なる数値や文字列には専用のクラス（型）を作り、ロジックをカプセル化する」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| 型の選択 | `int money;` や `String zipCode;` | `Money money;` や `ZipCode zipCode;`（新しいクラスを作る） |
| 値の変更 | `this.value = 100`（後から書き換える） | `return new HitPoint(100)`（不変性を保ち、新しいインスタンスを返す） |

> ※ オブジェクト指向設計において、クラスは「現実世界の概念のモデル化」です。単なる `int` ではなく「HP」という独自の概念としてシステムに登場させましょう。
