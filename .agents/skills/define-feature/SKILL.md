---
name: define-feature
description: docs/ideas配下の初期要件または日付付き機能仕様を日本語で作成・更新する。新規プロジェクトのbootstrap要件、追加機能の仕様化、既存specの更新に明示的に使用する。永続docs、.steering、アプリコードの作成には使用しない。
---

# Define Feature

ユーザーとの対話で要求とスコープを確定し、`docs/ideas/`の仕様正本へ反映する。

## 入力

- ざっくりしたアイデア、または更新対象の`docs/ideas/*.md`を受け取る。
- 明示されたパスは、正規化後もリポジトリの`docs/ideas/`配下にあるMarkdownファイルだけを受け付ける。`docs/ideas/initial-requirements.md`は不存在でも新規作成対象として受け付ける。それ以外の明示パスは、実在する更新対象に限る。
- 初期要件テンプレートには[initial-requirements-template.md](assets/initial-requirements-template.md)、追加仕様テンプレートには[feature-spec-template.md](assets/feature-spec-template.md)を使う。
- 初期要件の状態は[initial-requirements-validation.md](references/initial-requirements-validation.md)で判定する。

## モード選択

1. `docs/ideas/initial-requirements.md`のファイル存在状態を記録し、共通判定基準で内容を検査する。
2. 結果が`missing`または`blank`なら、初期要件作成・更新モードを選ぶ。追加仕様へ進まない。
3. 結果が`valid`で、既存の`docs/ideas/*.md`が明示されていれば既存仕様更新モードを選ぶ。
4. 結果が`valid`で、新しい機能アイデアが入力されていれば追加仕様作成モードを選ぶ。
5. 対象や意図が複数に解釈できる場合は、書き込み前にユーザーへ短く確認する。

## 実行手順

1. `AGENTS.md`、`PROJECT_CONTEXT.md`、対象仕様、関連する既存`docs/`を必要最小限だけ読む。
2. main agentが日本語で不足情報を確認し、目的、対象ユーザー、スコープ、スコープ外、受け入れ条件、未決事項を決定する。
3. 初期要件ファイルが存在せず`missing`なら、assetを`docs/ideas/initial-requirements.md`へ複製し、確定内容でプレースホルダーを置き換える。既存ファイルが`missing`なら、入力済み内容を保持して不足セクションだけを追加・補完する。既存ファイルが`blank`なら、構成と入力済み内容を保持して空欄またはプレースホルダーだけを更新する。既存ファイル全体をassetで置換しない。ユーザーが既存要件の変更を指定した場合は、合意した変更と整合に必要な関連箇所を反映し、それ以外の入力済み内容を保持する。作成・更新時の技術バージョンは`PROJECT_CONTEXT.md`から確認する。
4. 追加仕様を新規作成する場合は`docs/ideas/YYYYMMDD-[feature-name].md`を使う。日付は作成日、feature nameは英小文字のkebab-caseにする。
5. custom agent `document_author`へ確定内容から対象仕様だけを作成・更新するよう委任する。所有範囲を対象1ファイルに限定し、同じコードベースに他の作業者がいること、他者の変更を戻さないことを伝える。
6. `document_author`から結論、変更内容、参照した根拠、残課題の要約を受け取る。
7. main agentが最終diff、テンプレート残存、受け入れ条件の検証可能性、所有範囲外の変更を確認する。
8. 安定した要求が既存`docs/`へ影響する場合は更新候補を仕様へ記録する。このSkill内では永続文書を変更しない。

## 停止条件

- 指定パスが`docs/ideas/`外、またはMarkdownでない。
- 更新対象の指定パスが存在しない。ただし、新規作成対象の`docs/ideas/initial-requirements.md`は除く。
- 初期要件が`missing`または`blank`のまま追加仕様を求められた。
- 必須情報を合理的に確定できない。
- `document_author`を使用できず、委任必須の文書作成を完了できない。

## 完了条件

- 初期要件は共通判定で`valid`になり、`$setup-project`へ渡せる。
- 既存の初期要件を更新した場合、不足・空欄の補完とユーザーが合意した要件変更が反映され、それ以外の入力済み内容が保持されている。
- 追加仕様はテンプレートの必須論点を満たし、`$plan-feature`へ渡せる。
- 既存仕様の更新では対象ファイルだけが変更されている。
- `.steering/`、永続文書、アプリコードを変更していない。
