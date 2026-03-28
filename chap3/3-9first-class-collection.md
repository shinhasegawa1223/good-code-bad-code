## ファーストクラスコレクション（First Class Collection）

> 💡 **用語: ファーストクラスコレクション（First Class Collection）とは？**
> コレクション（リストや配列）を **そのまま使わず、専用のクラスで包む** 設計パターン。コレクションに対する操作（追加・削除・検索・集計）を全てそのクラスの中に閉じ込める。

### ❌ 悪い例（コレクションをむき出しで使う）

**Java:**
```java
// ❌ List をむき出しで使っている → 操作ルールが散在する
class Party {
    List<Member> members = new ArrayList<>();
}
```

```java
// 使う側のコード（あちこちに同じチェックが散らばる）
Party party = new Party();

// パーティの上限チェック → 使う側が毎回書く
if (party.members.size() < 4) {
    party.members.add(newMember);
}

// 別の場所でも同じチェック... 書き忘れたらバグ！
if (party.members.size() < 4) {
    party.members.add(anotherMember);
}

// ❌ 外部から直接操作できてしまう
party.members.clear();  // パーティ全員消去！ルール無視！
```

**Python:**
```python
# ❌ リストをむき出しで使っている
class Party:
    def __init__(self) -> None:
        self.members: list['Member'] = []

# 使う側が毎回上限チェックする必要がある
party = Party()
if len(party.members) < 4:
    party.members.append(new_member)

party.members.clear()  # ❌ ルール無視で全消去可能
```

**TypeScript:**
```typescript
// ❌ 配列をむき出しで使っている
class Party {
    members: Member[] = [];
}

const party = new Party();
if (party.members.length < 4) {
    party.members.push(newMember);
}
party.members.length = 0; // ❌ ルール無視で全消去可能
```

---

### 🚨 何がダメなのか？

| # | 問題 | 説明 |
|---|---|---|
| 1 | 🔴 ルールの散在 | 「パーティは最大4人」等のルールが **使う側のあちこちにコピペ** される。書き忘れたら上限を超えた不正状態に |
| 2 | 🔴 外部からの不正操作 | `members.clear()` や `members.remove()` が自由にできてしまい、 **クラスが知らないうちに中身が変わる** |
| 3 | 🔴 集計ロジックの重複 | 「HPが最も低いメンバーを探す」等の処理が、使う箇所ごとに重複して書かれる |

---

### 🌟 改善版コード

**一言でいうと:** `List<Member>` を直接公開せず、 **`Party` クラスの中にコレクションを隠蔽** し、操作は全てメソッド経由にする。

**Java:**
```java
// ✅ ファーストクラスコレクション：コレクションを専用クラスで包む
class Party {
    private static final int MAX_SIZE = 4;
    private final List<Member> members;

    Party() {
        this.members = new ArrayList<>();
    }

    // ✅ 追加ルールをクラスの中に閉じ込める
    void add(final Member member) {
        if (members.size() >= MAX_SIZE) {
            throw new IllegalStateException("パーティは最大" + MAX_SIZE + "人です。");
        }
        members.add(member);
    }

    // ✅ 読み取り専用のコピーを返す（外部からの直接操作を防ぐ）
    List<Member> getMembers() {
        return Collections.unmodifiableList(members);
    }

    // ✅ 集計ロジックもクラス内に閉じ込める
    Member lowestHpMember() {
        return members.stream()
            .min(Comparator.comparingInt(Member::getHp))
            .orElseThrow();
    }

    int size() { return members.size(); }
    boolean isEmpty() { return members.isEmpty(); }
}
```

📝 **使い方:**
```java
Party party = new Party();
party.add(warrior);    // ✅ ルールチェック済みで追加
party.add(mage);
party.add(healer);
party.add(thief);
// party.add(extra);   // ❌ IllegalStateException: パーティは最大4人です。

Member weakest = party.lowestHpMember(); // ✅ 集計もクラスに任せる

// party.getMembers().clear();  // ❌ UnsupportedOperationException（変更不可！）
```

**Python:**
```python
# ✅ ファーストクラスコレクション
class Party:
    MAX_SIZE: int = 4

    def __init__(self) -> None:
        self._members: list['Member'] = []

    def add(self, member: 'Member') -> None:
        if len(self._members) >= self.MAX_SIZE:
            raise ValueError(f"パーティは最大{self.MAX_SIZE}人です。")
        self._members.append(member)

    @property
    def members(self) -> tuple['Member', ...]:
        """読み取り専用（タプルで返すことで外部からの変更を防ぐ）"""
        return tuple(self._members)

    def lowest_hp_member(self) -> 'Member':
        if not self._members:
            raise ValueError("パーティが空です。")
        return min(self._members, key=lambda m: m.hp)

    def size(self) -> int:
        return len(self._members)
```

**TypeScript:**
```typescript
// ✅ ファーストクラスコレクション
class Party {
    private static readonly MAX_SIZE = 4;
    private readonly members: Member[] = [];

    add(member: Member): void {
        if (this.members.length >= Party.MAX_SIZE) {
            throw new Error(`パーティは最大${Party.MAX_SIZE}人です。`);
        }
        this.members.push(member);
    }

    getMembers(): readonly Member[] {
        return [...this.members]; // コピーを返す
    }

    lowestHpMember(): Member {
        if (this.members.length === 0) throw new Error("パーティが空です。");
        return this.members.reduce((min, m) => m.hp < min.hp ? m : min);
    }

    get size(): number { return this.members.length; }
}
```

| 項目 | ❌ 改善前 | ✅ 改善後 |
|---|---|---|
| 上限チェック | 使う側が毎回 `if (size < 4)` を書く | `add()` メソッドが自動でチェック |
| 外部からの操作 | `members.clear()` で勝手に消去可能 | 読み取り専用コピーを返すので変更不可 |
| 集計ロジック | 使う箇所ごとにコピペ | `lowestHpMember()` 等のメソッドに集約 |

<br>
