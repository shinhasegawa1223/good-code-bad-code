## 何重にもネストしたロジック (深いネスト)

**Java:**
```java
if (0 < member.hitpoint){
    if(member.canAct()){
        if(magic.costMagincPoint <= member.magicPoint){
            member.counsumeMagicPoint(magic.costMagicPoint);
            member.chant(magic);
        }
    }
}
```

**Python:**
```python
if 0 < member.hitpoint:
    if member.can_act():
        if magic.cost_magic_point <= member.magic_point:
            member.consume_magic_point(magic.cost_magic_point)
            member.chant(magic)
```

**TypeScript:**
```typescript
if (0 < member.hitpoint) {
    if (member.canAct()) {
        if (magic.costMagicPoint <= member.magicPoint) {
            member.consumeMagicPoint(magic.costMagicPoint);
            member.chant(magic);
        }
    }
}
```

---

### 🚨 何がダメなのか？ (ネストしたロジックの問題点)

<br>

#### 1. 🔴 可読性の著しい低下 (Arrow Anti-Pattern)
> 対象: 連続する3つの `if` 文

コードの形が **右に向かって矢印のように** 深く入り込んでいる 👇

```
if (...) {
    if (...) {
        if (...) {      ← どんどん右に伸びる！ 🏹
            本当の処理
        }
    }
}
```

→ 「今どの条件のもとで動いているのか」を追うのが難しく、**認知負荷が非常に高い** 。

> 💡 **用語: Arrow Anti-Pattern（矢印アンチパターン）とは？**
> ネストが深くなりインデントが右に伸び、コードの形が「→（矢印）」に見えること。
> 可読性が著しく低下するため、避けるべきパターン。

<br>

#### 2. 🔴 本質的な処理（正常系）が埋もれる
> 対象: `member.chant(magic);` の実行部分

| 状態 | 説明 |
|:---:|---|
| 🎯 **やりたいこと** | 「魔法を詠唱する」（`chant`）← **これが本題** |
| 😱 **現状** | 3層の条件チェックの **奥底に埋もれている** |

→ 重要な処理にたどり着く前に、読み手は3つの条件を **頭の中で記憶し続ける** 必要がある。

---

### 🌟 改善版コード（早期リターン / ガード節）

> 💡 **用語: 早期リターン / ガード節（Early Return / Guard Clause）とは？**
> メソッドの冒頭で「ダメなケース」を先にチェックして即 `return` で抜けるテクニック。
> 正常系のコードをネストさせずに **フラットに書ける** 。

**一言でいうと:** 「ダメなケースを先に弾く」→ 残った正常系が **ネストなし** で読める。

**Java:**
```java
// ① HPが0以下 → 弾く
if (member.hitpoint <= 0) return;
// ② 行動不能 → 弾く
if (!member.canAct()) return;
// ③ MP不足 → 弾く
if (member.magicPoint < magic.costMagicPoint) return;

// ✅ 全条件クリア → メインの処理（魔法の詠唱）
member.consumeMagicPoint(magic.costMagicPoint);
member.chant(magic);
```

**Python:**
```python
# ① HPが0以下 → 弾く
if member.hitpoint <= 0: return
# ② 行動不能 → 弾く
if not member.can_act(): return
# ③ MP不足 → 弾く
if member.magic_point < magic.cost_magic_point: return

# ✅ 全条件クリア → メインの処理（魔法の詠唱）
member.consume_magic_point(magic.cost_magic_point)
member.chant(magic)
```

**TypeScript:**
```typescript
// ① HPが0以下 → 弾く
if (member.hitpoint <= 0) return;
// ② 行動不能 → 弾く
if (!member.canAct()) return;
// ③ MP不足 → 弾く
if (member.magicPoint < magic.costMagicPoint) return;

// ✅ 全条件クリア → メインの処理（魔法の詠唱）
member.consumeMagicPoint(magic.costMagicPoint);
member.chant(magic);
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| **条件チェック** | `if (OK) { if (OK) { if (OK) { ... }}}` | `if (NG) return;` を3回 |
| **ネストの深さ** | 3段 🏹 | **0段**（フラット）✨ |
| **本質的な処理** | 3層の奥底に埋もれている | **最下層にインデントなし** で登場 |
| **読み方** | 「条件OK → 次の条件OK → 次も…」 | 「NG→弾く、NG→弾く、残った ＝ OK」 |

---

### 🎯 まとめ（ネスト排除における考え方）

#### 1. 目的（Why）
> **「認知負荷を下げるため」**

* 人間の脳が同時に記憶できる前提条件には **限界がある**
* 深いネスト → 「今どの条件下？」を常に覚える → **バグの温床**

#### 2. 目標（What）
> **「正常系が一直線に読める状態」**

* 上から下にスッと読んだとき、メインの処理が **インデントなし** で書かれている

#### 3. 手段（How）
> **「ガード節（早期リターン）の活用」**

| ステップ | 内容 |
|:---:|---|
| **「条件OK→続行」を逆転** | ❌ `if (条件OK) { 続く... }` → ⭕️ `if (条件NG) return;` |
| **先頭で弾く** | 処理の最初に「ダメなケース」を見つけたら即 `return` で脱出 |
| **残りがフラット** | 後続のコードは **ネストなし** で本題だけを書ける |
