# コンポーネント定義

## 設計方針
- 分割方針は **チャネル別優先**（Q1=A）
- 共通変換ロジックは同期APIで提供（Q6=A）
- OAuthはCognito Hosted UIを採用（Q4=A）
- エラー処理はユーザー表示と監視情報のバランス重視（Q7=C）

## コンポーネント一覧

### 1. SlackGatewayComponent
- **目的**: Slash Commandを入口に、Slack向け変換フローを受け付ける
- **責務**:
  - Slack署名検証
  - コマンド入力の正規化
  - 返信対象文（reply target）と感情入力の抽出・必須チェック
  - 変換要求を共通APIへ連携
  - Slack向けレスポンス整形
- **公開インターフェース**:
  - `POST /slack/commands/generate-reply-draft`
  - `POST /slack/commands/send-reply`

### 2. WebGatewayComponent
- **目的**: Webアプリからの入力/変換/履歴一覧操作を受け付ける
- **責務**:
  - 入力バリデーション
  - 返信対象文（reply target）と感情入力の必須チェック
  - 認証済みユーザーコンテキスト付与
  - 変換・履歴ユースケース呼び出し
- **公開インターフェース**:
  - `POST /web/reply-drafts`
  - `POST /web/replies`
  - `GET /web/replies/history`

### 3. PublicApiGatewayComponent（将来拡張）
- **目的**: 将来の外部連携需要に備えた公開API入口
- **MVP扱い**: **非対象（実装しない）**
- **責務（将来）**:
  - API契約の一元化
  - チャネル間で共通の入力/出力スキーマ適用
  - エラーコード標準化
- **将来インターフェース案**:
  - `POST /api/v1/reply-drafts`
  - `POST /api/v1/replies`
  - `GET /api/v1/replies/history`

### 4. AuthComponent
- **目的**: OAuth認証とユーザー同定を提供する
- **責務**:
  - Cognito Hosted UIフロー開始/コールバック処理
  - セッション発行/検証
  - 未認証アクセスの遮断
- **公開インターフェース**:
  - `GET /auth/login`
  - `GET /auth/callback`
  - `POST /auth/logout`

### 5. ReplyDraftGenerationComponent
- **目的**: 感情・意図を汲み取ったビジネス文章変換を提供する中核コンポーネント
- **責務**:
  - 返信対象文 + 感情入力を前提とした生成リクエストの統一処理
  - LLM呼び出しとプロンプト組み立て
  - 変換結果の品質基準チェック（形式、空文字、長さ）
- **公開インターフェース**:
  - `generateReplyDraft(request): ReplyDraftResult`

### 6. HistoryComponent
- **目的**: 変換履歴の保存・一覧参照を提供する
- **責務**:
  - ユーザー単位の履歴永続化
  - 日付順の履歴一覧取得
  - 履歴再利用用データ提供
- **公開インターフェース**:
  - `saveReplyDraftHistory(historyRecord): HistoryId`
  - `saveSentReplyHistory(historyRecord): HistoryId`
  - `listSentRepliesByUser(userId, pagination): SentReplyHistoryList`

### 7. UserContextComponent
- **目的**: 会社コンピテンシー/自己コンテキストの設定を管理する
- **責務**:
  - 設定情報の取得・更新
  - 未設定時のフォールバック定義適用
  - 変換時に利用する文脈データ整備
- **公開インターフェース**:
  - `getContext(userId): UserContext`
  - `upsertContext(userId, payload): UserContext`

### 8. PersistenceComponent
- **目的**: RDBアクセスを一元化し、データ整合性を担保する
- **責務**:
  - トランザクション管理
  - Repository実装の提供
  - DBエラーのドメイン例外変換
- **公開インターフェース**:
  - `historyRepository`
  - `userRepository`
  - `contextRepository`
