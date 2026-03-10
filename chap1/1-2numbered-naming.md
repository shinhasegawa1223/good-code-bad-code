## 連番命名

**Java:**
```java
class Class001 {
    void method001();
    void method002();
    void method003();
}
```

**Python:**
```python
class Class001:
    def method_001(self):
        pass
    def method_002(self):
        pass
    def method_003(self):
        pass
```

**TypeScript:**
```typescript
class Class001 {
    method001(): void {}
    method002(): void {}
    method003(): void {}
}
```

---

### 🚨 何がダメなのか？ (連番命名の問題点)

<br>

#### 1. 🔴 意図や目的がわからない無意味な名前
> 対象: `Class001`, `method001`, `method002`, `method003`

| 名前 | 問題 |
|---|---|
| `Class001` | 「クラス001」→ **何をするクラスなのか** 全く不明 |
| `method001` | 「メソッド001」→ 中身を1行ずつ読まないと処理が分からない |

→ コードを読む際の **認知的負荷が非常に高い** 。

<br>

#### 2. 🔴 機能の追加や変更に弱い
> 対象: `method001`〜`method003` の構造

| やりたいこと | 困ること |
|---|---|
| 001と002の間に新メソッドを追加 | 番号を振り直す必要がある 😓 |
| 末尾に004を追加 | 順番と処理の流れが **矛盾** する |
| 処理の中身を変更 | 名前は変わらない → **名前と中身が乖離** していく |

> 💡 **用語: 自己文書化コード（Self-Documenting Code）とは？**
> コメントや仕様書を読まなくても、変数名やメソッド名だけで **「何をしているか」が分かる** コードのこと。
> 連番命名 (`method001`) はこの真逆 → 名前を読んでも何も伝わらない。

---

### 🌟 改善版コード（役割に基づく命名）

**一言でいうと:** 「このメソッドは何をするのか？」を **そのまま名前にする** 。

ここでは「ユーザー登録のフロー」を処理していると仮定して書き直します。

**Java:**
```java
class UserRegistrationService {
    void validateInput();
    void saveToDatabase();
    void sendWelcomeEmail();
}
```

**Python:**
```python
class UserRegistrationService:
    def validate_input(self) -> None:
        pass
    
    def save_to_database(self) -> None:
        pass
    
    def send_welcome_email(self) -> None:
        pass
```

**TypeScript:**
```typescript
class UserRegistrationService {
    validateInput(): void {}
    saveToDatabase(): void {}
    sendWelcomeEmail(): void {}
}
```

<br>

#### ✅ 改善のビフォー・アフター

| 項目 | ❌ 改善前 | ✅ 改善後 | 変わったこと |
|---|---|---|---|
| クラス名 | `Class001` | `UserRegistrationService` | 「ユーザー登録」のクラスだと一目で分かる |
| メソッド1 | `method001` | `validateInput` | 「入力を検証する」という処理内容 |
| メソッド2 | `method002` | `saveToDatabase` | 「DBに保存する」という目的 |
| メソッド3 | `method003` | `sendWelcomeEmail` | 「歓迎メールを送る」というアクション |

---

### 🎯 まとめ（連番命名における考え方）

#### 1. 目的（Why）
> **「名前による意図の伝達と保守性の確保」**

* プログラムはコンピュータだけでなく **人間のため** に書かれる
* 名前から役割が推測できれば、未来の自分の **調査時間を大幅に削減** できる

#### 2. 目標（What）
> **「自己ドキュメント化されたコード」**

* コメントや仕様書を読まなくても、名前一覧を見るだけで **「何が行われているか」が把握できる** 状態

#### 3. 手段（How）
> **「意味のある動詞・名詞で具体的に表現する」**

| ステップ | 内容 |
|:---:|---|
| **名前に数字が出てきたら立ち止まる** | ❌ `method001` → ⭕️ `validateInput` |
| **「この処理の目的は何か？」を自問する** | 名前が処理の **本質（目的）** を表しているか確認する |