# 実装検証の判定契約

`validate-implementation` と `implementation_validator` の判定のSingle Source of Truth。`feature-development` はこの結果を利用し、別の採点や緩和で上書きしない。仕様品質のGate判定は `review-docs` の責務とする。

## 入力と対象

- 対象steeringの `requirements.md`、`design.md`、`tasklist.md`。
- 現在の6永続文書: `docs/product-requirements.md`、`docs/functional-design.md`、`docs/architecture.md`、`docs/repository-structure.md`、`docs/development-guidelines.md`、`docs/glossary.md`。要求・ACだけでなく、設計制約、開発規則、用語の整合も全体として確認する。
- 関連仕様と現在コード・テスト、変更前後のGit差分/履歴、親が作成した要求単位の `Unchanged` / `Added` / `Modified` / `Removed` 分類と影響範囲。
- 過去のValidation結果、今回のbuild/test/AC確認の証跡。要求版、コード版（commitと未commit差分を含むスナップショット）、実行環境と証跡の対象が対応していること。

単独呼び出しで差分分類が未提供ならmainが現在の仕様・履歴・既存証跡から入力を補う。比較元が不明なら不明と記録し、現在の全要求を検証する。過去証跡がないこと自体は停止理由にしない。steeringの3文書またはアプリの6永続文書が不足する場合は、足りない入力を `BLOCKED` として返す。テンプレート・文書のみの変更では、存在する規約と対象成果物を検証し、アプリ固有文書/build/testの非適用理由を示す。この例外をアプリの機能開発へ適用しない。

## Traceabilityと状態

現在の6永続文書の要求・ACをすべて列挙する。元のIDを使い、IDがなければファイル/見出し/項目を安定した参照として使う。検証のために仕様を書き換えない。設計制約や用語は対応する要求へのリンク、または個別の整合確認行で網羅する。

各行に少なくとも以下を含める。

| 項目 | 内容 |
| --- | --- |
| 要求・AC | ID、要求本文の参照と版、P0/禁止動作との関係 |
| 差分と影響 | Unchanged / Added / Modified / Removed、影響を受ける既存要求 |
| 実装状態 | Implemented / Not Implemented / Not Applicable |
| コード根拠 | ファイル・位置・該当する分岐や動作。部分実装と不足も示す |
| 検証状態 | Verified / Not Verified / Failure / Not Applicable |
| 検証根拠 | ACごとの確認方法、テスト名・位置、実行結果、ログ/result bundle等の証跡 |
| 再利用・回帰 | 過去Verified証跡の再利用根拠、影響分析、今回のRegression結果 |
| 残課題 | 未実装/Failure/未検証の理由、修正候補、再検証方法 |

実装状態と検証状態を一つに潰さない。

- `Implemented`: 要求の全実装がコードで確認できる。テスト未実行でも実装根拠があればこの状態になり得るが、`Verified` は別判定。
- `Not Implemented`: 未実装、部分実装、または要求された動作が不足している。未確認で根拠を得られない場合は状態を確定せず、検証状態を `Not Verified`、理由を「実装根拠不足」と明記する。
- `Verified`: 要求/ACに適した検証が成功し、現在の仕様・実装へ適用できる証跡がある。コード読解やbuild成功だけで実行時ACをVerifiedにしない。
- `Not Verified`: 検証証跡が不足、未実行、環境で実行不能、0件実行、または必須確認がskipされている。原因を区別する。
- `Failure`: 実行された検証が要求/ACを満たさない。最後の失敗を過去の成功で隠さない。
- `Not Applicable`: 現在の仕様上その確認が対象外である根拠が必要。仕様参照、適用条件と今回の対象との関係を明記し、未実装・失敗・環境不足・実機待ちの代替にしない。片方の軸だけが非適用の場合も理由を別々に示す。

実装状態を確定できない行は空欄ではなく `未判定（根拠不足）` と記録し、判定をPASSにしない。

実機iPhoneだけで検証可能なACは `Not Verified - requires physical device` と表記する。実機が必要な理由、確認済みのコード/Simulator範囲、未確認動作、実機での確認手順を付す。実装不足を実機待ちへ分類せず、Simulatorや自動化で確認可能な部分は先に検証する。

## 差分検証と証跡

- `Added` / `Modified` を今回の重点対象とし、これらが影響する `Unchanged` 要求をRegression対象に含める。全体対応表から変更のない要求を消さない。
- 過去 `Verified` の再利用には、参照できる旧結果、要求とコードの対応、依存/API/設定/実行環境の変化、今回の変更から影響を受けない理由が必要。関係する変更・証跡不足・不明な影響があれば再検証する。再利用は不要な再実装を避けるためであり、今回必要なbuild/testやRegressionを省略するためではない。
- `Removed` は旧要求との追跡行を残し、現行仕様が機能削除を要求するか、その影響と削除後の回帰を確認する。差分ラベルだけで削除やNot Applicableを正当化しない。
- build/testはmainが [共通Xcode検証手順](../../development-guidelines/references/process.md) に従って実行する。validatorは既存証跡の読み取りと必要な追加確認の提示に徹する。実在する入口、scheme、target、configuration、destination、Test action/test planを確認してから実行し、設定を推測しない。
- 証跡にコマンド、ツール版、対象構成、実行日時、コード版、終了結果、実行テスト数、skip数、失敗数、ログの場所を含める。成功、失敗、未設定、未実行、非適用を区別する。終了コード0だけではテスト成功としない。
- 固定カバレッジ閾値は `docs/development-guidelines.md` で合意済みの場合だけ適用する。build成功を起動・画面・権限・実機確認の代わりにしない。

## 判定

`PASS` は以下をすべて満たす場合に限る。

1. 6永続文書全体とsteering・コード・テストのTraceabilityが揃い、整合している。
2. 現行仕様が要求する実装はすべて `Implemented`。非適用行には仕様上の根拠がある。
3. 現在のコードに対する必須buildが成功し、自動testが実際に実行され成功している。必要なRegressionと自動検証可能なACがすべて成功し、その対象にFailure/Not Verifiedがない。必須テスト未設定、0件実行、必須確認のskip、未実行をPASSにしない。
4. Critical = 0、Major = 0。Minor/Suggestionが残る場合も要求・ACの充足や完了条件を損なわない。
5. 残る未検証がある場合は、実装が完了した実機限定ACだけであり、すべて `Not Verified - requires physical device` として明示されている。

実機限定未検証が残る `PASS` は「実装範囲PASS（実機確認残あり）」と報告し、「全AC Verified」「実機検証済み」と報告しない。実機未検証を隠すためにACを非適用へ変更しない。全ACがVerifiedかどうかを判定とは別フィールドで返す。

テンプレート・文書のみの変更では、アプリのbuild/testの代わりに成果物に適した構文・参照・整合確認を実施し、非適用の根拠を報告する。アプリ機能開発の完了条件はこの例外で緩和しない。

`FAIL` は要求のNot Implemented、実行済みbuild/test/ACのFailure、必要な回帰の不具合、Critical/Major、または既知の不整合がある場合。必要な検証を未実行にして既知のFAILをBLOCKEDへ置き換えない。

`BLOCKED` は既知のFAILがなく、入力・証拠・実行環境・権限等が足りず判定を確定できない場合。実機限定の例外以外の必須自動検証未実行、0件実行、skipを含む。自動修復できる不足と外部要因を区別して返す。FAILと検証障害が併存する場合は、主判定FAILに障害も列挙する。

CriticalはP0/禁止動作、重大なセキュリティ/プライバシー、データ損失等を損なう問題、Majorは要求/ACの不達、主要な回帰、判定に必須の検証設計不足とする。Minorは要求/ACに影響しない局所的品質問題、Suggestionは任意の改善提案とする。

## 返却と修正ループ

日本語で以下を返す。

1. `PASS` / `FAIL` / `BLOCKED`、判定理由、Critical/Major/Minor/Suggestion件数、全AC Verifiedの真偽、実機確認残の有無。
2. 6永続文書全体のTraceability表と、未実装・Failure・未検証・非適用の件数および根拠。
3. 重大度順のfinding: 対応する要求/AC、ファイルと位置、影響、再現/確認方法、修正候補、再検証範囲。
4. build/test/Regression/ACの実行結果と証跡、過去Verified再利用の根拠、実機確認手順。
5. 親が実施できる修正・追加検証と、親の [自律判断Policy](../../feature-development/references/autonomy-policy.md) の停止条件に該当する問題があればその根拠。

validatorはコード、テスト、設定、docs、steeringを修正せず、ユーザーへ直接質問しない。standaloneでは結果の報告で終了し、修正Workflowを暗黙に開始しない。`feature-development` 内ではmainへ結果を返し、mainが修正と証跡/進捗更新を行う。mainは停止条件に該当しないNot Implemented、Failure、自動修復可能な検証不足を修正し、Build → Test → Validationを繰り返す。FAIL/BLOCKEDというラベル自体はユーザー確認の理由ではない。再検証でもvalidatorがこの契約で判定し、親が判定を再定義しない。
