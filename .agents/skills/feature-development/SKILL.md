---
name: feature-development
description: 確定済みの永続6文書からGateレビュー、差分計画、実装、Build/Test、独立Validationと修正を完了まで統括する。ユーザーが機能開発を一括実行するため明示使用する。Ideasの壁打ち、初期文書のbootstrap、個別Skillだけの依頼には使用しない。
---

# Feature Development

既存Skillの成果物と判定を受け渡すEnd-to-End Orchestrator。レビュー採点、計画作成、実装検証の詳細は担当Skillを正本とする。

## 入力と権限

- ユーザーの `$feature-development` 明示実行を入口にする。機能名、確定した変更仕様、再開するsteeringは任意。省略時は現在の永続6文書全体の未達要求を対象にする。
- `docs/product-requirements.md`、`docs/functional-design.md`、`docs/architecture.md`、`docs/repository-structure.md`、`docs/development-guidelines.md`、`docs/glossary.md` をすべて読む。存在だけで確定・実装可能とみなさない。
- `AGENTS.md`、`PROJECT_CONTEXT.md`、[自律判断Policy](references/autonomy-policy.md)を読み、Git作業状態・既存変更を記録する。ユーザーの変更を保持する。
- 永続文書が不足する場合は不足パスと `$setup-project` を案内し、入力不足として終了する。このSkillからbootstrapしない。draftや未合意の製品判断はPolicyで扱う。
- 確定した日付付き変更仕様が明示された場合、その合意と置換対象が一意なら `document_author` に所有する永続文書を指定して反映してからGateへ渡す。明示的な変更合意を根拠として記録する。不一致だけを理由に新仕様を優先せず、要求変更が未合意なら停止条件を適用する。初期要件を直接の実装入力にしない。
- 子Skillはmainが名前とパスを明示して読み、同じ親実行の工程として使用する。架空のCLIや引数を作らない。子Skillの完了は親への返却であり、ユーザーへの工程承認ではない。

## 実行

1. 関連コード、既存テスト、Git履歴、過去のsteering/Validation証跡を調査する。[差分分析](references/incremental-development.md)に従い現状と要求差分を仮整理する。実装にはまだ着手しない。
2. [review-docs](../review-docs/SKILL.md)を **Gate Mode**、対象は永続6文書全体として実行する。確定仕様・技術制約・実現可能性の根拠を渡し、独立 `doc_reviewer` の完了を待つ。親が再採点や判定の上書きをしない。
3. FAILの `AUTO_FIXABLE` は根拠と所有ファイルを指定して `document_author` に修正させる。差分を確認し、再び6文書のGateを実行する。`USER_DECISION_REQUIRED` はPolicyに従い親が停止理由を提示する。未実施や評価者不在をPASSにしない。
4. Gate PASS後、差分分析を確定する。[prepare-steering](../prepare-steering/SKILL.md)へ「feature-development内の呼び出し」、6文書・Gate結果・差分表・調査結果・出力先を渡す。`feature_planner` が3計画文書を作り、親へ返す。日付付き仕様を捏造して入力要件を満たさない。
5. requirementsに要求差分、designに影響と実装順、tasklistに所有・回帰・検証方法があることを確認する。`Unchanged` でも未実装・未検証なら是正対象を計画し、Verified済み実装の作り直しを避ける。
6. [implement-steering](../implement-steering/SKILL.md)へ対象steeringと親の自律実行コンテキストを渡す。mainが実装・修正・Build/Testを行い、必要時だけ所有範囲を切ったworkerを使う。内部設計や失敗対応はPolicyで決定する。
7. 確認済み構成でBuild/Testと必要なRegression・起動確認を実行し、現行コードの証跡をtasklistへ同期する。環境不足、実機限定、自動テスト失敗を区別する。詳細は子SkillのXcode検証手順に従う。
8. [validate-implementation](../validate-implementation/SKILL.md)へ対象steering、永続6文書全体、差分表、現在のコードとテスト、証跡を渡す。独立 `implementation_validator` に全体のTraceabilityと今回の回帰を判定させる。実装者は判定を代行しない。
9. 未実装・自動修正可能なFailureを親へ戻し、tasklistに修正taskを記録する。実装修正 → Build → Test → Validationを反復する。仕様・安定設計の文書が変わる場合は先に根拠を確認し、所有範囲を指定した `document_author` が更新し、6文書のGateから再実行する。旧PASSや変更前の検証結果を流用しない。
10. 子Skillの評価者が返したレビュー・実装判定と証跡を照合し、最終diffと所有範囲を確認する。完了条件を満たすまで可能な作業を続ける。

## 証跡と再開

- Gate前の記録は作業メモに保持し、計画作成後にmainが対象 `tasklist.md` へ集約する。Gate対象・文書の版/hash、要求差分の比較基準、各判定とfinding、修正履歴、コードのcommitと未コミット差分、コマンド・結果の場所、Traceabilityを記録する。
- 各反復で原因・修正・再検証結果を残す。同じコマンドを根拠なく反復せず、別の原因調査・修正へ進む。失敗回数や所要時間だけでユーザー確認へ切り替えない。
- agent/ツール/環境が使えない場合は復旧可能性を調査し、依存しない実装可能な作業を終える。外部状態が変わらない限り進めないことを根拠付きで確認した場合のみ未完了として阻害要因・再開点を報告する。工程承認を求めず、完了とも報告しない。
- 再開時は記録した文書・コード・環境と現在値を比較し、影響するGate/計画/検証を再実行する。単に直近のPASSを採用しない。

## 完了と報告

次のすべてを満たす場合だけ実装完了とする。

- 現行の6文書に対するreview-docs Gate = PASS。
- validate-implementation = PASS。その判定契約によりCritical/Majorなし、実装可能な要求すべてImplemented、Build成功、自動Test成功、自動検証可能なACすべてPASS、6文書全体のTraceabilityを確認済み。
- mainの最終diff確認とtasklist・検証証跡の同期が完了している。
- 実機限定の `Not Verified - requires physical device` があれば対象AC・理由・手順を明示し、「コード実装・自動検証完了／実機確認待ち」と「全AC検証済み」を分ける。実機待ちを理由に他の実装・自動検証を残して終了しない。

途中は簡潔な進捗共有に留め、工程ごとの承認や完了報告でターンを終えない。最後に実装差分、Gate/Validation判定、Build/Test/Regression結果、証跡パス、未確認項目だけをまとめる。要求上の停止条件または外部阻害で中断した場合は未完了範囲を明記する。
