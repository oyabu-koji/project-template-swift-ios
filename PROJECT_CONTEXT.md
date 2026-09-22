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
- セットアップ後の機能開発: `$define-requirements` → `$prepare-steering` → `$implement-steering` → `$validate-implementation`
- `$review-docs` は必要時の独立した読み取り専用レビューとする
- 要件定義は対話と合意、setupは永続6文書、prepareはsteering計画、implementは実装と進捗・検証証跡、validateは変更を伴わない独立検証を担当する
- 次工程は案内に留め、別Workflowを暗黙に開始しない。invocation policyと詳細なroutingは `AGENTS.md` を参照する

## 制約

- 確定済みのXcode・Swift設定・最低対応iOSを明示依頼なしに変更しない
- Apple標準機能を基本とし、外部依存は必要性と導入範囲を合意した場合だけ追加する
- XcodeGen、Tuist、その他のproject generatorを導入しない。SwiftLint、CocoaPods等の外部ツールを標準導入しない
- ビルドやテストのために署名設定、Team、Capabilities、依存解決結果を無断変更しない
- プロジェクト固有の要件は `docs/ideas/initial-requirements.md` から開始し、安定後は `docs/` の永続文書を正本とする
