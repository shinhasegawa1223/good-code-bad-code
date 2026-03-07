

## 技術駆動命名

**Java:**
```java
class MemoryStateManager {
    void changeIntValue01(int changeValue) {
        intValue01 -= changeValue; 
        if (intValue01 < 0) {
            intValue01 = 0; 
            updateState02Flag();        
        }   
    }
    // ...
}
```

**Python:**
```python
class MemoryStateManager:
    def change_int_value_01(self, change_value: int):
        self.int_value_01 -= change_value 
        if self.int_value_01 < 0:
            self.int_value_01 = 0 
            self.update_state_02_flag()
    # ...
```

**TypeScript:**
```typescript
class MemoryStateManager {
    intValue01: number = 0;

    changeIntValue01(changeValue: number): void {
        this.intValue01 -= changeValue;
        if (this.intValue01 < 0) {
            this.intValue01 = 0;
            this.updateState02Flag();
        }
    }
    // ...
}
```

---

### 🚨 何がダメなのか？ (技術駆動命名の問題点)

記載されているコードの命名には、以下の問題があります。

#### 1. 技術(データ型など)由来の命名になっている
> 対象: `MemoryStateManager`, `intValue01`

クラス名に「メモリ」「状態管理」、変数名に「`int`（整数の型）」といったプログラムの実装都合の言葉（技術用語）が含まれています。
これらは **「ビジネス要件として何を表すデータなのか」を全く説明しておらず、この変数が何の数値なのか（金額なのか、体力なのか、個数なのか）が全く分かりません。**

#### 2. 変数の使われ方が名前から推測できない
> 対象: `changeIntValue01`, `changeValue`

`changeIntValue`（`int`値を変更する）や `changeValue`（変更する値）という名前は、「値をどう変更するのか」「どういう意図で値を渡すのか」が不明確です。
減算していることから何かを減らしていることは分かりますが、目的が分かりません。

---

### 🌟 改善版コード（目的駆動・ドメイン駆動の命名）

変数が **「何を表すのか（ビジネスドメインにおける意味）」** を明確にする名前に変更します。
たとえば、このクラスがゲームにおいて「プレイヤーのHP（ヒットポイント）」を管理していると仮定した場合、以下のような構造に変更すべきです。

**Java:**
```java
class PlayerHealth {
    int currentHealth;

    void takeDamage(int damageAmount) {
        currentHealth -= damageAmount; 
        if (currentHealth <= 0) {
            currentHealth = 0; 
            updateDeathState(); // HPが0になったら死亡状態に更新する
        }   
    }
    // ...
}
```

**Python:**
```python
class PlayerHealth:
    def take_damage(self, damage_amount: int):
        self.current_health -= damage_amount 
        if self.current_health <= 0:
            self.current_health = 0 
            self.update_death_state()
    # ...
```

**TypeScript:**
```typescript
class PlayerHealth {
    currentHealth: number = 0;

    takeDamage(damageAmount: number): void {
        this.currentHealth -= damageAmount;
        if (this.currentHealth <= 0) {
            this.currentHealth = 0;
            this.updateDeathState();
        }
    }
    // ...
}
```

#### 改善のポイント
- `MemoryStateManager` → **`PlayerHealth`**: 「メモリ管理状態」という抽象的すぎる名前から、「プレイヤーの体力」を管理するクラスか一目でわかる名前に直しました🐘
- `intValue01` → **`currentHealth`**: ただの「整数」ではなく、「現在の体力」であることがわかります。
- `changeIntValue01` → **`takeDamage`**: 内側で「値を変更する」という処理内容ではなく、「ダメージを受ける」という目的を表しています。
- `changeValue` → **`damageAmount`**: 受け取る引数も、ただの「変更する値」ではなく「受けるダメージの量」を指すようにしました。

---

### 🎯 まとめ（命名における考え方）

技術駆動の命名から脱却し、正しい名前をつけるための考え方を「目的・目標・手段」の順に整理します。

#### 1. 目的（Why: なぜ命名にこだわるのか？）
* **「可読性と保守性の向上」**
  * コードは書く時間よりも、後から誰か（未来の自分を含む）に読まれる時間の方が圧倒的に長いです。
  * 変数やクラスの「意図」が名前に書いてあれば、わざわざ内部のロジックまで読み解かなくてもバグを防ぎやすくなります。

#### 2. 目標（What: どのような状態を目指すのか？）
* **「ビジネス上のドメイン（業務領域）を表現すること」**
  * プログラムの実装都合（`int`、`Memory`、`Flag`）ではなく、それが現実世界・ビジネス要件において「何のデータなのか（例: `Health`、`Damage`）」がひと目でわかる状態を目指します。

#### 3. 手段（How: どうやって達成するのか？）
* **「型やコンピュータの用語を名前から排除する」**
  * ❌ `MemoryStateManager` → ⭕️ `PlayerHealth` (何を管理するクラスか？)
  * ❌ `intValue`、`flag` → ⭕️ `currentHealth`、`isDead` (何の数値/状態か？)
  * ❌ `changeValue` → ⭕️ `takeDamage` (何をどうしたいのか？ 動詞で表現する)
  * 面倒でも、**「これはビジネスドメイン上で何を意味する言葉だろう？」** と自問自答して単語を選ぶことが大切です。
