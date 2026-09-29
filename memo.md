このリポジトリの既存AI開発環境を調査し、現在手動で1工程ずつ進めている機能開発Workflowを、原則としてユーザー確認なしで実装完了まで進められるEnd-to-End Workflowへ拡張してください。

今回の目的は、現在使っている「IdeasをAIと壁打ちしながら育てる工程」は残しつつ、6つの永続ドキュメントが確定した後は、仕様レビュー → 実装準備 → 実装 → Build/Test → Validation → 修正 → 再検証までCodexが自律的に完遂する構成にすることです。

今回はFrameRelayアプリ本体の新機能実装ではありません。
AI開発Workflow、Skill、AGENTS.md、Codex Agent設定の改修のみを行ってください。

---

# 0. 最初に既存環境を調査する

変更前に以下をすべて確認してください。

- `AGENTS.md`
- `PROJECT_CONTEXT.md`
- `.agents/skills/`
- `.codex/`
- `.codex/config.toml`
- 既存のAgent設定
- 既存Skill間の参照関係

特に以下の既存Skillが存在するか確認してください。

- `define-requirements`
- `review-docs`
- `prepare-steering`
- `implement-steering`
- `implement-validation`

名前や役割が実際のリポジトリでは異なる場合は、実在するものを優先してください。

既存Skillを削除したり、同じ役割のSkillを重複作成したりしないでください。

---

# 1. 開発工程の基本方針

開発Workflowを以下の2領域に分けます。

## A. 人間とAIで仕様を育てる領域

ここは今後も残します。

```text
Ideas
↓
review-docs Interactive Mode
↓
ユーザーとの壁打ち
↓
仕様修正
↓
define-requirements等
↓
6つの永続ドキュメント確定
```

この工程ではユーザーへの質問やA/B案の提示を許可します。

## B. 永続ドキュメント確定後の自律実装領域

```text
6つの永続ドキュメント
↓
feature-development
↓
review-docs Gate Mode
↓
差分分析
↓
実装計画
↓
実装
↓
Build / Test
↓
implement-validation
↓
修正
↓
再Build / Test / Validation
↓
完了
```

この領域では、原則として工程ごとのユーザー確認を行わず、Codexが自律的に実装完了まで進めます。

---

# 2. review-docsを2モード化する

既存の `.agents/skills/review-docs/SKILL.md` を複製せず、1つのSkillの中に以下の2モードを持たせてください。

レビュー基準そのものはSingle Source of Truthとして共通化してください。

## Interactive Mode

用途:
- Ideas段階
- 要求整理中
- ユーザーとの仕様壁打ち

目的:
- 曖昧さを発見する
- 抜けを発見する
- 代替案を提案する
- A/B案を比較する
- 将来問題になりそうな点を指摘する
- 必要ならユーザーへ質問する

Interactive Modeでは、文書がまだ実装可能状態でなくても構いません。

ユーザーが通常 `$review-docs` を明示的に実行した場合は、原則Interactive Modeとして扱ってください。

---

## Gate Mode

用途:
- `$feature-development` から呼び出された場合
- 永続ドキュメントが自律実装可能か判定する場合

目的:
- ユーザーとの壁打ちは行わない
- 実装開始可能か客観的に判定する
- PASS / FAILを返す
- 問題を自動修正可能か、人間判断が必要か分類する

Gate Modeではユーザーへ直接質問しないでください。

各blocking issueを以下に分類してください。

- `AUTO_FIXABLE`
- `USER_DECISION_REQUIRED`

`AUTO_FIXABLE` の場合は親Workflowが修正して再レビューします。

`USER_DECISION_REQUIRED` の場合のみ、親Workflowへ停止理由を返します。

---

# 3. review-docsの5点評価

Interactive / Gate共通の評価基準として、以下の6観点を各5点満点で評価してください。

1. 完全性
2. 一貫性
3. 明確性
4. 検証可能性
5. 実現可能性
6. スコープ明確性

必要なら0.5点刻みを使用してください。

レビュー結果には必ず以下を含めます。

- 各観点の点数
- 総合平均点
- Critical件数
- Major件数
- Minor件数
- Suggestion件数
- 最終判定 PASS / FAIL

Gate ModeのPASS条件は以下すべてを満たすこととします。

- 総合平均 4.5 / 5.0以上
- 各評価項目 4.0 / 5.0以上
- Critical = 0
- Major = 0
- 未解決の要求矛盾 = 0
- 実装時に製品仕様を推測しないと決められない事項 = 0
- 検証不能なAcceptance Criteria = 0
- P0要求または禁止動作の重大な定義不足 = 0

平均点が高くてもCriticalまたはMajorが存在する場合は必ずFAILとしてください。

Minor / Suggestionは、製品動作・要求・Acceptance Criteriaに影響しないもののみPASS時に残存を許可します。

親Workflow側で独自に再採点せず、`review-docs Gate Mode` のPASS / FAILを唯一のレビュー判定として使用してください。

---

# 4. feature-development親Skillを追加する

以下を新規作成してください。

`.agents/skills/feature-development/SKILL.md`

このSkillはEnd-to-End Orchestratorです。

詳細ロジックを重複して持たず、既存Skillを呼び出して全体工程を管理してください。

想定Workflow:

1. 対象機能の6つの永続ドキュメントを読む
2. AGENTS.md / PROJECT_CONTEXT.mdを読む
3. 関連コードと既存テストを調査する
4. `review-docs Gate Mode` を実行する
5. FAILの場合、自動修正可能な問題を修正
6. Gate Reviewを再実行
7. PASS後、既存の準備工程を実行
8. 実装対象の差分を分析
9. 実装順序をCodex自身で決定
10. 実装
11. Build / Test
12. `implement-validation`
13. 未実装・失敗項目を修正
14. 再Build / Test
15. 再Validation
16. 完了条件を満たすまで繰り返す
17. 最終結果のみユーザーへ報告

---

# 5. Incremental Feature Developmentを実装する

既にアプリの一部または全部が実装済みの場合、6つの永続ドキュメントを毎回ゼロから再実装してはいけません。

まず以下を比較してください。

- 現在の6つの永続ドキュメント
- 現在のコード
- Git上の変更履歴
- 既存Validation結果
- 今回追加または変更された要求

要求を以下に分類してください。

- `Unchanged`
- `Added`
- `Modified`
- `Removed`

原則として実装対象は `Added` / `Modified` としてください。

`Removed` は影響を確認し、既存機能を削除すべき要求である場合のみ対応してください。

`Unchanged` かつ既にVerified済みの要求は理由なく再実装しないでください。

ただし、Added / Modifiedの影響を受ける既存要求についてはRegression Test / Validationを行ってください。

追加機能開発でも、最終的には現在の6つの永続ドキュメント全体と実装が整合していることを確認してください。

---

# 6. 自律判断Policy

永続ドキュメント確定後の実装では、判断可能な事項についてユーザーへ質問しないでください。

判断が必要な場合は以下の優先順位で決定してください。

1. 6つの永続ドキュメント
2. P0要求
3. 禁止動作
4. Acceptance Criteria
5. AGENTS.md
6. PROJECT_CONTEXT.md
7. 既存Architecture
8. 既存コードのConvention
9. 後方互換性を保てる選択
10. 変更範囲が小さい選択
11. 将来変更しやすい可逆的な選択

この順序で合理的に決定できる場合、ユーザー確認は不要です。

以下についてはCodex自身で決定してください。

- 実装順序
- ファイル分割
- クラス分割
- 関数分割
- 内部API
- 内部状態管理
- テスト構成
- 軽微なリファクタリング
- Build/Test failureへの対応
- Review指摘への対応
- Validation failureへの対応

---

# 7. ユーザー確認が必要な停止条件

以下の場合のみユーザーへ確認してください。

- Source of Truth同士が矛盾し、選択によって製品動作が変わる
- 明示された要求そのものの追加・削除・変更が必要
- P0要求または禁止動作を満たせない
- 技術的に要求が成立せず、代替仕様を決める必要がある
- 大規模なArchitecture変更が必要
- データ消失等の不可逆な操作が必要
- Security / Privacy上の重大な製品判断が必要
- 外部サービス、本番環境、課金等へ重大な影響がある

以下の場合はユーザーへ確認しないでください。

- Build失敗
- Test失敗
- Lint失敗
- Review FAIL
- Validation FAIL
- 実装上の内部設計判断
- 変更順序
- 軽微な文書修正
- 既存要求から一意に判断できる不足事項

「次へ進んでよいですか」
「この方針で実装してよいですか」
「この修正を行いますか」

のような工程承認は求めないでください。

---

# 8. implement-validationを確認・強化する

既存の `implement-validation` Skillを確認してください。

6つの永続ドキュメントの各要求・Acceptance Criteriaについて、最低限以下の状態を追跡できるようにしてください。

- Implemented
- Verified
- Not Verified
- Not Implemented
- Not Applicable

Validationで `Not Implemented` または自動検証可能なFailureが見つかった場合は、Main Codexへ戻して修正してください。

その後、

```text
修正
↓
Build
↓
Test
↓
Validation
```

を再実行してください。

停止条件に該当しない限り、ユーザーへ確認しないでください。

実機iPhoneでしか確認できないAcceptance Criteriaは、

`Not Verified - requires physical device`

として明示してください。

実機確認できないことを理由に、実装可能な他の要求を残したままWorkflowを終了しないでください。

---

# 9. 完了条件

`feature-development` は以下すべてを満たした場合のみ完了としてください。

- review-docs Gate Mode = PASS
- Critical = 0
- Major = 0
- Build成功
- 自動Test成功
- 実装可能な要求がすべてImplemented
- 自動検証可能なAcceptance CriteriaがすべてPASS
- implement-validation PASS
- 6つの永続ドキュメントと実装のTraceability確認済み
- 実機でしか確認できない項目が明示済み

`Not Verified - requires physical device` が残っていても、コード実装そのものが完了している場合は、その事実を明確に区別してください。

---

# 10. AGENTS.mdを更新する

既存の `AGENTS.md` を確認し、重複を避けて短いRepository Policyを追加してください。

詳細WorkflowをAGENTS.mdにコピーしないでください。

以下の趣旨だけを書いてください。

- Ideasや仕様の壁打ちでは `$review-docs` Interactive Modeを使用する
- ユーザーが個別Skillを明示指定した場合はそのSkillのみ実行する
- 6つの永続ドキュメント確定後、機能を実装完了まで進める場合は `$feature-development` を使用する
- feature-development中は工程ごとのユーザー承認を求めない
- 合理的に判断可能な事項では質問しない
- 停止条件に該当する場合のみ質問する
- Build/Test/Review/Validation failureは自律修正・再検証する
- 追加機能では差分実装とRegression確認を行う

---

# 11. Subagent構成

ReviewとValidationには独立Subagentを使用できる構成にしてください。

## Main Codex

担当:
- Workflow管理
- コード調査
- 差分分析
- 実装
- 修正
- Build/Test

## doc-reviewer

担当:
- review-docsの評価
- Interactive / Gate両モードに対応
- Gate Modeでは実装を行わない
- 独立した視点で仕様品質を評価
- 6観点5点評価
- PASS / FAIL判定

## validator

担当:
- 実装と永続ドキュメントの照合
- Acceptance Criteriaの確認
- 未実装・未検証項目の抽出
- 原則としてコードを直接修正せずMain Codexへ結果を返す

既存Agent設定がある場合は再利用してください。

必要であれば、

`.codex/agents/doc-reviewer.toml`
`.codex/agents/validator.toml`

等を作成してください。

最新のCodex設定形式に従い、`.codex/config.toml` から正しく登録してください。

既存 `.codex/config.toml` は破壊せずマージしてください。

未知の設定キーを推測で追加しないでください。

---

# 12. Codex実行権限

既存 `.codex/config.toml` のapproval / sandbox設定を確認してください。

目的は、通常のリポジトリ内編集・Build・Test・Validationで毎回ユーザー承認を求めないことです。

ただし既存の安全設定を無条件に弱めないでください。

変更が必要と判断した場合は、

- 現在値
- 変更案
- 変更理由
- 影響

を最終報告に記載してください。

---

# 13. 既存責務を崩さない

以下を守ってください。

- 既存Skillを削除しない
- 同じSkillを複製しない
- review-docsのレビュー基準をSingle Source of Truthにする
- review-docs Interactive / Gateで共通評価基準を使用する
- implement-validationを実装検証判定のSingle Source of Truthにする
- feature-developmentはOrchestratorに徹する
- AGENTS.mdはRepository PolicyとSkill routingに留める
- Main CodexとSubagentが同じコードを同時編集しない
- Review / Validation Subagentは独立評価を優先する

---

# 14. 変更後の検証

改修後、以下を必ず実施してください。

1. 作成・変更した全ファイルを再読込
2. SkillのYAML front matter確認
3. Skill名・参照先が実在するか確認
4. review-docs Interactive / Gateの責務が混ざっていないか確認
5. Gate ModeのPASS条件が明確か確認
6. feature-developmentの停止条件が明確か確認
7. Incremental Feature Developmentが定義されているか確認
8. AGENTS.mdとSkillに矛盾がないか確認
9. TOML構文を確認
10. Codex設定キーが有効か確認
11. Subagent登録を確認
12. 既存Workflowを壊していないか確認
13. `git diff` を確認

可能であれば、新しいSkillとAgentがCodexから認識できることも確認してください。

問題があれば途中確認せず修正し、再検証してください。

---

# 15. 最終報告

作業完了後、以下を簡潔に報告してください。

- 作成したファイル
- 修正したファイル
- review-docs Interactive Modeの動作
- review-docs Gate Modeの動作
- Gate ReviewのPASS条件
- feature-developmentのWorkflow
- Incremental Feature Developmentの動作
- Subagent構成
- ユーザー確認が発生する停止条件
- `.codex/config.toml` の変更内容
- AGENTS.mdの変更内容
- 検証結果
- 残った注意事項

このAI開発Workflow構築作業自体についても、上記停止条件に該当しない限り途中でユーザー確認を求めず、完成まで進めてください。