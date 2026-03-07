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

**typescript:**
```typescript
class Class001 {
    method001(): void {}
    method002(): void {}
    method003(): void {}
}
```

---

### 🚨 何がダメなのか？ (連番命名の問題点)

記載されているコードには、以下の問題があります。

#### 1. 意図や目的がわからない無意味な名前
> 対象: `Class001`, `method001`, `method002`, `method003`

クラス名やメソッド名に「001」「002」といった連番が振られていますが、これらは **「そのクラスやメソッドが何をするためのものなのか」** を全く説明していません。
名前から処理の目的や役割を推測することができず、実装の中身を一行ずつ読まないと何を行っているのか理解できないため、コードを読む際の認知的負荷が非常に高くなります。

#### 2. 機能の追加や変更に弱い
> 対象: `method001`〜`method003` の構造

もし「001」と「002」の間に新しいメソッドを追加したくなった場合、番号を振り直す必要が出てきたり、気にせず末尾に「004」として追加することで順番と処理の流れに矛盾が生じる等の不都合が生まれます。
また、処理の中身が変更されたとしても名前は変更されないため、ますます中身と名前が乖離していく原因になります。

---

### 🌟 改善版コード（役割に基づく命名）

クラスやメソッドが **「何を目的としているか（どんな役割を持っているか）」** を明確にする名前に変更します。
ここでは例として、このクラスが「ユーザー登録のフロー」を処理していると仮定して書き直します。

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
    def validate_input(self):
        pass
    
    def save_to_database(self):
        pass
    
    def send_welcome_email(self):
        pass
```

**typescript:**
```typescript
class UserRegistrationService {
    validateInput(): void {}
    saveToDatabase(): void {}
    sendWelcomeEmail(): void {}
}
```

#### 改善のポイント
- `Class001` → **`UserRegistrationService`**: 「クラス001」という無意味な名前から、「ユーザー登録を行う機能」を持つクラスであることが一目でわかる名前にしました。
- `method001` → **`validateInput`**: 「メソッド001」ではなく、「入力を検証する」という具体的な処理内容を表しています。
- `method002` → **`saveToDatabase`**: 「データベースに保存する」という目的を表しています。
- `method003` → **`sendWelcomeEmail`**: 「歓迎メールを送信する」というアクションを表しています。

---

### 🎯 まとめ（連番命名における考え方）

連番命名から脱却し、正しいコードを書くための考え方を「目的・目標・手段」の順に整理します。

#### 1. 目的（Why: なぜこれにこだわるのか？）
* **「名前による意図の伝達と保守性の確保」**
  * プログラムはコンピュータのためだけでなく、人間のために書かれるものです。
  * 名前から役割が推測できれば、他の開発者を始め、未来の自分自身がコードを変更・拡張する際の調査時間を大幅に削減できます。

#### 2. 目標（What: どのような状態を目指すのか？）
* **「自己ドキュメント化されたコード」**
  * コメントや仕様書を読まなくても、クラス名やメソッド名の一覧を見るだけで「この処理では何が行われているか」の全体像や目的が把握できる状態を目指します。

#### 3. 手段（How: どうやって達成するのか？）
* **「意味のある動詞・名詞で具体的に表現する」**
  * ❌ `method001` → ⭕️ `validateInput` (何を検証するのか？)
  * 仮の連番や「1」「2」といった数字が名前に登場しそうになったら、一度立ち止まり **「これらが処理している内容の本質（目的）は何だろうか」** と自問自答して名前を付け直すことが大切です。