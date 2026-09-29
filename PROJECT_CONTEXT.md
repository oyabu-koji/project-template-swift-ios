# Project Context

## 目的

このリポジトリは、SwiftでネイティブiOSアプリをAI駆動開発するための再利用テンプレートである。アプリケーション固有の要件、実装、Xcodeプロジェクトは含めず、仕様作成、設計、実装、検証を一貫して進めるCodex開発環境を提供する。

## 技術前提

- Language: Swift
- UI: SwiftUI
- Platform: native iOS
- IDE / build system: Xcode
- Build / test environment: macOS + Xcode + 対応するiOS SDK・Simulator runtime
- Xcodeバージョン、SwiftコンパイラとSwift言語モード、最低対応iOS、対象端末は実際のアプリrepoで確定し、この文書へ記録する。このテンプレートでは具体値を固定しない。

## 初期プロジェクトと開発

- このテンプレートrepoに `.xcodeproj` / `.xcworkspace` は不要である。
- 実際のアプリrepoで利用対象のproject/workspaceがなければ、ユーザーがXcodeで「iOS App / SwiftUI / Swift」を選んで作成する。Codexは独自生成せず、作成を案内して停止する。
- 作成後はCodexが既存のproject/workspace、scheme、target、destination、テスト構成を確認し、計画に従ってSwiftコードを実装する。名前や実行先は推測しない。
- ビルド・テストは確認済みの構成を指定した `xcodebuild`、起動・画面確認はXcodeとSimulatorまたは実機を使う。詳細は `.agents/skills/development-guidelines/references/process.md` を参照する。

## Workflow

- 初期開発: `$init-project` → `$define-requirements` → `$setup-project`
- Ideas・要求整理: `$review-docs` Interactive Modeと `$define-requirements` で質問・代替案を交えて合意する
- 永続6文書確定後の一括実行: `$feature-development` がGate Review、差分計画、実装、Build/Test、独立Validation、修正・再検証を管理する。工程ごとの承認は求めない
- 個別実行も維持する: `$prepare-steering` → `$implement-steering` → `$validate-implementation`。個別Skillはその工程だけで終了し、矢印は次工程の案内を示す
- `$feature-development` の入力正本は永続6文書。追加のconfirmed specは合意済み変更を永続文書へ反映してからGate Reviewを行う。単独のprepareは引き続き日付付きspecを入力とする
- reviewとvalidationは読み取り専用の独立評価とし、自動修正はmain agentが管理する。追加開発では差分実装・回帰確認と永続6文書全体の整合確認を行う
- invocation policyは `AGENTS.md`、自律判断と停止条件は `.agents/skills/feature-development/references/autonomy-policy.md` を参照する

## 制約

- 確定済みのXcode・Swift設定・最低対応iOSを明示依頼なしに変更しない
- Apple標準機能を基本とし、外部依存は必要性と導入範囲を合意した場合だけ追加する
- XcodeGen、Tuist、その他のproject generatorを導入しない。SwiftLint、CocoaPods等の外部ツールを標準導入しない
- ビルドやテストのために署名設定、Team、Capabilities、依存解決結果を無断変更しない
- プロジェクト固有の要件は `docs/ideas/initial-requirements.md` から開始し、安定後は `docs/` の永続文書を正本とする
