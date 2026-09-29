---
name: review-docs
description: docs配下のMarkdownを独立agentでread-onlyレビューする。通常はInteractive Modeで仕様の壁打ちを支援し、feature-developmentからのGate Modeでは永続6文書の実装開始可否を共通基準で判定する。コード実装の検証、docs外の対象文書、文書修正には使用しない。
---

# Review Docs

必ずcustom agent `doc_reviewer`へ委任する。両モードとも、採点・重大度・PASS条件・blocking分類・出力項目の正本である[共通評価基準](references/review-criteria.md)を読む。親Workflowやagent設定に基準を複製しない。

## モード

- **Interactive Mode（既定）**: 通常の `$review-docs` 明示呼び出しと自然文の文書レビュー依頼に使用する。Ideas、要求整理中の文書でも受け付け、曖昧さ、抜け、代替案、A/B比較、将来の問題を示し、必要ならユーザーへ質問する。実装準備判定がFAILでも壁打ちは続けられる。このSkillから実装Workflowを開始しない。
- **Gate Mode**: `$feature-development` が `mode: Gate` として呼び出す場合、または永続文書の実装開始可否の判定が明示された場合に使用する。永続6文書全体を対象にし、PASS / FAILと各blocking issueの分類を返す。ユーザーへ直接質問せず、議論、仕様修正、実装を行わない。

親からのGate依頼には、mode、永続6文書のパス、対象機能、既知の変更要求と関連根拠、前回finding（再レビュー時）を含める。対象機能が狭くてもGateの文書範囲を狭めない。

## 入力契約

Interactiveでは1件指定時はそのMarkdownだけ、複数指定時は指定ファイル群だけを対象とする。勝手に追加・置換しない。指定がない場合は以下の永続6文書を対象とする。Gateでは常にこの6文書全体を対象とし、部分集合の判定を全体のGateに流用しない。

- `docs/product-requirements.md`
- `docs/functional-design.md`
- `docs/architecture.md`
- `docs/repository-structure.md`
- `docs/development-guidelines.md`
- `docs/glossary.md`

各入力を正規化し、symlink解決後の実体もリポジトリの`docs/`配下にあること、存在する通常ファイルで拡張子が`.md`であることを確認する。1件でも不正・不足ならレビューを開始しない。Interactiveでは該当パスと理由を示して再指定を求める。Gateではユーザーへ質問せず、`review_status: NOT_RUN`、不足・不正の理由を親へ返す。点数、件数、PASS / FAILを作らず`N/A（未実施）`とする。

整合性確認に必要な関連`docs/`、`.steering/`、`PROJECT_CONTEXT.md`、`AGENTS.md`は補助入力にできる。対象文書と補助入力を区別し、補助入力を読んだことをもってレビュー対象範囲を拡張しない。

## 実行と引き渡し

1. main agentがmode、対象パス、補助入力を確定し、共通評価基準を読む。
2. `doc_reviewer`へ目的、mode、対象・補助入力パス、共通評価基準のパス、read-only制約、出力形式、完了条件を渡す。レビュー担当はすべての対象を読み、根拠に沿って独立評価する。
3. `doc_reviewer`の完了を待つ。利用不能、読込不能、評価未完了の場合はmain agentが代行せず`review_status: NOT_RUN`と理由を返す。未実施をFAILやPASSと取り違えない。
4. 完了結果が共通評価基準の必須項目を満たすことを確認する。不足や計算不整合は同agentへ補正を依頼し、main agentは独自採点・判定変更をしない。
5. Interactiveでは結果と必要な質問・代替案をユーザーへ返す。部分的な対象のPASSは対象文書限定の判定であり、永続6文書全体の実装開始承認ではないと明記する。
6. Gateでは結果を親へ返す。`AUTO_FIXABLE`は親が修正して同じ6文書全体を再レビューする。`USER_DECISION_REQUIRED`は根拠と停止理由を返し、質問・停止は親の[自律判断・停止Policy](../feature-development/references/autonomy-policy.md)に委ねる。Review FAILだけを工程承認の理由にしない。単独Gateでは結果と停止理由を報告して終了する。

## 完了条件

- `doc_reviewer`が指定範囲をread-onlyで評価し、共通基準の6観点、平均、重大度別件数、判定、根拠を返している。
- 両モードとも文書、コード、設定、`.steering/`を変更していない。親による文書修正やレビュー証跡保存はこのレビュー呼び出しの外で行う。
- Gateではすべての不合格条件をblocking issueへ対応付け、各issueを分類している。親はこの結果を唯一の文書レビュー判定として使用する。
- 実装検証が必要なら既存の `$validate-implementation` を案内し、このSkillで代行しない。
