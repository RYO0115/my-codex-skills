# アーキテクチャ・依存方向の詳細

## レイヤード構成

```
src/<package>/
├── core/          # 共通基盤（DB接続・ベースモデル・設定・例外）
└── features/      # 機能別モジュール
    ├── feature_a/
    ├── feature_b/
    └── ...
```

- `core/` は特定の機能に依存しない汎用基盤のみを置く。
- 各 feature は自分のディレクトリ内に router / service / model / schema などを閉じ込める。

## 依存方向のルール

- 基本方向は **`core ← features/*`**（features が core を import してよいが、逆は禁止）。
- features 同士に依存が必要な場合は**一方向のみ**許可し、循環を作らない。
- 依存グラフは設計書に明記し、新しい依存を足すときはグラフを更新する。

### 依存方向の例（プロジェクトごとに定義する）

```
core ← feature_a, feature_b, feature_c
feature_a ← feature_b      # b は a に依存してよい
feature_b ← feature_c      # c は b に依存してよい
# 逆方向（a → b など）・循環は禁止
```

## 共通化の指針

- 2 箇所以上で同じ処理が必要になったら `core/` に切り出す。
- ただし「たまたま似ているだけ」の早すぎる共通化は避け、意味的に同一の処理だけをまとめる。
