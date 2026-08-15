# 静的解析 (clang-tidy)

## 実行タイミング

- **機能を1つ実装するたび**に実行する(テスト全件グリーンの後)。
- **PR 提出前**に必ず手元で全ファイルに実行し、警告ゼロ(オールグリーン)を確認する。CI でも自動実行されるが、CI 任せにしない。

## .clang-tidy (プロジェクトルートに置く)

```yaml
Checks: >
  bugprone-*,
  cert-*,
  clang-analyzer-*,
  concurrency-*,
  cppcoreguidelines-*,
  google-*,
  misc-*,
  modernize-*,
  performance-*,
  readability-*,
  -modernize-use-trailing-return-type,
  -readability-identifier-length
WarningsAsErrors: '*'
HeaderFilterRegex: '(include|src)/.*'
CheckOptions:
  - key: readability-identifier-naming.ClassCase
    value: CamelCase
  - key: readability-identifier-naming.FunctionCase
    value: CamelCase
  - key: readability-identifier-naming.VariableCase
    value: lower_case
  - key: readability-identifier-naming.PrivateMemberSuffix
    value: '_'
  - key: readability-identifier-naming.ConstexprVariablePrefix
    value: 'k'
  - key: readability-identifier-naming.ConstexprVariableCase
    value: CamelCase
```

## 実行コマンド

`CMAKE_EXPORT_COMPILE_COMMANDS=ON` でビルド済みであること(project-setup.md のテンプレートは設定済み)。

```bash
# 変更したファイルに対して実行(通常はこれ)
clang-tidy -p build src/foo.cpp src/bar.cpp

# 全ファイル(PR 前)
run-clang-tidy -p build

# 自動修正可能なものを修正
clang-tidy -p build --fix src/foo.cpp
```

## 警告への対応

- 警告は**すべて修正**する。抑制 (`// NOLINT`) は原則禁止。
- どうしても抑制が必要な場合(外部ライブラリ起因、false positive)は `// NOLINT(check-name)` と理由コメントを付け、ユーザーに報告する。
- チェック自体の無効化(.clang-tidy の変更)は勝手に行わず、ユーザーに提案して承認を得る。
