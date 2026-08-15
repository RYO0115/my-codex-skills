# my-codex-skills

Codex用のプロジェクトスキル集です。

各ディレクトリの`SKILL.md`がスキル本体で、`agents/openai.yaml`にCodex向けの表示情報を定義しています。

## 収録スキル

- `business-planning`
- `cpp-dev-workflow`
- `git-dev-workflow`
- `market-research`
- `math-explanation`
- `python-dev-workflow`
- `subagent-delegation`
- `task-triage`

## プロジェクトへの導入

```bash
git submodule add https://github.com/RYO0115/my-codex-skills.git .agents/skills
git submodule update --init --recursive
```

Codexは`.agents/skills/<skill-name>/SKILL.md`をプロジェクトスキルとして読み込みます。
