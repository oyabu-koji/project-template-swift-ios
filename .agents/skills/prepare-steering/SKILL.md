---
name: prepare-steering
description: 確定した日付付き機能仕様、またはfeature-developmentが渡すGate通過済み永続6文書と要求差分から.steeringのrequirements、design、tasklistを準備する。実装前の調査と計画に使用する。initial-requirementsからの直接計画、コード実装には使用しない。
---

# Prepare Steering

確定した正本を実装可能な計画へ変換する。単独実行は設計完了で終了し、feature-development内では成果物を親へ返す。

## 入力契約

### 単独実行

- `docs/ideas/YYYYMMDD-[feature-name].md` を1件、明示入力として受け取る。引数なし、不存在、docs/ideas外、Markdown以外、initial-requirementsでは変更せず停止する。
- 正規化した実体がdocs/ideas配下にあること、`Status: confirmed` または会話上の同内容への合意を確認する。draft/合意不明なら `$define-requirements` を案内する。この工程で仕様を確定・更新しない。
- 計画に必要な製品判断が欠ける場合はmainが確認する。別Workflowを暗黙に開始しない。

### feature-development内

- 親から、現在の永続6文書のパス、同じ版へのreview-docs Gate PASS結果、[要求差分](../feature-development/references/incremental-development.md)、調査結果、出力先steeringを明示的に受け取る。日付付き仕様は任意の補助入力とする。
- 子はGateを再採点しない。PASS未取得・文書改訂後・不足入力なら未完了として親へ返す。必要な判断は [自律判断Policy](../feature-development/references/autonomy-policy.md) に従い親が決め、子からユーザーへ質問しない。
- 仕様ファイルを捏造せず、6文書と差分の根拠をrequirementsから参照する。計画完了後は親へ返し、親が実装を継続する。

両経路とも永続6文書（product-requirements、functional-design、architecture、repository-structure、development-guidelines、glossary）がdocsに揃っていることを確認する。不足時は単独なら `$setup-project` を案内し、親実行なら不足を返す。

## 実行手順

1. mainが入力、計画範囲、出力先、完了条件を確定する。単独では仕様名から `.steering/[YYYYMMDD]-[feature-name]/` を選ぶ。親実行では親が実行日と機能名（省略時はimplementation）から一意に選ぶ。再開は既存の記録と一致する対象だけを使い、無関係な計画を上書きしない。
2. 関連コード・テスト・6文書を調査する。親の調査を再利用し、必要なら `explorer` に狭い問いを委任する。
3. `feature_planner` に対象steeringだけの所有権を渡し、`$steering` を明示使用させる。他の作業者がおり、他者の変更を戻さないことを伝える。
4. requirementsへ目的、要件・AC、スコープ外、正本参照を記載する。親実行では比較基準、Unchanged/Added/Modified/Removedと現在の状態、是正対象・削除影響を含める。
5. designへ現状、責務、SwiftUIの状態所有、モデル、サービス、非同期・エラー、テスト戦略と実装順を記載する。アプリでは既存targetへの所属や設定への影響も確認する。不要な節や設計パターンを強制しない。
6. tasklistへ依存順の小さなtask、所有範囲、完了条件、検証・Regression範囲を定義する。既存Verifiedの再利用根拠、未達の是正、実機限定確認は区別する。
7. plannerの完了を待ち、mainが根拠との対応、3文書の整合、最終diff、所有範囲外の変更がないことを確認する。

## 完了条件

- 対象steeringのrequirements.md、design.md、tasklist.mdだけを計画成果物として作成・更新した。
- アプリ、入力仕様、永続文書は変更していない。
- 単独ではsteeringパス付きで `$implement-steering` を案内して終了する。親実行では同じパスを親へ返す。planner自身は実装しない。
