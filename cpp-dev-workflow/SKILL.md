---
name: cpp-dev-workflow
description: C++プロジェクトの設計・実装・テスト・静的解析の開発ワークフロー。CMake / Google Test / clang-tidy を使う C++ コードの新規プロジェクト作成、機能追加、バグ修正、リファクタリング時に必ず使用する。Use when creating, modifying, or reviewing C++ code with CMake, Google Test, or clang-tidy.
---

# C++ 開発ワークフロー

C++ プロジェクトで作業する際は、このワークフローに**必ず**従うこと。
詳細な手順は `references/` に分割されている。**今のタスクに必要なファイルだけ**を読むこと(下の表を参照)。

## 絶対ルール(常に適用)

1. **テストファースト**: 機能実装の前に Google Test でテストを書く。実装 → テストの順は禁止。
2. **1機能ごとに検証**: 機能を1つ実装するたびに (a) 新規テスト、(b) 既存テスト全件(リグレッション)、(c) clang-tidy をすべてグリーンにする。
3. **ハードコード禁止**: マジックナンバー・固定パス・固定文字列を埋め込まない。必要になった場合は実装せず、設計段階で理由をユーザーに報告して指示を仰ぐ。
4. **グローバル変数禁止**: 必要になった場合は実装せず、設計段階で理由をユーザーに報告して指示を仰ぐ。
5. **Google C++ Style Guide 準拠**: 命名・フォーマットは `references/coding-style.md` の規則に従う。
6. **重複実装禁止**: 同じ処理を2箇所に書かない。書く前に既存コードを検索し、共通化する。
7. **循環参照禁止**: モジュール間の依存は一方向のみ。相互 include・相互所有は設計で排除する。

## ワークフロー

機能追加・修正は必ずこの順で進める:

```
1. 設計        … 既存コード調査 → 設計方針を決定(ハードコード/グローバル変数が
                  必要ならここで報告して停止)
2. テスト作成  … 正常系・異常系の単体/機能テストを先に書く(この時点では失敗してよい)
3. 実装        … テストが通る最小限の実装
4. 検証        … 新規テスト + 既存テスト全件 + clang-tidy をオールグリーンに
5. バージョン更新 … CMakeLists.txt のバージョンを規則に従って上げる
6. PR 前チェック  … references/checklists.md の PR チェックリストを完走
```

## Reference の読み込み表

| タスク | 読むファイル |
|---|---|
| 新規プロジェクト作成 | `references/project-setup.md`, `references/versioning.md` |
| クラス・モジュール設計 | `references/design-rules.md` |
| マルチスレッドを含む実装 | `references/design-rules.md`, `references/thread-safety.md` |
| テスト作成・実行 | `references/testing.md` |
| コードを書く(常時) | `references/coding-style.md` |
| clang-tidy の実行・設定 | `references/static-analysis.md` |
| バージョン番号の更新 | `references/versioning.md` |
| CI (GitHub Actions) 設定 | `references/ci.md` |
| 機能完成時・PR 提出前 | `references/checklists.md` |

読み込みの目安: コードを書くタスクでは `coding-style.md` + タスク該当ファイルの **2〜3 ファイルまで**に留め、不要なファイルは読まない。
