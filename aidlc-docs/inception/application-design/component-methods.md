# コンポーネントメソッド定義

## 1) SlackGatewayComponent
- `generateReplyDraftFromSlashCommand(payload: SlackReplyDraftCommandPayload, headers: HeaderMap): SlackReplyDraftResponse`
  - 署名検証後、`replyTargetMessageText`（返信対象文）と `emotionText`（感情入力）を含む生成要求を作成してReplyServiceへ委譲
- `buildSlackReplyDraftResponse(result: ReplyDraftResult): SlackReplyDraftResponse`
  - Slack表示形式へ返信文ドラフトを整形
- `sendReplyFromSlackCommand(payload: SlackSendReplyPayload, headers: HeaderMap): SlackSendReplyResponse`
  - ユーザーが確定した返信文を送信し、送信済み返信履歴を保存

## 2) WebGatewayComponent
- `createReplyDraft(request: WebReplyDraftRequest, session: UserSession): WebReplyDraftResponse`
  - `replyTargetMessageText`（返信対象文）と `emotionText`（感情入力）を検証し、返信文ドラフト生成とドラフト履歴保存を実行
- `sendReply(request: WebSendReplyRequest, session: UserSession): WebSendReplyResponse`
  - ユーザー確定後の返信送信を実行し、送信履歴へ記録
- `listReplyHistory(query: ReplyHistoryQuery, session: UserSession): ReplyHistoryListResponse`
  - ユーザー単位の送信済み返信履歴一覧を取得

## 3) PublicApiGatewayComponent（将来拡張）
- `createReplyDraft(request: ReplyDraftApiRequest, principal: Principal): ReplyDraftApiResponse`
  - 将来の公開API向け（MVPでは未実装）。`replyTargetMessageText` と `emotionText` を必須入力とする想定
- `sendReply(request: ReplyApiRequest, principal: Principal): ReplyApiResponse`
  - 将来の公開API向け（MVPでは未実装）
- `listReplyHistory(query: ReplyHistoryApiQuery, principal: Principal): ReplyHistoryApiResponse`
  - 将来の公開API向け（MVPでは未実装）

## 4) AuthComponent
- `startLogin(redirectUri: string): RedirectResponse`
  - Cognito Hosted UIへリダイレクト
- `handleCallback(code: string, state: string): AuthSession`
  - 認可コード交換、ユーザー同定、セッション発行
- `logout(sessionId: string): void`
  - セッション破棄

## 5) ReplyDraftGenerationComponent
- `generateReplyDraft(input: ReplyDraftInput): ReplyDraftOutput`
  - `replyTargetMessageText` と `emotionText`、必要に応じてユーザー文脈を用いてプロンプト生成し、返信文ドラフトを生成
- `validateReplyDraftInput(input: ReplyDraftInput): ValidationResult`
  - 必須項目（返信対象文/感情入力）と長さ/形式をチェック
- `normalizeReplyDraftOutput(raw: string): NormalizedText`
  - 文体・改行・不要記号を整形

## 6) HistoryComponent
- `saveReplyDraftHistory(record: ReplyDraftHistoryRecord): HistoryId`
  - 返信文ドラフト生成履歴をユーザー単位で保存
- `saveSentReplyHistory(record: SentReplyHistoryRecord): HistoryId`
  - 送信済み返信履歴をユーザー単位で保存
- `findSentRepliesByUser(userId: string, page: PageRequest): SentReplyHistoryPage`
  - 送信済み返信を日付降順で一覧取得

## 7) UserContextComponent
- `getUserContext(userId: string): UserContext`
  - コンピテンシー/自己コンテキスト取得
- `upsertUserContext(userId: string, payload: UserContextPayload): UserContext`
  - 設定の新規作成または更新

## 8) PersistenceComponent
- `withinTransaction<T>(fn: () => T): T`
  - トランザクション境界の統一
- `mapDbError(error: DbError): DomainError`
  - DB例外のドメイン例外化

## 注記
- ここでは高レベルのシグネチャと責務のみを定義する。
- 詳細な業務ルール・分岐条件は CONSTRUCTION フェーズの Functional Design で定義する。
