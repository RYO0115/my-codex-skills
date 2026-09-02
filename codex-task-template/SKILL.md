---
name: codex-task-template
description: >-
  Codexへ渡す作業指示をtask.yamlとして作成・整理するスキル。goal、context、deliverables、
  scope、constraints、do_not、acceptance、process、outputを分離し、実装の完了条件と権限境界を
  明確にする。「Codex用のtask.yamlを作って」「実装指示をYAMLにして」「他エージェント用の
  task YAMLをCodex向けに移行して」といった依頼で使う。内部goal作成や通常の会話だけには使わない。
---

# Codex task template

[assets/codex-task.yaml](assets/codex-task.yaml)を使う。

## 作成手順

1. テンプレートを依頼先workspaceの `task.yaml` へコピーする。
2. `goal`、`context`、`deliverables` は必ず具体化する。
3. 任意項目は必要なものだけ残す。空の見出しや例文を成果物に残さない。
4. `acceptance` は実行可能なコマンドや観測可能な結果にする。
5. 外部書き込み、破壊操作、課金、scope拡大など、承認が必要な地点を `do_not` または
   `process.confirm_before` に明記する。
6. プロジェクトの `AGENTS.md`、README、既存のテストラッパーを確認し、実在するコマンドへ
   置き換える。他エージェント固有のモデル名・ツール名・命令は残さない。

`task.yaml` は作業指示の可搬形式であり、Codex内部のgoal状態とは別物。ユーザーがgoal作成を
明示していない限り、YAMLを作っただけで内部goalを開始しない。

使い捨ての指示ファイルは、プロジェクト規約で要求されない限りコミットしない。PR本文への
添付規約がある場合はそれに従う。
