# CI (GitHub Actions)

PR 提出時にリグレッションテストと clang-tidy が自動実行される。新規プロジェクトでは以下を `.github/workflows/ci.yml` として作成する。

**注意**: CI があっても、PR 提出前に手元でテスト全件と clang-tidy をオールグリーンにすること(checklists.md)。CI は最終確認であり、赤い状態で PR を出すことは禁止。

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install dependencies
        run: sudo apt-get update && sudo apt-get install -y clang-tidy

      - name: Configure
        run: cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug

      - name: Build
        run: cmake --build build -j

      - name: Run tests (regression)
        run: ctest --test-dir build --output-on-failure

      - name: Run clang-tidy
        run: run-clang-tidy -p build

  tsan:
    # マルチスレッドコードを含むプロジェクトのみ有効化
    if: ${{ false }}  # 有効化するときはこの行を削除
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Configure (TSan)
        run: >
          cmake -S . -B build-tsan -DCMAKE_BUILD_TYPE=Debug
          -DCMAKE_CXX_FLAGS="-fsanitize=thread -g"
      - name: Build
        run: cmake --build build-tsan -j
      - name: Run tests under TSan
        run: ctest --test-dir build-tsan --output-on-failure
```

- マルチスレッドコードを導入した時点で `tsan` ジョブを有効化する(`if: ${{ false }}` の行を削除)。
- CI が落ちた場合は、手元で同じコマンドを再現して修正してから push する。
