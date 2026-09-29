# Swift iOS AI Development Template

## 1. このテンプレートについて

Swift / SwiftUIによるネイティブiOSアプリをCodexと開発するためのAI開発環境です。Ideasや要求整理では人間とAIで仕様を育て、6つの永続文書が確定した後は、仕様レビューから実装、Build/Test、独立検証、修正・再検証までを一括して進められます。既存の工程別実行も利用できます。

このリポジトリ自体はiOSアプリではありません。アプリ固有の要件やコード、Xcodeプロジェクトは含まず、実際のiOSアプリのリポジトリで使うためのテンプレートです。Xcodeプロジェクトの生成機能もありません。

## 2. 技術前提

| 項目 | 前提 |
| --- | --- |
| 言語 | Swift |
| UI | SwiftUI |
| プラットフォーム | ネイティブiOS |
| 開発・ビルド | Xcode |

Xcodeのバージョン、Swiftコンパイラ・言語モード、最低対応iOS、対象端末は、このテンプレートでは固定していません。実際のアプリで確定し、[PROJECT_CONTEXT.md](PROJECT_CONTEXT.md)へ記録します。技術上の制約も同ファイルを正本とします。

## 3. 最初にどう使うか

1. このテンプレートのAI開発環境を、開発対象のアプリリポジトリに用意する。
2. 人間がXcodeで「iOS App / SwiftUI / Swift」を選び、そのリポジトリにプロジェクトを作成する。
3. Codexで`$init-project`を実行し、既存プロジェクトと最小開発基盤を確認する。プロジェクトがなければ作成を案内して停止する。
4. `$define-requirements`で初期アプリ要件を対話で具体化し、`docs/ideas/initial-requirements.md`に保存する。仕様の壁打ちには`$review-docs`も使える。
5. 初期要件が固まったら、`$setup-project`で6つの永続文書を作成・レビューする。
6. 永続6文書を確定したら、`$feature-development`で実装完了まで進める。必要な工程だけを個別に実行することもできる。

## 4. Workflowと責務

| Workflow | 役割 |
| --- | --- |
| `$init-project` | 人間が作成したXcodeプロジェクトの構成と最小開発基盤を確認する。プロジェクトは生成しない。 |
| `$define-requirements` | 初期要件・変更要件を質問と回答で具体化し、合意した仕様を`docs/ideas/`へ保存する。 |
| `$setup-project` | validな初期要件から永続6文書を初期作成・レビューする。 |
| `$review-docs` | `doc_reviewer`が読み取り専用で文書を評価する。通常はInteractive、親Workflow内ではGateを使う。 |
| `$feature-development` | 永続6文書を正本にGate Review、差分計画、実装、Build/Test、Validation、修正・再検証を管理する。 |
| `$prepare-steering` | 単独実行では確定した日付付きspecから`requirements.md`、`design.md`、`tasklist.md`を準備する。親Workflow内ではGate通過文書と要求差分を受け取る。 |
| `$implement-steering` | 指定steeringに従い実装・テスト・進捗更新・検証証跡の記録を行う。 |
| `$validate-implementation` | `implementation_validator`が実装と要求・Acceptance Criteriaの対応を読み取り専用で独立検証する。 |

既存の実装検証Skill名は`validate-implementation`です。`implement-validation`という別Skillは作りません。詳細な入力・完了条件は各`.agents/skills/<workflow>/SKILL.md`を参照してください。

## 5. 仕様を育てる工程と自律実装

```text
Ideas ⇄ review-docs Interactive ⇄ ユーザーとの壁打ち
    ↓ define-requirements / setup-project等
確定した6つの永続文書
    ↓ $feature-development
review-docs Gate → 差分分析 → prepare-steering
    → implement-steering → Build/Test → validate-implementation
    → 必要な修正・再Build/Test・再Validation → 完了
```

Interactive Modeでは未完成のIdeasも扱い、曖昧さ、抜け、代替案、A/B案を示し、必要な質問をします。通常の`$review-docs`や自然文のレビュー依頼はこのモードです。

Gate Modeでは質問せず、6観点の採点とPASS / FAILを親Workflowへ返します。PASSには平均4.5 / 5.0以上、各項目4.0以上、Critical・Majorが0件であることに加え、要求矛盾、製品仕様を推測しないと決められない事項、検証不能なAcceptance Criteria、P0要求・禁止動作の重大な定義不足がないことが必要です。Minor / Suggestionを残せるのは製品動作・要求・Acceptance Criteriaに影響しない場合だけです。評価基準の正本は[review-docs](.agents/skills/review-docs/SKILL.md)です。FAILの問題は`AUTO_FIXABLE`と`USER_DECISION_REQUIRED`に分類し、自動修正できる問題は親が修正して再レビューします。

`$feature-development`の明示は配下工程と修正・再検証までを許可します。工程ごとの承認や、合理的に判断できる内部設計への質問は行いません。製品動作を変える未解消の仕様矛盾、要求変更、不可逆な操作などの停止条件は[自律判断Policy](.agents/skills/feature-development/references/autonomy-policy.md)にまとめています。通常のBuild/Test/Review/Validation failureは修正・再検証へ戻ります。実行環境のsandbox・approval設定は引き続き適用されます。

個別の`$prepare-steering`、`$implement-steering`、`$validate-implementation`を指定した場合は、その工程だけで終了します。次工程の案内は実行許可を意味しません。

## 6. 追加機能と差分実装

`$define-requirements`で合意した変更は日付付きのconfirmed specへ保存します。`$feature-development`にそのspecを渡すと、`document_author`が合意済みの変更だけを永続6文書へ反映し、更新後の文書をGate Reviewへ渡します。未確定のIdeasを実装判断の正本にはしません。

現在の永続文書、コード、Git履歴、既存Validation結果、今回の変更を比較し、要求を`Unchanged` / `Added` / `Modified` / `Removed`に分類します。基本の実装対象は`Added`と`Modified`で、`Removed`は削除意図と影響を確認して対応します。Verified済みの`Unchanged`を理由なく再実装せず、変更の影響を受ける既存要求は回帰確認します。初回の未実装要求や検証不足も見落とさず、最終的に現在の永続6文書全体との対応を検証します。

Validationでは要求・Acceptance Criteriaごとに`Implemented`、`Verified`、`Not Verified`、`Not Implemented`、`Not Applicable`と根拠を追跡します。実機だけで確認できる項目は`Not Verified - requires physical device`と明示し、コード実装完了と検証の残件を区別します。他の実装可能な要求や自動検証を残したまま完了扱いにはしません。

## 7. 生成・管理される主な文書

| 保存先 | 役割 |
| --- | --- |
| `docs/ideas/` | 初期要件`initial-requirements.md`と、日付付きの追加・変更仕様`YYYYMMDD-[feature-name].md` |
| `docs/` | 安定したプロダクト・設計・開発規則の永続文書 |
| `.steering/[YYYYMMDD]-[task]/` | 実装単位の要求、設計、task進捗、要求差分、Traceability、検証証跡の正本 |

`$setup-project`が初期要件から作成する永続文書は次の6つです。

- `docs/product-requirements.md`
- `docs/functional-design.md`
- `docs/architecture.md`
- `docs/repository-structure.md`
- `docs/development-guidelines.md`
- `docs/glossary.md`

各steeringでは`requirements.md`、`design.md`、`tasklist.md`を使います。これらの文書は対象アプリで工程を進めたときに作成されます。初期要件がvalidでも永続6文書が不足する場合は、追加仕様や自律実装へ進まず`$setup-project`を案内します。

## 8. 呼び出し例

`$review-docs`以外のWorkflow Skillは`$skill-name`で明示します。パスは実在する対象へ置き換えてください。

```text
$define-requirements 家族で動画メモを使うアプリの初期要件を詰めたい
$setup-project
$review-docs docs/architecture.md
```

永続6文書が確定した後の一括実行:

```text
$feature-development
$feature-development docs/ideas/20260922-video-library.md
```

引数を省略すると現在の永続6文書全体を対象に未達要求を特定します。日付付きspecの指定は、既存の永続6文書に対する合意済み変更の引き渡しに使います。どちらも実装済み・Verified済みの範囲を考慮します。

工程別に実行する場合:

```text
$prepare-steering docs/ideas/20260922-video-library.md
$implement-steering .steering/20260922-video-library/
$validate-implementation .steering/20260922-video-library/
```

## 9. 内部構成と設定

- `.agents/skills/`：ユーザー向けWorkflowと専門Skill。概要は[.agents/README.md](.agents/README.md)を参照。
- `.codex/agents/`：既存の`document_author`、`feature_planner`、`doc_reviewer`、`implementation_validator`。レビュー・検証担当は対象ファイルを変更せず、main agentへ結果を返す。
- [.codex/config.toml](.codex/config.toml)：`[agents]`の`enabled = true`と`max_concurrent_threads_per_session = 4`を維持。approval・sandboxはここで上書きせず、実行環境の設定を継承する。
- [AGENTS.md](AGENTS.md)：Workflow routing、委任、Repository Policy。
- [PROJECT_CONTEXT.md](PROJECT_CONTEXT.md)：目的と技術前提の正本。

Codex 0.158.0では`.codex/agents/*.toml`からcustom agentを自動検出し、各定義に`name`、`description`、`developer_instructions`を記述します。構成の根拠は[公式subagentsガイド](https://learn.chatgpt.com/docs/agent-configuration/subagents)と[公式config reference](https://learn.chatgpt.com/docs/config-file/config-reference)です。
