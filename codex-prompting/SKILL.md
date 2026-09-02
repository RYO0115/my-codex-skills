---
name: codex-prompting
description: >-
  Codex向けの指示を、精度を保ちながら短く・明確に設計または改善するスキル。
  「Codexのpromptを最適化して」「token消費を抑えたい」「reasoning effortを選びたい」
  「指示が長い・過剰に確認する・勝手にscopeを広げる」といった依頼で使う。
  一般的な文章添削や、Codex以外のモデル専用promptには使わない。
---

# Codex prompting

Codexへの指示を「目的達成に必要な契約」まで削る。長いpromptそのものを成果とせず、
代表タスクで品質・token・所要時間を比較できる形にする。

## 先に確認すること

1. 対象がCodexアプリの指示、`AGENTS.md`、skill、API promptのどれかを特定する。
2. 現在うまく動いている点と、直したい症状を分ける。
3. モデル名・reasoning設定・料金など変更されうる事実は、実装時点の公式OpenAI
   ドキュメントで確認する。他モデル固有の挙動や数値をCodexへ流用しない。

## Promptを組み立てる順序

必要な項目だけを、原則として一度ずつ書く。

1. **objective**: 達成したい結果
2. **context**: リポジトリから読み取れない背景だけ
3. **deliverables / acceptance**: 成果物と機械的に確認できる完了条件
4. **scope / constraints**: 触ってよい範囲、守る技術条件
5. **authority boundary**: 外部書き込み、破壊操作、課金、scope拡大で確認が必要な地点
6. **output**: 必須の形式・長さ・根拠。不要な挨拶や実況は要求しない

既存のシステム指示、`AGENTS.md`、skillにある規約を個別promptへ重複させない。
同じ禁止や確認要求を言い換えて反復すると、不要な停止やtoken増加を招く。

## Reasoning effortの選び方

- `low`: 定型変更、既知箇所の小修正、速度優先
- `medium`: 通常の実装・調査の出発点
- `high` / `xhigh`: 複数制約の設計、難しいデバッグなど、代表タスクで改善が測れた場合
- `max`: 最難関の品質優先タスクに限定し、`xhigh`との差を測る

常に最大値を指定しない。同じ代表タスクで現在値と1段階低い値を比較し、成功率を落とさず
tokenまたは時間が減る最小設定を採用する。モデルが対応するeffort値は実行環境で確認する。

## 削る指示

- 「必ず再確認」「徹底的に検証」を複数箇所で繰り返す指示
- 小タスクでもsubagentを必須にする指示
- 思考過程や自己修正の実況を要求する指示
- リポジトリから取得できる構成・コマンド・一般知識の再掲
- 成果物に不要な候補案、長い前置き、一般論

ただし、安全境界、受入条件、必要な根拠、失敗時の診断出力は削らない。

## 検証

変更前後を同じ代表タスクで比較する。最低限、完了条件の充足、誤変更、確認回数、
入力・出力token、所要時間を見る。短くなったことだけで改善と判定しない。

公式根拠:

- https://developers.openai.com/api/docs/guides/latest-model
- https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6
