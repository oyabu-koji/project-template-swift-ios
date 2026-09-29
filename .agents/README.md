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

| Workflow | 役割 |
| --- | --- |
| `$init-project` | 人間が作成したXcodeプロジェクトの最小開発基盤を整える |
| `$define-requirements` | 質問と回答で初期要件・追加要件を詰め、合意した仕様をdocs/ideasへ保存する |
| `$setup-project` | validな初期要件から永続6文書を初期作成・レビューする |
| `$review-docs` | doc_reviewerがInteractive / Gateの共通基準で文書を読み取り専用レビューする |
| `$feature-development` | 確定永続6文書からGate・差分計画・実装・Build/Test・Validation・修正・再検証を管理する親Workflow |
| `$prepare-steering` | 単独では日付付きconfirmed spec、親配下ではGate通過文書と要求差分からrequirements / design / tasklistを準備する |
| `$implement-steering` | 指定steeringに従い実装・テスト・進捗更新・検証証跡の記録を行う |
| `$validate-implementation` | implementation_validatorが実装・要求・Acceptance Criteriaの対応を変更せず独立検証する |

`$review-docs` は自然文のレビュー依頼でも使用でき、通常はInteractive Modeです。Ideasや要求整理では質問・代替案を扱います。Gate Modeは質問せず、PASS / FAIL、評価点、blocking issueの`AUTO_FIXABLE` / `USER_DECISION_REQUIRED`分類を親へ返します。採点と判定の正本は[review-docs/SKILL.md](skills/review-docs/SKILL.md)から参照する共通基準です。

それ以外のWorkflow Skillは `$skill-name` で明示します。個別Skillを呼び出した場合はその工程だけを実行し、次工程は案内して終了します。`$feature-development` を明示した場合だけ、配下のreview Gate / prepare / implement / validateと必要な修正・再検証を続けて実行します。

新規アプリは `$init-project` → `$define-requirements` → `$setup-project`。永続6文書が確定したら `$feature-development` を使用できます。従来の `$prepare-steering` → `$implement-steering` → `$validate-implementation` という個別実行も維持します。

`$define-requirements` は初期要件を詰める段階では `docs/ideas/initial-requirements.md` を扱い、validでも永続6文書が不足する中間状態では `$setup-project` を案内します。セットアップ済みアプリの合意した変更は `docs/ideas/YYYYMMDD-[feature-name].md` へ保存します。このSkill自体は永続文書やコードを更新しません。

`$feature-development` は確定永続6文書を入力正本とし、引数なしなら全体の未達要求を調べます。日付付きconfirmed specを渡した場合は、`document_author` が合意済み変更だけを永続文書へ反映してからGate Reviewを行います。差分分類、回帰、Traceability、完了条件は[feature-development/SKILL.md](skills/feature-development/SKILL.md)、判断の優先順位と質問が必要な停止条件は[autonomy-policy.md](skills/feature-development/references/autonomy-policy.md)を参照します。Build/Test/Review/Validation failureは親が修正・再検証へ戻します。

呼び出し例（仕様・steeringのパスは実在する対象へ置き換えてください）:

```text
$review-docs docs/ideas/20260922-video-library.md
$define-requirements 録画した動画を端末内で整理する機能の要件を詰めたい
$feature-development
$feature-development docs/ideas/20260922-video-library.md
```

工程別に使う場合:

```text
$prepare-steering docs/ideas/20260922-video-library.md
$implement-steering .steering/20260922-video-library/
$validate-implementation .steering/20260922-video-library/
```

## Specialist Skills

- `$prd-writing`
- `$functional-design`
- `$architecture-design`
- `$repository-structure`
- `$development-guidelines`
- `$glossary-creation`
- `$steering`

専門Skillsは担当Workflow・agentが名前を明示して使用します。`$setup-project` や `document_author` が対象文書と依存順に応じたSkillを選びます。`$steering` は3文書の構造・進捗・証跡を維持する内部専門Skillで、ユーザー向けの `$prepare-steering` / `$implement-steering` とは別物です。

## Agentsとの関係

- main agent: Workflow管理、コード調査、差分分析、実装・修正、Build/Test、証跡管理、最終diff確認と結果統合
- `.codex/agents/document_author.toml`: 仕様・永続文書作成と、親Workflowが委任する自動修正
- `.codex/agents/feature_planner.toml`: `.steering/` 計画作成
- `.codex/agents/doc_reviewer.toml`: 読み取り専用の文書レビュー。Interactive / Gate両モードを独立評価
- `.codex/agents/implementation_validator.toml`: 読み取り専用の実装検証と要求・Acceptance CriteriaのTraceability判定
- 組み込み `explorer`: 読み取り中心の調査
- 組み込み `worker`: 所有範囲を指定した実装

既存のagent名と `validate-implementation` Skill名を再利用します。レビュー・検証担当は指摘を返し、main agentが修正を管理します。同じファイルをmainとsubagentが同時に編集せず、書き込みを委任する場合は所有範囲を指定します。

Codex 0.158.0は`.codex/agents/*.toml`を自動検出します。各ファイルに`name` / `description` / `developer_instructions`を持たせ、[.codex/config.toml](../.codex/config.toml)では既存の有効化・同時実行数を維持します。approval / sandboxは同ファイルで上書きしません。詳細は[公式subagentsガイド](https://learn.chatgpt.com/docs/agent-configuration/subagents)と[公式config reference](https://learn.chatgpt.com/docs/config-file/config-reference)を参照してください。

旧 `.agents/commands/`、`.agents/agents/`、`.agents/settings.json` は移行時に削除済みで、使用しません。
