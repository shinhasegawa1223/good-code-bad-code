

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

> 💡 **用語: 技術駆動命名（Technology-Driven Naming）とは？**
> データ型（`int`、`String`）や実装の仕組み（`Memory`、`Flag`）など、プログラムの **技術的な都合** をベースに名前をつけてしまうこと。ビジネス上「何を表すか」が名前から読み取れなくなる。

<br>

#### 1. 🔴 技術(データ型など)由来の命名になっている
> 対象: `MemoryStateManager`, `intValue01`

| 名前 | 問題 |
|---|---|
| `MemoryStateManager` | 「メモリ」「状態管理」→ **技術用語のかたまり** 。何を管理するのか不明 |
| `intValue01` | 「int型の値01」→ **金額？体力？個数？** 全く分からない |

<br>

#### 2. 🔴 変数の使われ方が名前から推測できない
> 対象: `changeIntValue01`, `changeValue`

| 名前 | 問題 |
|---|---|
| `changeIntValue01` | 「int値を変更する」→ **どう変更？ なぜ変更？** が不明 |
| `changeValue` | 「変更する値」→ 減算しているが、**目的（ダメージ？消費？）** が不明 |

---

### 🌟 改善版コード（目的駆動・ドメイン駆動の命名）

> 💡 **用語: ドメイン駆動の命名（Domain-Driven Naming）とは？**
> 技術的な都合ではなく、「ビジネス上・現実世界で何を意味するか（ドメイン）」をベースに名前をつけること。
> 例: `intValue01` → `currentHealth`（現在の体力）

**一言でいうと:** 「何の型か」ではなく **「何を表すか（業務用語）」** で名前をつける。

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
    current_health: int = 0

    def take_damage(self, damage_amount: int) -> None:
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

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| クラス名 | `MemoryStateManager` | `PlayerHealth` | 「プレイヤーの体力」を管理すると一目で分かる |
| 変数名 | `intValue01` | `currentHealth` | 「現在の体力」であることが分かる |
| メソッド名 | `changeIntValue01` | `takeDamage` | 「ダメージを受ける」という **目的** を表す |
| 引数名 | `changeValue` | `damageAmount` | 「受けるダメージの量」を表す |

---

### 🎯 まとめ（命名における考え方）

#### 1. 目的（Why）
> **「可読性と保守性の向上」**

* コードは書くより **読まれる時間の方が圧倒的に長い**
* 「意図」が名前に書いてあれば、ロジックを読み解かなくてもバグを防げる

#### 2. 目標（What）
> **「ビジネス上のドメイン（業務領域）を表現すること」**

* `int`、`Memory`、`Flag` ではなく `Health`、`Damage` が **ひと目で分かる** 状態

#### 3. 手段（How）
> **「型やコンピュータの用語を名前から排除し、目的で命名する」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| クラス名 | `MemoryStateManager` | `PlayerHealth`（何を管理？） |
| 変数名 | `intValue`, `flag` | `currentHealth`, `isDead`（何の値？） |
| メソッド名 | `changeValue` | `takeDamage`（何をどうする？） |

> ※ ただし `userList` や `nameMap` のように、コレクションの種類を補足する程度の型情報は可読性を助ける場合もある。**「型名だけで意味のない名前にしない」** ことがポイント。
