# Tasklist

## 実行規則

- taskごとに所有範囲と完了条件を確認する
- 完了時に状態と検証証跡を更新する
- 同じファイルを複数agentへ同時に割り当てない

## Tasks

- [ ] [task]
  - Status: pending
  - Owner: [main / agent]
  - Files: [paths]
  - Done when: [condition]
  - Verify: [command or check]

## 検証証跡

| 日付 | 検証 | 結果 | 備考 |
| --- | --- | --- | --- |
| - | - | 未実施 | - |

## 親Workflowの判定履歴（feature-development内で使用）

- 文書の版/hash・コードcommitと未コミット差分:
- Gate結果と対象版:
- Validation結果・Traceability: [validate-implementationの出力を記録]
- 修正原因・対象・再Build/Test/Validation結果:
- 過去Verifiedの再利用根拠とRegression結果:
- 実機限定の未検証AC・理由・手順:
- 阻害要因と再開点:

## 振り返り

- 実装完了日: [date]
- 計画との差分: [difference]
- 残課題: [issue]
