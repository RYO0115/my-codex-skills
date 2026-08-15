# 新規プロジェクトのセットアップ

## ディレクトリ構成

新規プロジェクトは必ずこの構成で作成する:

```
project_root/
├── CMakeLists.txt          # ルート。バージョンはここで一元管理
├── .clang-format           # coding-style.md 参照
├── .clang-tidy             # static-analysis.md 参照
├── include/<project>/      # 公開ヘッダ (.h)
├── src/                    # 実装 (.cpp) と非公開ヘッダ
├── tests/                  # Google Test のテストコード
│   └── CMakeLists.txt
└── .github/workflows/      # ci.md 参照
```

## ルート CMakeLists.txt テンプレート

```cmake
cmake_minimum_required(VERSION 3.20)

# バージョンはここが唯一の管理場所 (versioning.md 参照)
project(my_project VERSION 0.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)  # clang-tidy に必須

# アプリがバージョンを取得できるようにヘッダを生成 (versioning.md 参照)
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/include/${PROJECT_NAME}/version.h.in
  ${CMAKE_CURRENT_BINARY_DIR}/generated/${PROJECT_NAME}/version.h
  @ONLY
)

add_library(${PROJECT_NAME}_lib
  # src/*.cpp をここに列挙する (GLOB は使わない)
)
target_include_directories(${PROJECT_NAME}_lib PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_BINARY_DIR}/generated
)
target_compile_options(${PROJECT_NAME}_lib PRIVATE
  -Wall -Wextra -Wpedantic -Werror
)

add_executable(${PROJECT_NAME} src/main.cpp)
target_link_libraries(${PROJECT_NAME} PRIVATE ${PROJECT_NAME}_lib)

enable_testing()
add_subdirectory(tests)
```

規則:

- ソースファイルは **GLOB ではなく明示的に列挙**する(追加漏れ・意図しないビルドを防ぐ)。
- ロジックは `_lib` ライブラリに置き、`main.cpp` は起動処理のみにする(テスト対象をライブラリとしてリンクするため)。
- `-Werror` を外さない。警告が出たら修正する。

## ビルドコマンド

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j
ctest --test-dir build --output-on-failure
```
