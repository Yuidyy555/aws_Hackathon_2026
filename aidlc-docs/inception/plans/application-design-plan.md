# アプリケーション設計計画

## 目的
Slackアプリ + Webアプリ + APIで共通の変換体験を提供するため、コンポーネント境界・サービス責務・依存関係を定義する。

## 実行チェックリスト
- [x] 1. 要件・ユーザーストーリー・実行計画を読み込み、設計スコープを確定する
- [x] 2. 設計方針（コンポーネント分割、依存方向、責務分離）を定義する
- [x] 3. 質問項目を作成し、[Answer]形式でユーザー入力を受ける
- [x] 4. 回答を分析し、曖昧点・矛盾点を解消する
- [x] 5. `components.md` を作成する
- [x] 6. `component-methods.md` を作成する
- [x] 7. `services.md` を作成する
- [x] 8. `component-dependency.md` を作成する
- [x] 9. `application-design.md`（統合ドキュメント）を作成する
- [x] 10. 整合性チェックを実施する
- [ ] 11. レビュー承認を取得する

## 必須成果物（作成予定）
- [x] `aidlc-docs/inception/application-design/components.md`
- [x] `aidlc-docs/inception/application-design/component-methods.md`
- [x] `aidlc-docs/inception/application-design/services.md`
- [x] `aidlc-docs/inception/application-design/component-dependency.md`
- [x] `aidlc-docs/inception/application-design/application-design.md`

## 設計前提（現時点）
- チャネル: Slack（自動返信補助） + Web（履歴参照/操作） + API
- 技術: Ruby、RDB必須、OAuth必須
- 価値軸: 感情・意図を汲み取った自然なビジネス文章生成

## 設計確認質問

以下の各質問で、`[Answer]:` の後に選択肢を記入してください。  
該当しない場合は最後の `X) Other` を選んで補足してください。

## Question 1
コンポーネント境界はどの分割方針を優先しますか？

A) チャネル別（Slack入口 / Web入口 / API入口）に明確分離
B) ドメイン別（変換、履歴、認証、ユーザー設定）を優先し、入口は薄くする
C) MVP速度優先で最小分割（入口 + 変換 + 永続化の3層）
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2
Slack連携のMVP実装形態はどれを優先しますか？

A) まずはSlash Command中心（操作の明示性を重視）
B) Event Subscriptions中心（メッセージイベント起点）
C) 双方を最小構成で対応
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3
WebアプリのMVP範囲をどこまで含めますか？

A) 入力 + 変換 + 履歴一覧（最小）
B) Aに加えて履歴詳細と再利用導線
C) Bに加えて設定画面（コンピテンシー/自己コンテキスト）まで含める
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4
認証方式（OAuth）はどの方針を取りますか？

A) Cognito Hosted UIベースで実装（AWS標準）
B) 外部IdP（Google等）を主にしてCognitoで連携
C) MVPでは認証を最小実装し、履歴機能は限定提供
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5
データ設計の優先方針はどれですか？

A) 正規化重視（拡張性優先）
B) クエリ性能重視（履歴参照を最優先）
C) MVP簡素化重視（必要最小テーブル）
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6
共通変換ロジックの公開インターフェースはどの形が良いですか？

A) 同期API（request/response）を基本とする
B) 非同期ジョブ化を前提にする
C) 同期を基本に、重い処理のみ非同期へ逃がす
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7
エラーハンドリングの設計優先順位はどれですか？

A) ユーザー向け分かりやすさ優先（再試行導線重視）
B) 運用監視優先（詳細ログとトレース重視）
C) AとBのバランス（ユーザー表示 + 監視情報の両立）
X) Other (please describe after [Answer]: tag below)

[Answer]: C
