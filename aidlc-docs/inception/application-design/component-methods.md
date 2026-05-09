# コンポーネントメソッド定義

## 共通: ドメインユーザーと外部 IO の対応付け

- **単一ソース・キー**: アプリ内部のユーザーを **`internalUserId`**（論理ユーザー主キー）で表す。**`UserContext`**・ドラフト／送信履歴はすべてこのキーで一貫し、特定チャネルの所有物ではない。
- **Web → `internalUserId`**: Cognito Hosted UI で得た **`sub`** でアプリ側 `User` を upsert。**`UserSession`** が **`internalUserId` を運ぶ**。ゲートウェイは **`AuthService`**/`AuthComponent` に確認し **`ReplyService` / `UserContextService` は ID のみを受け取る**。
- **Slack → `internalUserId`**: Slash の **`team_id` / `user_id`** は **`LinkedIdentityResolverService`** のみが `internalUserId` に結び、`SlackInteractionService` が利用する。**リンク未確立時**は解決サービスが失敗型を返し、以降オーケストレーションへ進めない。
- **`UserContext` の所在**: **`UserContextService`／`UserContextComponent`** が共有読み書き。**Web は HTTP 経路**。LLM向け入力の組み立ては **`ReplyDraftContextAssembler`** が担当し **`UserContextComponent` が「プロンプト用整形」を持たない**。

---

## 返信ドラフト生成フローの呼び出し順（`application-design.md` と整合）

**内部オーケストレーション（`ReplyService` 内の論理順）**

1. `UserContextService#getUserContext(internalUserId)`
2. **`ReplyDraftContextAssembler#assemble`（協調オブジェクト／モジュール）** → `ReplyDraftInput`
3. **`ReplyDraftGenerationComponent#generateReplyDraft`** → 内部で PromptBuilder → LanguageModelClient → OutputNormalizer
4. `HistoryComponent#saveReplyDraftHistory`

**Slack の戻り path（ドメイン結果 → ユーザー向け）**

5. `SlackReplyPresentationMapperComponent#…` が **`ReplyDraftResult` またはエラー区分** を Slack 応答へ

**Web**: ゲートウェイが JSON で返す（Step 5 相当なし。**HTTP 状態コード／JSON フィールドだけ**）。

---

## 1) SlackGatewayComponent

- `verifySlackRequest(headers: HeaderMap, body: RawBody): void`
  - 署名・再送防止など **Slack Request 検証のみ**。
- `handleGenerateReplyDraftWebhook(payload: SlackReplyDraftCommandPayload, headers: HeaderMap): SlackReplyDraftResponse`
  - 検証後 **`SlackInteractionService#handleGenerateReplyDraftCommand`** を呼ぶ。
  - 返却値を **`SlackReplyPresentationMapperComponent#formatGenerateReplyDraftResponse`** で Slack 応答へ（成功／未リンクなどの区分ごと）。
- `handleSendReplyWebhook(payload: SlackSendReplyPayload, headers: HeaderMap): SlackSendReplyResponse`
  - 同上の検証 → **`SlackInteractionService#handleSendReplyCommand`**（命名は Functional Design で確定）→ **`SlackReplyPresentationMapper`** で応答体裁。

---

## 2) SlackInteractionService

- `handleGenerateReplyDraftCommand(payload: SlackReplyDraftCommandPayload, headers?: HeaderMap): ReplyDraftUseCaseOutcome`
  - **`normalizeSlashCommandToReplyDraft(payload): NormalizedReplyDraftInputs`**（返信対象文・感情の抽出と必須チェック）。
  - **`LinkedIdentityResolverService#resolveSlackUser(teamId, slackUserId): IdentityResolution`**。
  - 成功時 **`ReplyService#createReplyDraft(internalUserId, normalized.replyTargetMessageText, normalized.emotionText)`**。
  - **戻り値はチャネル中立**（ドラフト結果 or リンク未達などのコード）。Slack メッセージ文字列は作らない。
- `normalizeSlashCommandToReplyDraft(payload: SlackReplyDraftCommandPayload): NormalizedReplyDraftInputs`
- （送信など）各 Slash ユースケースに対応する `handle*` を同サービスが持つことを許容。**本人解決は常に LinkedIdentity に委譲**。

---

## 3) LinkedIdentityResolverService

- `resolveSlackUser(teamId: string, slackUserId: string): IdentityResolutionResult`
  - 永続リンク参照。**成功時は `internalUserId`**。未リンクは **失敗**（`SlackInteractionService` と `SlackReplyPresentationMapper` が解釈する区分へ）。

---

## 4) SlackReplyPresentationMapperComponent

- `formatGenerateReplyDraftResponse(outcome: ReplyDraftUseCaseOutcome): SlackReplyDraftResponse`
  - `ReplyDraftResult`・リンク未達・入力エラーなど **区分入力**のみを Slack 表示に写像。**ビジネス判断はしない**。

---

## 5) WebGatewayComponent

（実装ではコントローラ分割可。論理グループのみ列挙。）

- `getUserContext(session: UserSession): UserContextDto`
  - セッションから `internalUserId` → **`UserContextService#getUserContext`**。
- `upsertUserContext(payload: WebUserContextUpsertRequest, session: UserSession): UserContextDto`
  - 同上。
- `createReplyDraft(request: WebReplyDraftRequest, session: UserSession): WebReplyDraftResponse`
  - **HTTP入力のスキーマ／必須検証のみ** → **`ReplyService#createReplyDraft`**。**Assembler／生成詳細には触れない**。
- `sendReply(...)`／`listReplyHistory(...)`
  - それぞれ **`ReplyService`**／**`HistoryQueryService`** に委譲。

---

## 6) AuthComponent

- `startLogin(redirectUri: string): RedirectResponse`
- `handleCallback(code: string, state: string): AuthSession`
- `logout(sessionId: string): void`
- `resolveInternalUserIdFromWebSession(session: UserSession): string`（名前は実装準拠で可）

---

## 7) ReplyService

- `createReplyDraft(internalUserId: string, replyTargetMessageText: string, emotionText: string): ReplyDraftResult`
  1. `UserContextService#getUserContext`
  2. **`ReplyDraftContextAssembler.assemble(internalUserId, context, texts) → ReplyDraftInput`**
  3. `ReplyDraftGenerationComponent#generateReplyDraft`
  4. `HistoryComponent#saveReplyDraftHistory`
- 送信など他ユースケースは Functional Design で同ファイルに追加。

---

## 8) ReplyDraftContextAssembler（協調モジュール）

- `assemble(userId: string, userContext: UserContext, replyTargetMessageText: string, emotionText: string): ReplyDraftInput`
  - **永続 `UserContext` → `ReplyDraftInput` の単一経路**。`UserContextComponent` がこの責務を持たない。

---

## 9) UserContextService

- `getUserContext(userId: string): UserContext`
- `upsertUserContext(userId: string, payload: UserContextPayload): UserContext`

---

## 10) ReplyDraftGenerationComponent（ファサード）

- `generateReplyDraft(input: ReplyDraftInput): ReplyDraftOutput`
  - `ReplyDraftPromptBuilder#build(input)` → `ReplyDraftLanguageModelClient#complete(...)` → `ReplyDraftOutputNormalizer#normalize(...)`

### 10a) ReplyDraftPromptBuilder

- `build(input: ReplyDraftInput): PromptInvocation`
  - コンピテンシー拘束／フォールバックの切替など **文言ルールはここに集約**

### 10b) ReplyDraftLanguageModelClient

- `complete(invocation: PromptInvocation): string`
  - Bedrock/API・リトライ・タイムアウト

### 10c) ReplyDraftOutputNormalizer

- `normalize(raw: string): NormalizedText`
- `validateOutputShape(text: string): ValidationResult`

---

## 11) HistoryComponent

- `saveReplyDraftHistory(record: ReplyDraftHistoryRecord): HistoryId`
- `saveSentReplyHistory(record: SentReplyHistoryRecord): HistoryId`
- `findSentRepliesByUser(userId: string, page: PageRequest): SentReplyHistoryPage`
  - **一覧は `HistoryQueryService` が呼ぶ**。ゲートウェイは直接呼ばない方針。

---

## 12) UserContextComponent

- `getContext(userId: string): UserContext`
- `upsertContext(userId: string, payload: UserContextPayload): UserContext`

---

## 13) PersistenceComponent

- `withinTransaction<T>(fn: () => T): T`
- `mapDbError(error: DbError): DomainError`

---

## 注記

- 本構成は **`application-design.md` §1.5（責務分離）** と整合させる。
- 詳細型名・未リンク時のユーザー向け文面文言は Functional Design で定義する。
