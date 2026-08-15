# バージョン管理

## 形式

`xxx.yyy.zzz` (メジャー.マイナー.パッチ)。管理場所は **ルート CMakeLists.txt の `project(... VERSION x.y.z)` のみ**。他の場所にバージョン文字列を書かない。

## 更新規則

| 変更内容 | 上げる桁 | 例 |
|---|---|---|
| アーキテクチャ変更・大幅な構造変更 | メジャー (xxx) | 1.4.2 → 2.0.0 |
| 機能追加・既存機能の修正 | マイナー (yyy) | 1.4.2 → 1.5.0 |
| 細かい実装・動作確認用ビルド | パッチ (zzz) | 1.4.2 → 1.4.3 |

- 上位の桁を上げたら下位の桁は 0 にリセットする。
- どの桁を上げるか迷う場合はユーザーに確認する(勝手にメジャーを上げない)。
- 機能追加・修正を完了したら、PR 提出前に必ずバージョンを更新する。

## アプリからの取得方法

CMake の `configure_file` でヘッダを生成し、アプリはそれを include する。

`include/<project>/version.h.in`:

```cpp
#ifndef <PROJECT>_VERSION_H_
#define <PROJECT>_VERSION_H_

namespace <project> {

inline constexpr int kVersionMajor = @PROJECT_VERSION_MAJOR@;
inline constexpr int kVersionMinor = @PROJECT_VERSION_MINOR@;
inline constexpr int kVersionPatch = @PROJECT_VERSION_PATCH@;
inline constexpr char kVersionString[] = "@PROJECT_VERSION@";

}  // namespace <project>

#endif  // <PROJECT>_VERSION_H_
```

`configure_file` の呼び出し方は `project-setup.md` の CMakeLists.txt テンプレートを参照。

使用例:

```cpp
#include "my_project/version.h"
std::cout << "my_project version " << my_project::kVersionString << '\n';
```
