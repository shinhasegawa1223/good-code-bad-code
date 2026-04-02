## 不変 ― 変数の使い回しと再代入を防ぐ

> 🔒 **身近なたとえ:** 伝言ゲームを想像してください。
> メモ用紙に伝言を書いて、途中で誰かが消しゴムで消して別の内容に書き換えてしまったら、最終的に何が書いてあったのか分からなくなります。
> プログラムでも同じで、 **変数の中身を途中で書き換える** と「今この変数はいくつ？」の追跡が困難になり、バグの原因になります。

**Java:**
```java
// ❌ 計算のためにtmp変数を使い回す
int damage() {
    int tmp = member.power() + member.weaponAttack();
    tmp = (int)(tmp * (1f + member.speed() / 100f));
    tmp = tmp - (int)(enemy.defence / 2);
    tmp = Math.max(0, tmp);
    return tmp;
}
```

**Python:**
```python
# ❌ 計算のためにtmp変数を使い回す
def damage(self) -> int:
    tmp: int = self.member.power() + self.member.weapon_attack()
    tmp = int(tmp * (1.0 + self.member.speed() / 100.0))
    tmp = tmp - int(self.enemy.defence / 2)
    tmp = max(0, tmp)
    return tmp
```

**TypeScript:**
```typescript
// ❌ 計算のためにtmp変数を使い回す
damage(): number {
    let tmp = this.member.power() + this.member.weaponAttack();
    tmp = Math.floor(tmp * (1 + this.member.speed() / 100));
    tmp = tmp - Math.floor(this.enemy.defence / 2);
    tmp = Math.max(0, tmp);
    return tmp;
}
```

---

### 🚨 何がダメなのか？ (変数の使い回しの問題点)

> 💡 **用語: 可変（ミュータブル / Mutable）とは？**
> 一度代入した変数の中身を、後から **何度でも書き換えられる** 状態のこと。`tmp` のように1つの変数に何度も再代入するコードは、途中の値を追跡するのが非常に難しくなる。

<br>

#### 1. 🔴 変数の意味が途中で変わってしまう
> 対象: `tmp` 変数への繰り返し再代入

| 名前 | 問題 |
|---|---|
| `tmp` の使い回し | 1行目の `tmp` は「基礎攻撃力」、2行目は「速度補正後の攻撃力」、3行目は「防御力を引いた後のダメージ」と **同じ変数名なのに中身の意味がコロコロ変わる** 。途中でバグが混入しても、どの時点の `tmp` がおかしいのか特定が非常に困難。 |

<br>

#### 2. 🔴 変数名が「何を表しているか」を伝えていない
> 対象: `tmp` という命名

| 名前 | 問題 |
|---|---|
| `tmp`（テンポラリ） | 「一時的な値」という意味しかなく、 **いま何を計算しているのか** が変数名から読み取れない。コードを読む人は、毎回右辺の計算式を解読して意味を推測しなければならない。 |

<br>

#### 3. 🔴 意図しない再代入が起きてもエラーにならない
> 対象: `int tmp`（再代入可能な宣言）

| 名前 | 問題 |
|---|---|
| 再代入可能な変数 | 再代入を許す宣言にしているため、 **うっかり別の値を代入してしまっても** コンパイラが止めてくれない。コードが長くなるほど、意図しない再代入のリスクが増大する。 |

---

### 🌟 改善版コード（不変にして再代入を防ぐ）

> 💡 **用語: 不変（イミュータブル / Immutable）とは？**
> 一度代入した値を **後から変更できない** ようにすること。Java では `final`、Python では慣習的にアンダースコアと型ヒント、TypeScript では `const` を使う。変更したい場合は **新しい変数を作る** 。

**一言でいうと:** `tmp` を使い回すのではなく、 **計算の各段階に意味のある名前を付けた `final` 変数** をそれぞれ作る。再代入は一切しない。

**Java:**
```java
// ✅ 各計算段階に意味のある名前を付け、全て final で不変にする
int damage() {
    final int basicAttackPower = member.power() + member.weaponAttack();
    final int finalAttackPower = (int)(basicAttackPower * (1f + member.speed() / 100f));
    final int reduction = (int)(enemy.defence / 2);
    final int damage = Math.max(0, finalAttackPower - reduction);
    return damage;
}
```

📝 **使い方:**
```java
int result = damage(); // ✅ 各変数が何を表すか一目瞭然
// basicAttackPower = 100  → 基礎攻撃力
// finalAttackPower = 120  → 速度補正後の攻撃力
// reduction = 25          → 防御力による減算
// damage = 95             → 最終ダメージ
```

**Python:**
```python
# ✅ 各計算段階に意味のある名前を付ける
def damage(self) -> int:
    basic_attack_power: int = self.member.power() + self.member.weapon_attack()
    final_attack_power: int = int(basic_attack_power * (1.0 + self.member.speed() / 100.0))
    reduction: int = int(self.enemy.defence / 2)
    damage: int = max(0, final_attack_power - reduction)
    return damage
```

> 💡 **Python での不変性について:**
> Python には Java の `final` のような再代入を **言語レベルで禁止する** キーワードがありません。代わりに **型ヒントを付けて意図を明示** し、各変数に意味のある名前を付けることで、再代入しない意図をコードで表現します。

**TypeScript:**
```typescript
// ✅ const で再代入を禁止し、意味のある名前を付ける
damage(): number {
    const basicAttackPower = this.member.power() + this.member.weaponAttack();
    const finalAttackPower = Math.floor(basicAttackPower * (1 + this.member.speed() / 100));
    const reduction = Math.floor(this.enemy.defence / 2);
    const damage = Math.max(0, finalAttackPower - reduction);
    return damage;
}
```

<br>

---

### 🛡️ 引数も不変にする

変数だけでなく、 **メソッドの引数** も `final` にするべきです。引数を途中で書き換えると、元の値が失われてバグの原因になります。

#### ❌ 悪い例（引数を再代入してしまう）

**Java:**
```java
// ❌ 引数 productPrice を途中で上書きしてしまっている
void addPrice(int productPrice) {
    productPrice = totalPrice + productPrice;  // ← 元の商品価格が消えた！
    if (MAX_TOTAL_PRICE < productPrice) {
        throw new IllegalArgumentException("購入金額の上限を超えています。");
    }
}
```

**Python:**
```python
# ❌ 引数を途中で上書き
def add_price(self, product_price: int) -> None:
    product_price = self.total_price + product_price  # ← 元の値が消えた！
    if self.MAX_TOTAL_PRICE < product_price:
        raise ValueError("購入金額の上限を超えています。")
```

**TypeScript:**
```typescript
// ❌ 引数を途中で上書き
addPrice(productPrice: number): void {
    productPrice = this.totalPrice + productPrice; // ← 元の値が消えた！
    if (this.MAX_TOTAL_PRICE < productPrice) {
        throw new Error("購入金額の上限を超えています。");
    }
}
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 元の引数値が失われる | `productPrice` に合計値を再代入しているため、 **元々渡された商品価格** が上書きされて消えてしまう。後から元の価格を参照したくてもできない |
| 2 | 🔴 変数名と中身が一致しない | 再代入後の `productPrice` は「商品価格」ではなく「合計価格」になっている。 **名前と中身のズレ** は読み手を確実に混乱させる |

---

### 🌟 改善版コード（引数を final にする）

**一言でいうと:** 引数に `final` を付けて再代入を禁止し、計算結果は **新しい変数** に格納する。

**Java:**
```java
// ✅ 引数を final にして再代入を防ぐ
void addPrice(final int productPrice) {
    // 新しい変数に計算結果を格納（引数は書き換えない！）
    final int increasedTotalPrice = totalPrice + productPrice;
    if (MAX_TOTAL_PRICE < increasedTotalPrice) {
        throw new IllegalArgumentException("購入金額の上限を超えています。");
    }
    totalPrice = increasedTotalPrice;
}
```

📝 **使い方:**
```java
// productPrice = 500 は最後まで500のまま変わらない
addPrice(500);
// increasedTotalPrice = totalPrice + 500 で計算
// 元の引数が保持されているので、ログ出力やデバッグも安心
```

**Python:**
```python
# ✅ 引数は変更せず、新しい変数に計算結果を入れる
def add_price(self, product_price: int) -> None:
    increased_total_price: int = self.total_price + product_price
    if self.MAX_TOTAL_PRICE < increased_total_price:
        raise ValueError("購入金額の上限を超えています。")
    self.total_price = increased_total_price
```

**TypeScript:**
```typescript
// ✅ 引数は変更せず、新しい変数に計算結果を入れる
addPrice(productPrice: number): void {
    const increasedTotalPrice = this.totalPrice + productPrice;
    if (this.MAX_TOTAL_PRICE < increasedTotalPrice) {
        throw new Error("購入金額の上限を超えています。");
    }
    this.totalPrice = increasedTotalPrice;
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| 変数の使い回し | `tmp` に4回再代入 | `basicAttackPower`, `finalAttackPower` 等の個別変数 | **各変数の意味が名前から一目瞭然** になった |
| 変数の保護 | `int tmp`（再代入自由） | `final int damage`（再代入不可） | **意図しない再代入** がコンパイルエラーで即座に検出される |
| 引数の保護 | `int productPrice`（上書き可能） | `final int productPrice`（上書き不可） | **元の引数値が保持** され、名前と中身のズレが起きない |
| 可読性 | 右辺の式を全部読まないと意味が分からない | 変数名を読むだけで各段階の意味が分かる | **コードレビューやデバッグの速度** が大幅に向上 |

---

### 🎯 まとめ（不変における考え方）

#### 1. 目的（Why）
> **「変数の中身が途中で変わることによるバグを根絶するため」**

* 変数を使い回すと「いつ・どこで・何に変わったか」の追跡が困難になる。
* `final` で再代入を禁止すれば、 **変数の値はコード上の宣言箇所を見るだけで確定** する。

#### 2. 目標（What）
> **「全ての変数が、一度代入されたら最後まで同じ値を持っている状態」**

* コードのどの時点で読んでも、変数の値が予測可能。デバッグ時に「この変数いまいくつ？」と悩まない。

#### 3. 手段（How）
> **「変数・引数・ローカル変数に可能な限り `final` / `const` を付ける」**

| ステップ | ❌ 悪い例 | ⭕️ 良い例 |
|:---:|---|---|
| ローカル変数 | `int tmp = ...;` → `tmp = ...;` → `tmp = ...;` | `final int basicPower = ...;` `final int finalPower = ...;` |
| 引数 | `void method(int price)` → `price = ...;` | `void method(final int price)` → 再代入はコンパイルエラー |
| 命名 | `tmp`, `temp`, `val`, `x` | `basicAttackPower`, `increasedTotalPrice`（計算の意味を表す名前） |

> ※ 「`final` を付けるのが面倒」ではなく、 **「`final` を付けないことで将来のバグを1つ許している」** と考えましょう。
