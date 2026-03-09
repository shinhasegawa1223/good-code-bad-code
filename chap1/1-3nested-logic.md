## 何十にもネストしたロジック (深いネスト)

**Java:**
```java
if (0 < member.hitpoint){
    if(member.canAct()){
        if(magic.costMagicPoint <= member.magicPoint){
            member.consumeMagicPoint(magic.costMagicPoint);
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

記載されているコードには、以下の問題があります。

#### 1. 可読性の著しい低下 (Arrow Anti-Pattern)
> 対象: 連続する3つの `if` 文

何重にも `if` 文が入れ子（ネスト）になっているため、コードの形が右に向かって矢印のように深く入り込んでいます。
**「今どの条件を満たしてこの行が実行されているのか」** を視覚的に追うのが難しくなり、コードを読むための「認知負荷」が非常に高くなります。

#### 2. 本質的な処理（正常系）が埋もれる
> 対象: `member.chant(magic);` の実行部分

この処理で本当にやりたいこと（このメソッドの主要な目的）は「魔法を詠唱する (`chant`) こと」ですが、魔法を唱えるための「事前条件のチェック」の奥底にメインの処理が埋もれてしまっています。
重要な処理にたどり着くまでに、読み手はいくつもの条件を頭の中で記憶し続けなければなりません。

---

### 🌟 改善版コード（早期リターン / ガード節）

本質的な処理のジャマをする「異常系（条件を満たさないケース）」を、処理の先頭で **早期リターン（Early Return / Guard Clause）** して弾くことで、ネストを浅く（フラットに）保ちます。

**Java:**
```java
if (member.hitpoint <= 0) return;
if (!member.canAct()) return;
if (member.magicPoint < magic.costMagicPoint) return;

member.consumeMagicPoint(magic.costMagicPoint);
member.chant(magic);
```

**Python:**
```python
if member.hitpoint <= 0: return
if not member.can_act(): return
if member.magic_point < magic.cost_magic_point: return

member.consume_magic_point(magic.cost_magic_point)
member.chant(magic)
```

**TypeScript:**
```typescript
if (member.hitpoint <= 0) return;
if (!member.canAct()) return;
if (member.magicPoint < magic.costMagicPoint) return;

member.consumeMagicPoint(magic.costMagicPoint);
member.chant(magic);
```

#### 改善のポイント
- `if (前提条件)` → **`if (前提条件を満たさない) return;`**: 条件を満たす場合に処理を進めるのではなく、満たさない（異常な）場合に即座に処理を終了させて波括弧 `{}` をなくしました。
- `member.chant(magic);` のフラット化: 全ての前提チェックが終わった状態で正常系のコードが最下層（インデントなし）に書かれるため、一切ネストに潜らず本質的な処理を読むことができます。

---

### 🎯 まとめ（ネスト排除における考え方）

深いネストから脱却し、正しいコードを書くための考え方を「目的・目標・手段」の順に整理します。

#### 1. 目的（Why: なぜこれにこだわるのか？）
* **「認知負荷を下げるため」**
  * 人間の脳が同時に記憶できる前提条件の数には限界があります。
  * ネストが深いコードは「今どの条件のもとで動いているか」を常に覚えながら読む必要があり、バグの見落としや修正ミスの温床になります。

#### 2. 目標（What: どのような状態を目指すのか？）
* **「正常系が一直線に読める状態」**
  * コードを上から下にスッと読んだ時、エラー処理や前提チェックに視界を妨げられず、1番やりたいメインの処理が「インデントなし（浅いネスト）」で素直に書かれている状態を目指します。

#### 3. 手段（How: どうやって達成するのか？）
* **「ガード節（早期リターン）の活用」**
  * ❌ `if (条件OK) { 続く... }` → ⭕️ `if (条件NG) { return }`
  * 波括弧の中で正常系の処理を書くのではなく、処理の最初に「これ以上進めない条件」を見つけたらすぐに脱出（`return`）させることで、後続のコードをネストさせずに済みます。
