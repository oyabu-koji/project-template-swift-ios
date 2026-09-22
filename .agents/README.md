# `.agents/` directory

Codexがリポジトリ単位で自動検出する共有Skillsを配置します。

## 構成

```text
.agents/
└── skills/
    └── <skill-name>/
        ├── SKILL.md
        ├── agents/openai.yaml
        ├── references/    # 必要時だけ読む詳細資料
        ├── assets/        # 出力へコピーして使うテンプレート等
        └── scripts/       # 決定的な処理が必要な場合だけ
```

- `SKILL.md` は `name` と、利用条件・非対象範囲を含む `description` を持つ
- `agents/openai.yaml` は表示情報と暗黙呼び出し可否を定義する
- 詳細ガイドは `references/`、出力テンプレートは `assets/` に置く
- custom agentの役割定義は `.agents/` ではなく `.codex/agents/*.toml` に置く

## Workflow Skills

- `$init-project`
- `$define-feature`
- `$setup-project`
- `$plan-feature`
- `$implement-feature`
- `$review-docs`
- `$validate-implementation`

`$review-docs` だけは自然文による暗黙呼び出しを許可します。それ以外のWorkflow Skillは `$skill-name` で明示的に呼び出します。

## Specialist Skills

- `$prd-writing`
- `$functional-design`
- `$architecture-design`
- `$repository-structure`
- `$development-guidelines`
- `$glossary-creation`
- `$steering`

専門Skillsはすべて明示呼び出し専用です。`$setup-project` や各担当agentが、文書の依存順と対象作業に応じて名前を明示して使用します。

## Agentsとの関係

- `.codex/agents/document_author.toml`: 仕様・永続文書作成
- `.codex/agents/feature_planner.toml`: `.steering/` 計画作成
- `.codex/agents/doc_reviewer.toml`: 読み取り専用の文書レビュー
- `.codex/agents/implementation_validator.toml`: 読み取り専用の実装検証
- 組み込み `explorer`: 読み取り中心の調査
- 組み込み `worker`: 所有範囲を指定した実装

旧 `.agents/commands/`、`.agents/agents/`、`.agents/settings.json` は移行時に削除済みで、最終構成では使用しません。
