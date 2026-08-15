---
name: market-research
description: >-
  市場調査を体系的に実行するスキル。診断ヒアリング→調査計画→Codexサブエージェント
  による並列調査(マクロ環境・市場規模・競合・顧客)→統括モデルによる統合・品質ゲート
  という流れで、根拠と信頼度ラベル付きの市場調査レポートを knowledge/research/ に出力する。
  「市場調査して」「市場規模を知りたい」「競合を分析して」「参入すべきか調べて」
  といった依頼、および TAM/SAM/SOM・PEST・3C・Five Forces・SWOT・JTBD を扱うときに使う。
---

# 市場調査スキル

市場調査を「調査ごっこ」で終わらせず、根拠・前提・信頼度が追跡できるレポートに
仕上げるためのワークフロー。方法論の詳細は `references/` に機能別に分割してある。
必要になったステップで該当ファイルだけを読むこと(全部を先読みしない)。

| 機能 | 参照ファイル |
|---|---|
| 分析フレームワークと適用順序 | [references/frameworks.md](references/frameworks.md) |
| 市場規模算定(TAM/SAM/SOM) | [references/market-sizing.md](references/market-sizing.md) |
| 顧客分析(セグメント・ペルソナ・JTBD) | [references/customer-analysis.md](references/customer-analysis.md) |
| 情報源プレイブックと出典ルール | [references/data-sources.md](references/data-sources.md) |
| レポート雛形と品質ゲート | [references/report-template.md](references/report-template.md) |

## 役割分担(必須)

- **統括(このセッションのメインモデル)**: ヒアリング、調査計画の設計、サブエージェント
  へのタスク分割、結果の統合・矛盾の裁定、レポート執筆、品質ゲート判定。
- **調査担当(Codex サブエージェント)**: Web検索・情報収集・個別フレームワークの
  一次分析。利用可能な multi-agent ツールで独立した調査単位を並列化する。
  モデルは原則として親から継承し、必要な場合だけ明示的に上書きする。
- サブエージェントへのプロンプトには必ず含める: ①調査目的と対象の定義
  ②抽出してほしい具体項目(表の列まで指定) ③全ての数値に「出典URL+取得日+
  一次/二次の別」を付ける指示 ④反証・ネガティブ情報も探す指示 ⑤ファイル編集禁止
  (テキスト報告のみ)。

## ワークフロー

### Step 0 — 診断ヒアリング

いきなり調査を始めない。不足情報が結果を大きく変える場合だけ、ユーザーへ以下を確認する(既に会話で判明している
項目は聞き直さない。最大でも2回のまとめ聞きに収める):

1. 調査の目的(参入判断 / 事業計画の裏付け / 競合ウォッチ / 投資判断)
2. 対象市場の定義(地域 × 顧客層 × 製品カテゴリ)
3. 自社(または検討中の事業)の現状ステージと強み
4. 意思決定の期限と、判断に最低限必要な問い(Key Questions)

過去に同じ対象の調査があれば `knowledge/research/` 内を確認し、差分調査にする。

### Step 1 — 調査計画

Key Questions を 3〜6 個に絞り、各問いに「使うフレームワーク × 情報源 × 担当
サブエージェント」を割り当てた調査計画を短く提示する。フレームワークの適用順序は
**外→内(PEST → 3C/Five Forces → SWOT)**を守る([frameworks.md](references/frameworks.md))。

### Step 2 — 並列調査(Codex subagents)

典型的な分割(案件に応じて取捨選択、通常 2〜4 体):

- **マクロ・業界担当**: PEST 要因、業界構造(Five Forces)、規制動向
- **市場規模担当**: トップダウン+ボトムアップの両方で TAM/SAM/SOM の材料収集
  ([market-sizing.md](references/market-sizing.md) の指示テンプレを使う)
- **競合担当**: 競合 3〜7 社の機能・価格・ターゲット・メッセージング・弱点
- **顧客担当**: VOC(レビュー・フォーラム・SNS)から JTBD・ペイン・代替手段を抽出
  ([customer-analysis.md](references/customer-analysis.md))

情報源の選び方・一次/二次の扱いは [data-sources.md](references/data-sources.md) に従う。

### Step 3 — 統合(統括モデルの本業)

- サブエージェント間で数値・主張が食い違う場合は、出典の強さで裁定し、裁定理由を
  レポートに残す。三角測量できない数値は「単一ソース」と明記する。
- 全ての定量値に**「方法+前提条件+信頼度(高/中/低)」**を併記する。
- **反証セクションを必ず設ける**。参入見送り・ネガティブな結論も許容する
  (結論ありきの調査にしない)。

### Step 4 — レポート出力と品質ゲート

[report-template.md](references/report-template.md) の雛形で
`knowledge/research/<slug>/market-research-YYYY-MM-DD.md` に出力し、同ファイル末尾の
品質ゲート(曖昧さテスト・出典網羅・反証有無)を自己チェックしてから納品する。
ヒアリング結果は `knowledge/research/<slug>/context.md` に保存し、次回の再ヒアリングを省く。
書き出した/更新した各ファイルの末尾 `## 履歴` に `- <今日>: <何をしたか1行>` を
時系列で追記する(新規は `初版作成`。差分調査での更新も必ず1行残す)。
事業計画に進む場合は `business-planning` スキルがこのレポートを入力として使う。
