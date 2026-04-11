# 📘 良いコード / 悪いコードで学ぶ設計入門

「良いコード/悪いコードで学ぶ設計入門」の学習ノートです。  
各章ごとに **悪い例（アンチパターン）** と **良い例（改善版）** を Java / Python / TypeScript の3言語で記載しています。

---

## 📁 目次

### Chapter 1 ― 悪いコードの基本
| # | テーマ | ファイル |
|---|---|---|
| 1-1 | 技術駆動命名 | [1-1technology-driven-naming.md](chap1/1-1technology-driven-naming.md) |
| 1-2 | 連番命名 | [1-2numbered-naming.md](chap1/1-2numbered-naming.md) |
| 1-3 | ネストの深いロジック | [1-3nested-logic.md](chap1/1-3nested-logic.md) |
| 1-4 | 複雑なネストロジック | [1-4complex-nested-logic.md](chap1/1-4complex-nested-logic.md) |
| 1-5 | データクラス | [1-5data-class.md](chap1/1-5data-class.md) |
| 1-6 | ロジックの散在 | [1-6scattered-logic.md](chap1/1-6scattered-logic.md) |
| 1-7 | 未初期化・不正状態 | [1-7uninitialized-invalid-state.md](chap1/1-7uninitialized-invalid-state.md) |

### Chapter 2 ― 設計の初歩
| # | テーマ | ファイル |
|---|---|---|
| 2-1 | 省略命名 | [2-1omitted-naming.md](chap2/2-1omitted-naming.md) |
| 2-2 | 再代入される変数 | [2-2reassigned-variable.md](chap2/2-2reassigned-variable.md) |
| 2-3 | メソッドの抽出 | [2-3extract-methods.md](chap2/2-3extract-methods.md) |
| 2-4 | 関心の分離 | [2-4separation-of-concerns.md](chap2/2-4separation-of-concerns.md) |
| 2-5 | マジックナンバー | [2-5magic-numbers.md](chap2/2-5magic-numbers.md) |
| 2-6 | 値オブジェクト | [2-6value-object.md](chap2/2-6value-object.md) |

### Chapter 3 ― クラス設計
| # | テーマ | ファイル |
|---|---|---|
| 3-1 | カプセル化 | [3-1encapsulation.md](chap3/3-1encapsulation.md) |
| 3-2 | 関心の分離 | [3-2separation-of-concerns.md](chap3/3-2separation-of-concerns.md) |
| 3-3 | ポリモーフィズム | [3-3polymorphism.md](chap3/3-3polymorphism.md) |
| 3-4 | 成熟したクラス設計 | [3-4mature-class-design.md](chap3/3-4mature-class-design.md) |
| 3-5 | 完全コンストラクタ | [3-5complete-constructor.md](chap3/3-5complete-constructor.md) |
| 3-6 | 値オブジェクト | [3-6value-object.md](chap3/3-6value-object.md) |
| 3-7 | ストラテジパターン | [3-7strategy-pattern.md](chap3/3-7strategy-pattern.md) |
| 3-8 | ポリシーパターン | [3-8policy-pattern.md](chap3/3-8policy-pattern.md) |
| 3-9 | ファーストクラスコレクション | [3-9first-class-collection.md](chap3/3-9first-class-collection.md) |
| 3-10 | スプラウトクラス | [3-10sprout-class.md](chap3/3-10sprout-class.md) |

### Chapter 4 ― 不変
| # | テーマ | ファイル |
|---|---|---|
| 4-1 | 不変（イミュータビリティ） | [4-1immutability.md](chap4/4-1immutability.md) |
| 4-2 | 可変がもたらす副作用 | [4-2mutable-side-effects.md](chap4/4-2mutable-side-effects.md) |
| 4-3 | 不変と可変の使い分け | [4-3immutable-mutable-balance.md](chap4/4-3immutable-mutable-balance.md) |

### Chapter 5 ― 低凝集
| # | テーマ | ファイル |
|---|---|---|
| 5-1 | プリミティブ型執着 | [5-1primitive-obsession.md](chap5/5-1primitive-obsession.md) |

---

## 📝 フォーマット

各ノートは以下の統一フォーマットで記載しています：

1. **❌ 悪い例** — Java / Python / TypeScript のコード例
2. **🚨 何がダメなのか？** — 問題点をテーブルで整理
3. **🌟 改善版コード** — 3言語の改善例 + 📝使い方
4. **✅ ビフォーアフター** — 比較テーブル
5. **🎯 まとめ** — Why / What / How
