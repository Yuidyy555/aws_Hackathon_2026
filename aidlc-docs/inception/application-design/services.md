# サービス定義

## サービスレイヤー方針
- 入口（Slack/Web）は薄く保ち、ユースケースはサービス層に集約する
- 変換処理は同期実行を基本とする（Q6=A）
- 履歴参照は一覧重視（Q3=A）で最小MVPを構成する
- 公開APIは将来拡張とし、MVPでは実装対象外とする
- 命名ルールは「送信前: `reply-draft` / 送信後: `reply`」で統一する

## サービス一覧

### 1. ReplyService
- **責務**:
  - 返信文ドラフト生成ユースケースのオーケストレーション
  - 返信送信ユースケースのオーケストレーション
  - UserContext取得 -> 返信文ドラフト生成 -> 履歴保存の統合
  - ユーザー向けエラーと監視向けエラー情報の分離
- **主要連携**:
  - `UserContextComponent`
  - `ReplyDraftGenerationComponent`
  - `HistoryComponent`

### 2. SlackInteractionService
- **責務**:
  - Slash Command入力の正規化
  - Slack向け応答形式への変換
  - 再試行可能なメッセージ整備
- **主要連携**:
  - `SlackGatewayComponent`
  - `ReplyService`

### 3. HistoryQueryService
- **責務**:
  - 送信済み返信履歴一覧取得（日時順）
  - ページング処理
  - 表示用DTOへの整形
- **主要連携**:
  - `HistoryComponent`

### 4. AuthService
- **責務**:
  - OAuthログインフロー管理
  - セッションライフサイクル管理
  - 認証状態検証
- **主要連携**:
  - `AuthComponent`

### 5. UserContextService
- **責務**:
  - コンピテンシー/自己コンテキスト設定の取得・更新
  - 未設定時フォールバック付与
- **主要連携**:
  - `UserContextComponent`

### 6. ErrorHandlingService
- **責務**:
  - 例外分類（ユーザー提示可否）
  - 監視ログ/トレース出力
  - APIエラーコード標準化
- **主要連携**:
  - すべてのGateway/Service

## オーケストレーションパターン

### 返信文ドラフト生成フロー（同期）
1. 入口コンポーネントで入力受領・認証検証（返信対象文 + 感情入力を必須）
2. `ReplyService` が `UserContextService` から文脈取得
3. `ReplyDraftGenerationComponent` で返信文ドラフト生成
4. `HistoryComponent` で返信文ドラフト履歴保存
5. 入口チャネル形式でレスポンス返却

### 返信送信フロー（同期）
1. ユーザーが返信文ドラフトを確定
2. `ReplyService` が送信処理を実行
3. `HistoryComponent` で送信済み返信履歴保存
4. 送信結果を返却

### 返信履歴一覧フロー
1. 認証済みユーザー確認
2. `HistoryQueryService` がユーザー単位で履歴取得
3. 一覧DTOを返却
