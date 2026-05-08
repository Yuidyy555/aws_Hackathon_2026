# アプリケーション設計（統合）

## 設計対象
- Slackアプリ（Slash Command中心）
- Webアプリ（入力/変換/履歴一覧）
- OAuth認証（Cognito Hosted UI）
- RDB永続化
- 公開API（将来拡張、MVP対象外）

## 主要決定事項（Q&A反映）
- コンポーネント分割: チャネル別優先（Q1=A）
- Slack MVP: Slash Command中心（Q2=A）
- Web MVP範囲: 入力 + 変換 + 履歴一覧（Q3=A）
- 公開API: 将来追加（MVPでは未実装）
- 認証方式: Cognito Hosted UI（Q4=A）
- データ設計: 正規化重視（Q5=A）
- 共通返信文ドラフト生成インターフェース: 同期API（Q6=A）
- エラーハンドリング: ユーザー表示と監視のバランス（Q7=C）
- 命名規約: 送信前は `reply-draft`、送信済みは `reply`
- 返信文ドラフト生成の必須入力: `replyTargetMessageText`（返信対象文） + `emotionText`（感情入力）

## 成果物一覧
- `components.md`: コンポーネント責務とインターフェース
- `component-methods.md`: 高レベルメソッドシグネチャ
- `services.md`: サービス層定義とオーケストレーション
- `component-dependency.md`: 依存関係/通信パターン/データフロー

## 設計妥当性サマリー
- 要件FR-01〜FR-08に必要な責務がコンポーネントへ割り当て済み
- User Stories（US-01〜US-09）に対応する入口・中核処理・永続化が定義済み
- MVPスコープ（一覧重視、同期API、Slash Command中心）と整合

## 次ステップ（Units Generationへの引き継ぎ観点）
- コンポーネントを実装単位へ分解する
- 変換系・履歴系・認証系・入口系で依存順序を定義する
- NFRとインフラ設計へ引き継ぐ設計制約を明文化する
