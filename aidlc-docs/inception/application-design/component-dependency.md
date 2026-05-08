# コンポーネント依存関係

## 依存関係方針
- 外側（Gateway）から内側（Service/Domain）への一方向依存
- 入口ごとの差異はGateway層に閉じ込め、変換ロジックは共通化
- 永続化アクセスはPersistenceComponentを経由して統制

## 依存マトリクス

| From | To | 目的 |
|---|---|---|
| SlackGatewayComponent | SlackInteractionService | Slack入力処理委譲 |
| SlackInteractionService | ReplyService | 返信文ドラフト生成/返信送信オーケストレーション |
| WebGatewayComponent | ReplyService | Web返信文ドラフト生成/返信送信実行 |
| WebGatewayComponent | HistoryQueryService | 履歴一覧取得 |
| ReplyService | UserContextService | 返信文ドラフト生成用文脈取得 |
| ReplyService | ReplyDraftGenerationComponent | 返信文ドラフト生成実行 |
| ReplyService | HistoryComponent | 返信文ドラフト履歴/送信済み返信履歴保存 |
| HistoryQueryService | HistoryComponent | 履歴取得 |
| AuthService | AuthComponent | OAuth認証実行 |
| UserContextService | UserContextComponent | 設定取得/更新 |
| HistoryComponent | PersistenceComponent | DBアクセス |
| UserContextComponent | PersistenceComponent | DBアクセス |
| AuthComponent | PersistenceComponent | セッション永続化 |

## 通信パターン
- **Slack -> Gateway -> Service -> Domain -> DB**
- **Web -> Gateway -> Service -> Domain -> DB**
- すべて同期 request/response を基本とする
- 公開API経路は将来拡張で追加予定

## データフロー（テキスト）
1. 入力データ（文脈 + 感情 + ユーザーID）をGatewayが受信
2. Service層でユーザー文脈を取得し、返信文ドラフト生成を実行
3. 出力データ（返信文ドラフト）を履歴保存
4. ユーザー確定後、返信送信と送信済み返信履歴保存を実行
5. チャネルに応じたレスポンス形式で返却

## 例外・エラー伝播
- DomainエラーはService層で分類
- Gateway層でユーザー向けメッセージへ変換
- ErrorHandlingServiceで監視ログとトレースを付与
