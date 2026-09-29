# Codex Project Guidance

## 最初に読むもの

- プロジェクトの目的・技術前提は `PROJECT_CONTEXT.md` を正本とする
- 合意した変更仕様は `docs/ideas/`、安定したプロダクト・設計文書は `docs/` を正本とする
- 実装前に対象 `.steering/[YYYYMMDD]-[task]/` の `requirements.md`、`design.md`、`tasklist.md` を読む
- 実装中は `.steering/` を進捗の正本とし、完了したtaskを都度更新する

## Workflow routing

書き込みまたは高コストな処理を伴うWorkflow Skillの単独開始には、ユーザーによる `$skill-name` の明示が必要。

- `$init-project`: 人間が作成したXcodeプロジェクトを確認し、最小開発基盤を整える
- `$define-requirements`: 初期要件または追加・変更要件を対話で詰め、合意後に仕様へ保存する
- `$setup-project`: validな初期要件から6つの永続文書を作る
- `$prepare-steering`: 確定した日付付きfeature specから `.steering/` の計画を準備する
- `$implement-steering`: 指定 `.steering/` に従って実装し、進捗と検証証跡を同期する
- `$validate-implementation`: 専門agentが実装を読み取り専用で独立検証する
- `$feature-development`: 確定した永続6文書からGate Review・差分計画・実装・Build/Test・Validation・修正・再検証を完了まで管理する

個別Skillの明示はその工程だけを許可する。同等の自然文だけでは開始せず、対応する `$skill-name` の明示を求める。初期開発は `$init-project` → `$define-requirements` → `$setup-project`。永続6文書が不足する場合は `$setup-project` を案内する。

`$review-docs` は自然文の文書レビュー依頼でも使用でき、通常は **Interactive Mode** とする。Ideasや要求整理では質問・代替案による壁打ちを行う。**Gate Mode** はユーザーへ直接質問せず、独立したPASS / FAILと問題分類を親へ返す。

`$feature-development` の明示時だけ、配下の `$review-docs` Gate Mode、`$prepare-steering`、`$implement-steering`、`$validate-implementation` と修正・再検証の反復を一括して許可する。合理的に判断可能な事項や工程の移行では承認を求めず、Build/Test/Review/Validation failureは自律修正する。停止・質問の条件は [autonomy-policy.md](.agents/skills/feature-development/references/autonomy-policy.md) に従う。追加開発は要求差分を実装し、影響範囲の回帰と現在の永続6文書全体との整合を確認する。

専門Skillsは担当Workflow・agentが名前を明示して使用する。内部の `$steering` とユーザー向けの `$prepare-steering` / `$implement-steering` を区別する。

## Agent delegation

- main agentはユーザー対話、判断、Workflow管理、調査、差分分析、実装・修正、Build/Test、結果統合、完了判定を担当する
- 境界の明確な調査は `explorer`、実装は所有範囲を指定した `worker` に委任できる
- 仕様・永続文書の作成は `document_author`、steering計画の作成は `feature_planner` に委任する
- `docs/` 配下の文書レビューでは必ず `doc_reviewer` を使用する
- 実装検証では必ず `implementation_validator` を使用する
- ドキュメントレビューと実装検証の両方が必要なら、両agentを使用してmain agentが結果を統合する
- 書き込みagentには所有ファイルまたは所有ディレクトリを明示し、mainを含め同じファイルを同時編集しない
- subagentには目的、入力パス、制約、出力形式、完了条件を渡し、他者の変更を戻さないよう指示する
- subagentは結論、根拠、変更概要、検証結果、残課題を日本語で簡潔に返す
- main agentは報告だけで完了とせず、最終diffと所有範囲外の変更がないことを確認する

## Engineering constraints

- Swift + SwiftUI + native iOS + Xcodeを使用する
- `PROJECT_CONTEXT.md` の確定済みXcode・Swift設定・最低対応iOSを明示依頼なしに変更しない
- 初期iOS Appは人間がXcodeで作成する。Codexによる独自のproject/workspace生成やproject generator導入は行わない
- 外部依存・外部ツールは標準導入せず、必要性と導入範囲を合意してから追加する
- build/testは実在するproject/workspace、scheme、destination、テスト構成を確認してから実行する
- 固定カバレッジ閾値は `docs/development-guidelines.md` で合意済みの場合だけ適用する

## Document and task rules

- `docs/ideas/initial-requirements.md` は `$setup-project` のbootstrap入力にだけ使用する
- `docs/ideas/YYYYMMDD-[feature-name].md` は単独の `$prepare-steering` の標準入力とする
- `$feature-development` の正本は確定した永続6文書。日付付きconfirmed specが指定された場合は、合意済み変更だけを `document_author` が永続文書へ反映してからGate Reviewへ渡す。配下のprepareはGate通過文書と要求差分を受け取る
- 仕様は `docs/ideas/`、決定記録は `docs/decisions/`、短期計画は `.steering/` に置く
- 安定した要件・設計・用語・開発規則が変わった場合は、関連する `docs/` を更新する
- reviewとvalidationは対象ファイルを変更しない。指摘の修正は親Workflowが別の作業として担当する

## Completion

- 変更範囲に応じたビルド・診断確認、test、Simulator・実機での起動確認を実行する
- 実行できない検証は、理由と未確認範囲を報告する。実機限定項目は `Not Verified - requires physical device` としてコード実装完了と区別する
- `.steering/` を使う実装ではtasklistと検証証跡を最新にする
- `$feature-development` の完了は同Skillの完了条件に従い、Gate ReviewとValidationの判定をmain agentが独自に置き換えない
