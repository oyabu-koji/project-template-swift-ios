---
name: validate-implementation
description: 指定されたsteering、6つの永続文書と実装・テストの整合性を独立agentがread-onlyで検証する。単独の明示呼び出し、またはfeature-developmentの内部工程で使用する。修正や仕様文書だけのレビューは行わない。
---

# Validate Implementation

実装を変更せず、必ずcustom agent `implementation_validator` を使って判定する。状態、Traceability、証跡、PASS / FAIL / BLOCKEDの定義は [validation-contract.md](references/validation-contract.md) を正本とし、mainとvalidatorは検証前に読む。既存のこのSkillを使い、`implement-validation` 等の同役割Skillを追加しない。

## 呼び出しと入力契約

- 単独: ユーザーが `$validate-implementation` と対象 `.steering/[YYYYMMDD]-[task]/` を明示する。このSkillだけを実行し、判定の報告で終了する。
- 親Workflow内: 明示された `$feature-development` が同じsteeringと契約上の入力を渡す。工程ごとの再承認なしで検証し、判定と修正候補を親へ返す。
- mainは `requirements.md`、`design.md`、`tasklist.md`、6永続文書、今回の差分分類、現在コード・テスト、過去および現在の検証証跡を渡す。入力の詳細と欠落時の扱いは判定契約に従う。
- 対象未指定、不存在、`.steering/` 外、3文書不足、またはvalidator利用不可は `BLOCKED`。mainだけで判定を代行しない。未実装taskがあっても、確認可能な範囲は検証して `Not Implemented` を報告する。

## 実行手順

1. mainが入力と検証対象の版を揃え、validatorへ対象パス、契約、制約を渡す。validatorは内部Skill `$steering` をread-onlyで明示使用する。
2. validatorが6永続文書全体の要求・Acceptance Criteria（AC）をコード・テスト・証跡へ対応付ける。steeringの範囲、設計、tasklistの実態と、今回の差分が既存要求へ与える回帰を確認する。
3. build/test等の追加証跡が必要なら、validatorが必要なコマンド、確認済み構成、書き込み先をmainへ返す。mainが [Xcode検証手順](../development-guidelines/references/process.md) と既存のsandbox/approvalに従って実行し、結果をvalidatorへ戻す。構成を推測せず、validatorはbuild/testを実行しない。
4. validatorが判定契約に従い結論、全体Traceability、finding、実機限定未検証、修正候補と再検証範囲を返す。mainは証跡との食い違いや不足があればvalidatorへ再評価を依頼し、判定を独自に変更しない。
5. 単独呼び出しはその結果を報告して終了する。必要なら同じsteeringを対象とする `$implement-steering` を案内するが、修正、tasklist更新、別Workflowを開始しない。
6. 親Workflow内では判定を親へ返す。親が修正し、Build → Test → このSkillによる再Validationを行う。修正は検証passの終了後に行い、validatorと同じファイルを同時編集しない。失敗だけを理由に確認を求めず、質問の要否は親の [自律判断Policy](../feature-development/references/autonomy-policy.md) に従う。

## 完了条件

- validatorの判定と、実装状態・検証状態を区別した全体Traceabilityが返っている。
- 必須の自動検証未実行、0件実行、skipを合格扱いにしていない。
- 実機限定未検証が残る場合、実装範囲の `PASS` と全ACの `Verified` を区別している。
- 検証中にソース、テスト、設定、docs、`.steering/` を変更していない。Skillの報告完了と実装の `PASS` は別である。
