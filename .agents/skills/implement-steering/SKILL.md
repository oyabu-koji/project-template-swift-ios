---
name: implement-steering
description: 指定された.steeringタスクのrequirements、design、tasklistに従って機能を実装し、進捗と検証証跡を同期する。計画済みタスクを実行する場合に明示的に使用する。計画作成、計画なしの実装、独立した最終検証には使用しない。
---

# Implement Steering

`.steering/`を進捗の正本として、所有範囲を分けながらタスクを1件ずつ実装する。

## 呼び出しモード

単独実行はこのSkillの工程だけで終了する。`feature-development`から親コンテキストと対象steeringを受け取った場合は [自律判断Policy](../feature-development/references/autonomy-policy.md) を適用し、結果を親へ返す。下記の入力不足・矛盾・所有競合は親へ返し、子からユーザーへ工程承認を求めない。親実行ではmain自身が実装・修正・Build/Testを担当でき、workerの使用は必要時に限る。

## 入力契約と停止条件

- `.steering/[YYYYMMDD]-[task]/`を1件、明示入力として受け取る。
- 引数なし、複数候補、対象が曖昧な場合は、ユーザーへ対象steeringを確認する。最新・未完了などの理由で別のsteeringを勝手に選択しない。
- 対象が存在しない、または`requirements.md`、`design.md`、`tasklist.md`のいずれかが不足する場合は変更を開始せず、`$prepare-steering`を案内する。新しいsteeringを勝手に作成せず、別Workflowも暗黙に開始しない。`.steering/`外の指定は受け付けず対象の再指定を求める。
- 要求、設計、taskの矛盾や重要な製品判断は、単独実行ならmainが確認し、親実行ならPolicyの停止条件で扱う。根拠から決まる内部設計はmainが決定する。
- 未コミット変更と所有対象が重なる場合は、既存変更を保護できる方針を確定するまで停止する。

## 実行手順

1. main agentが3文書と関連する永続文書を読み、taskの依存順、受け入れ条件、技術制約を確認する。
2. `$steering`を明示使用し、着手するtaskを`in-progress`として`tasklist.md`へ反映する。一度に進行中にする書き込みtaskは1件にする。
3. 必要な読み取り調査を組み込み`explorer`へ委任する。独立した調査やログ解析は並列化してよいが、書き込みtaskとファイル所有を重ねない。
4. 単独実行では組み込み`worker`へtaskごとの実装とテストを委任する。親実行ではmainが実装するか、必要なtaskだけworkerに委任する。委任時は、所有ファイルまたはディレクトリ、入力、制約、完了条件を明示し、同じコードベースに他の作業者がいること、他者の変更を戻さないことを伝える。
5. 書き込みが競合するtaskは直列に実行する。複数workerを使う場合は所有範囲が重ならないことをmain agentが事前に確認する。
6. 委任した場合はworkerから結論、変更内容、根拠、実行した検証、残課題の要約を受け取る。main agentがdiff、要求との対応、所有範囲外の変更を確認する。
7. taskの受け入れ条件を満たした場合だけ`done`へ更新する。失敗は原因を記録し、修正可能ならin-progressのまま修正・再検証する。外部判断・環境に阻まれる場合だけblockedとして理由を記録し、依存taskを進めず独立taskを進める。
8. 実装が設計を変えた場合は、確定済みの要求に基づく判断と根拠を`design.md`へ反映する。親実行で許された内部判断に改めて承認を求めない。永続文書へ影響する安定した変更はdocument_authorへ所有範囲と合意済み根拠を指定して反映する。親実行では親へ戻してGateを再実行し、単独実行では別Workflowを開始しない。task状態と検証証跡は毎task後に`tasklist.md`へ同期する。

## ビルド・テスト構成と検証

1. [Xcode検証手順](../development-guidelines/references/process.md) を読み、実在するproject/workspace、scheme、app/test target、configuration、destinationと環境を確認する。アプリ実装でproject/workspaceがなければ独自生成せず、人間によるXcodeでのiOS App / SwiftUI / Swift作成を案内して停止する。テンプレート・文書のみのtaskではアプリ構成を要求しない。
2. 確認済みの構成からtaskに関連する `xcodebuild build` / `xcodebuild test` の引数を決める。project名、scheme名、Simulator名を固定しない。testはTest actionの有効なテスト対象と実行可能なdestinationを確認してから行う。
3. 未設定、失敗、環境・権限による未実行を区別する。未設定だけを自動的に失敗とはしないが、受け入れ条件が要求する検証は省略して完了にしない。親実行では自動Test成功が必須であり、未設定を合格にせず、既存方針と許可の範囲で必要なテストを整える。
4. 固定カバレッジ閾値は`docs/development-guidelines.md`で合意済みの場合だけ適用する。外部lintや合意外のテスト基盤を検証のためだけに追加しない。Apple標準テストの不足構成は、親実行ではXcode検証手順の「テスト構成が未設定の場合」に従い計画taskとして整える。
5. 親実行ではBuild/Test/Lint失敗の原因を自律修正して再実行する。検証失敗や必須項目の未確認があればtaskを完了扱いにせず、理由と再現方法を記録する。実行コマンド、対象構成、テスト実行件数、結果の場所を証跡に残す。
6. 必要な起動確認はSimulatorまたは実機で行い、build成功とは区別する。署名設定や依存を無断変更せず、生成物への書き込みも親ターンの権限に従う。

実機限定ACは `Not Verified - requires physical device` と記録し、実装taskと実機検証taskを分ける。実装と自動検証が完了したtaskだけdoneにでき、実機検証taskは未完了のまま残す。親は実機待ちを理由に他の実装可能なtaskを残さない。全体の実装完了と実機待ちの区別は [validate-implementationの契約](../validate-implementation/references/validation-contract.md) で判定する。

## 完了条件

- tasklistの対象実装taskが受け入れ条件と自動検証証跡付きで完了し、実機限定の検証taskがあれば未完了として別記されている。
- requirements、design、実装、テストが整合し、未解決事項が明示されている。
- 確認済み構成で必要な検証が実行され、結果と未確認範囲が記録されている。
- main agentが最終diffと所有範囲を確認している。
- 独立した最終検証は代行しない。単独実行では同じsteeringを明示して`$validate-implementation`を案内して終了する。親実行では対象steering・差分・検証結果を親へ返し、親がValidationと修正反復を続ける。
