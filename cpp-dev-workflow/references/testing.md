# テスト (Google Test / テストファースト)

## 鉄則

- **実装より先にテストを書く。** 手順は必ず: テスト作成 → 実装 → テスト実行。逆順は禁止。
- 新機能ごとに**正常系と異常系の両方**を書く(異常系: 不正入力、境界値、空データ、エラー戻り値)。
- 機能を1つ実装するたびに**既存テスト全件**を実行してオールグリーンを確認する(リグレッションテスト)。
- 失敗したテストを削除・スキップ・期待値書き換えで「通す」ことは禁止。実装を直す。テスト自体の誤りを直す場合はその旨を報告する。

## TDD の手順

1. 追加する機能のインターフェース(公開ヘッダ)を決める。
2. `tests/` にテストを書く: 正常系・異常系・境界値。
3. ビルドしてテストが**失敗する**ことを確認する(まだ実装がないため)。
4. テストが通る最小限の実装を書く。
5. 新規テスト → 既存テスト全件の順で実行し、オールグリーンにする。
6. clang-tidy を実行する(static-analysis.md)。

## セットアップ (tests/CMakeLists.txt)

```cmake
include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG v1.15.2
)
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

add_executable(unit_tests
  # test_*.cpp をここに列挙
)
target_link_libraries(unit_tests PRIVATE
  ${PROJECT_NAME}_lib
  GTest::gtest_main
  GTest::gmock
)

include(GoogleTest)
gtest_discover_tests(unit_tests)
```

## テストの書き方

- ファイル名: `test_<対象>.cpp`(例: `test_motor_controller.cpp`)。
- テスト名: `TEST(<対象>Test, <条件>_<期待結果>)` 形式で内容が読めるように(例: `TEST(SpeedCalcTest, NegativeInput_ReturnsError)`)。
- 1テスト1検証事項。Arrange → Act → Assert の順で書く。
- 外部依存(デバイス、時刻、I/O)はインターフェース経由で gMock のモックに差し替える(design-rules.md の依存性注入)。
- 浮動小数点は `EXPECT_DOUBLE_EQ` / `EXPECT_NEAR` を使う(`EXPECT_EQ` 禁止)。

## 実行コマンド

```bash
# 全件(リグレッション)
ctest --test-dir build --output-on-failure

# 単一テストの実行(デバッグ時)
./build/tests/unit_tests --gtest_filter='SpeedCalcTest.NegativeInput_ReturnsError'

# パターン指定
./build/tests/unit_tests --gtest_filter='SpeedCalcTest.*'
```

マルチスレッドのコードを含む場合は TSan ビルドでも実行する(thread-safety.md)。
