---
name: subagent-delegation
description: >-
  Codex の multi-agent ツールと Claude Code CLI を使い、独立した計画・探索・実装・
  機械作業・検証を適切なモデルへ安全かつ効率的に委譲するスキル。ユーザーが
  サブエージェント、Claude、Opus、Sonnet、Haiku、並列化、委譲、トークン節約を
  明示したとき、または適用中の規約が委譲を求めるときに使う。複雑な計画はOpus、
  仕様が固まった実装はSonnet、まとまった単純作業はHaikuを候補にし、ROI・利用可否・
  書込み競合を確認してCodex native agentと使い分ける。小タスクや重複作業には使わない。
---

# Codex / Claude サブエージェント委譲

サブエージェントは、親がローカルで進めるクリティカルパスと独立した仕事に使う。
委譲自体を目的にせず、並列化、文脈隔離、またはモデル適性で総コストを下げられる場合だけ使う。

## 委譲先とモデルを選ぶ

最初に [references/when-to-delegate.md](references/when-to-delegate.md) のROIテストを行い、
通った仕事だけを次の順で割り当てる。

| 仕事 | 第一候補 | 既定effort | 例 |
| --- | --- | --- | --- |
| 曖昧さ・影響範囲が大きい計画、設計比較、高リスクレビュー | Claude **Opus** | medium（重大時のみhigh） | 実装計画、移行設計、障害原因の仮説比較 |
| 仕様と受入条件が固まった実装・リファクタ・テスト修正 | Claude **Sonnet** | medium | 機能実装、複数ファイル改修、テストを緑にする |
| 判断不要でまとまった機械作業 | Claude **Haiku** | low | 定型置換、単純抽出、独立コマンド群、ログ整形 |
| Claudeを使えない、Codex固有ツールが必要、親との連絡を継続したい | Codex native agent | 親を継承 | 並列調査、レビュー、Codex toolが必要な作業 |

1回の短いコマンドや1〜2ファイルの小修正はHaikuへ出さず親が直接行う。Claudeのcold startと
ブリーフ作成の方が高くつくためである。迷ったらHaikuではなくSonnet、委譲自体に迷ったら
親で実行する。Opusに実装の手数を持たせず、計画をOpus、確定した計画の実装をSonnetへ分ける。

## Claude Code CLIを使う条件

- `claude --version`が成功し、利用中の認証方式で非対話実行できることを先に確認する。
- リポジトリまたは専用worktreeのルートを作業ディレクトリにし、`.claude/skills`が初期化済みか
  確認する。worktreeはリポジトリ指定の作成スクリプトで作り、skills submoduleを欠落させない。
- `--setting-sources user,project,local`を指定する。`--bare`、`--safe-mode`、
  `--disable-slash-commands`はskillとプロジェクト規約を無効化するため使わない。
- ブリーフに適用すべきClaude skillを列挙し、`.claude/skills/<skill>/SKILL.md`を最初に
  **全文読む**よう明記する。自動triggerだけに依存しない。参照ファイルもskillの指示どおり読む。
- `claude -p --model <opus|sonnet|haiku> --effort <level>`を使い、ブリーフは標準入力で渡す。
- 計画・レビューは`--permission-mode plan`または読み取り専用toolsに限定する。
- 実装は対象ディレクトリと必要toolsだけを許可する。`--dangerously-skip-permissions`を使わない。
- JSON出力が必要なら`--output-format json`を使い、`session_id`を保持する。差し戻しは
  `claude -p --resume <session_id>`で同じ文脈へ返し、cold startを繰り返さない。
- 認証エラー、利用上限、モデル利用不可、非対話permission停止が起きたら1回で打ち切る。
  Codex native agentまたは親へフォールバックし、Claudeへ委譲したと偽って報告しない。
- Claude sessionは既定1本ずつ起動する。独立性と待ち時間短縮が明確な場合だけ最大2本を並列にし、
  Opusの計画とSonnetの実装のような依存工程は順番に実行する。利用枠を先食いする多重起動をしない。

実行例:

```bash
claude -p --model opus --effort medium --permission-mode plan \
  --setting-sources user,project,local \
  --output-format json < /tmp/claude-plan-brief.md

claude -p --model sonnet --effort medium --permission-mode acceptEdits \
  --setting-sources user,project,local \
  --output-format json < /tmp/claude-implementation-brief.md
```

Haikuへコマンドを任せる場合も、許可するBashパターン、作業ディレクトリ、禁止する破壊的操作を
ブリーフへ明記する。単に「このコマンドを実行して」と丸投げしない。

## 手順

1. タスク全体を短く分解し、親が今すぐ進める作業を1つ決める。
2. 独立しており、結果を待たずに親が別作業を進められる部分だけを委譲する。
3. ROI、仕事の質、Claudeの利用可否から、Codex native / Opus / Sonnet / Haikuを選ぶ。
4. 書き込みを伴う場合は、エージェントごとに重ならないファイル範囲または専用worktreeを割り当てる。
   DB、生成済みデータ、lockfileなど共有正本を複数エージェントから書き換えない。
5. 目的、対象、制約、成果物、検証方法を[ブリーフ](references/brief-and-review.md)へ固定して渡す。
6. エージェント稼働中は、親が重複しない作業を進める。反射的に待機しない。
7. 結果が次の作業に必要になった時点でだけ待つ。Claude CLIではプロセス出力を回収する。
8. 返された変更や結論を親が短くレビューし、必要なら同じagent/sessionへ差し戻す。
9. 不要になったCodex agentへ追加作業を送らず、Claude session idや一時ブリーフを成果物へ混ぜない。

## 委譲してよい仕事

- 広域な読み取り探索を要点へ圧縮する
- 独立した複数観点のWeb調査を並列化する
- 明確な仕様と検証条件がある定型編集を、重ならないファイル群へ適用する
- 親の実装と並行して、独立した安全性・互換性チェックを行う

## 委譲しない仕事

- 数回のツール呼び出しで終わる小タスク
- 親の次の一手が結果待ちになるクリティカルパス
- 設計判断、事実の最終判定、ユーザー意図の解釈
- 親と同じファイル・同じ問いを扱う重複作業
- ユーザーや上位指示から委譲の許可がない状況

## 複合タスクの標準構成

重い実装では、必要な段だけを使う。全モデルを儀式的に起動しない。

1. **Opus（任意）**: 不確実性が高い場合だけ、読取専用で計画・リスク・受入条件を作る。
2. **Sonnet**: 親が承認した計画のうち、境界が明確な実装とテストを担当する。
3. **Haiku（任意）**: 実装から独立した大量の単純抽出・整形・コマンド群だけを担当する。
4. **親Codex**: ユーザー意図、最終設計判断、diffレビュー、統合検証、外部状態変更を保持する。

同じ問いをOpusとCodexへ同時に解かせる「保険の二重実行」はしない。異論が必要な高リスク案件では、
最初から「独立レビュー」という別成果物に切り分ける。

詳細な判断基準は [references/when-to-delegate.md](references/when-to-delegate.md)、
指示とレビューの雛形は [references/brief-and-review.md](references/brief-and-review.md) を読む。
