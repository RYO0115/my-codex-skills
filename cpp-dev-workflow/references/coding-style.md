# コーディングスタイル (Google C++ Style Guide 準拠)

コードを書く・修正するときは常にこの規則に従う。迷ったら Google C++ Style Guide に従う。

## 命名規則

| 対象 | 規則 | 例 |
|---|---|---|
| ファイル名 | snake_case | `motor_controller.cpp` / `.h` |
| 型 (class/struct/enum/alias) | PascalCase | `MotorController` |
| 関数・メソッド | PascalCase | `ComputeSpeed()` |
| 変数・引数 | snake_case | `wheel_radius` |
| メンバ変数 | snake_case + 末尾 `_` | `current_speed_` |
| 定数 (constexpr/const) | `k` + PascalCase | `kMaxRetryCount` |
| enum の値 | `k` + PascalCase | `kIdle`, `kRunning` |
| 名前空間 | snake_case | `motor_control` |
| マクロ (原則使わない) | UPPER_SNAKE_CASE | `MY_PROJECT_ASSERT` |

## ヘッダの規則

- すべてのヘッダにインクルードガード: `<PROJECT>_<PATH>_<FILE>_H_` 形式(`#pragma once` ではなく `#ifndef` 形式)。
- include 順: (1) 対応するヘッダ → (2) C システム → (3) C++ 標準 → (4) 外部ライブラリ → (5) プロジェクト内。グループ間は空行。
- 前方宣言より `#include` を優先するが、循環参照の回避に限り前方宣言を許可(design-rules.md 参照)。
- ヘッダに `using namespace` を書かない。

## コードの規則

- インデント 2 スペース、1 行 80 文字目安(最大 100)。
- `auto` は型が右辺から明らかな場合のみ。
- ポインタより参照・スマートポインタ(`std::unique_ptr` 優先、共有が必要な場合のみ `std::shared_ptr`)。生 `new`/`delete` 禁止。
- 所有しないポインタを渡す場合は生ポインタでよい(null を取りうる場合のみ)。
- 引数: 入力は値または `const&`、出力は戻り値で返す(出力引数を避ける)。
- 例外よりエラー戻り値(`absl::Status` / `std::optional` / 自前の Result 型)を優先。プロジェクトの既存方針があればそれに従う。
- キャストは C++ スタイル (`static_cast` 等) のみ。C スタイルキャスト禁止。
- `NULL`/`0` ではなく `nullptr`。
- 単一引数コンストラクタには `explicit`。
- オーバーライドには必ず `override`。

## .clang-format (プロジェクトルートに置く)

```yaml
BasedOnStyle: Google
ColumnLimit: 100
```

フォーマットは手で整えず `clang-format -i <files>` を実行する。
