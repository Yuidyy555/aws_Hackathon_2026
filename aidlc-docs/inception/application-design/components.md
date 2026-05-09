# コンポーネント定義

## 設計方針
- 分割方針は **チャネル別優先**（Q1=A）
- 共通変換ロジックは同期APIで提供（Q6=A）
- OAuthはCognito Hosted UIを採用（Q4=A）
- エラー処理はユーザー表示と監視情報のバランス重視（Q7=C）
- **責務分離**: 「HTTP／Slack署名」「チャネル非依存のアプリ入力」「本人解決」「ドメインユースケース」「Slack メッセージ体裁」「プロンプト／LLM／整形」を混在させない（`application-design.md` §1.5 と整合）。

## コンポーネント一覧

### 1. SlackGatewayComponent
- **目的**: Slack Slash Command を **HTTPS／署名検証**レベルで受け、アプリサービスへ委譲したうえで **応答のみ Slack 体裁へ写像**する
- **責務（境界）**:
  - Slack 署名検証・リクエスト生データの受理
  - **`SlackInteractionService` へ委譲**（Slash の正規化・本人解決は含めない）
  - **`SlackReplyPresentationMapper` を用いた**ドメイン結果 → Slash 応答形式（Block Kit／テキスト等）への変換
  - **ユーザー向けの Slack 応答体裁**のみ本層または `SlackReplyPresentationMapper` が担う。**アプリ入力の検証／ビジネスルール**は `SlackInteractionService` 以下が責務。
- **非責務**: `replyTargetMessageText` / `emotionText` の意味的正規化、必須チェックの本体、`internalUserId` 解決、LLM／履歴処理。

### 2. SlackReplyPresentationMapperComponent
- **目的**: **`ReplyDraftResult` のようなチャネル中立な結果**だけを入力とし **Slack クライアント向け応答オブジェクト**に変換する
- **責務**:
  - 成功レスポンス／エラー案内レスポンス（未リンクユーザー等、`SlackInteractionService` が返した区分に従う）の Slack メッセージ構築
  - メッセージ長・Slack上限に合わせた整形（チャネル都合のみ）
- **非責務**: OAuth、ユーザー解決、アプリ入力バリデーション、ドメインロジック

### 3. WebGatewayComponent
- **目的**: Web アプリから **HTTP 入力の受理・共通バリデーション・ルーティング**のみ行う。**業務ユースケース本体はサービスへ委譲**する。
- **責務（論理グループ）**:
  - **コンテキスト系**: `GET/PUT /web/user-context` → **`UserContextService`**（コンピテンシー・自己コンテキストの登録／取得は共通ドメインモデル）。
  - **返信ドラフト／送信**: `POST /web/reply-drafts`、`POST /web/replies` → **`ReplyService`**（HTTP 入力の必須・形式チェックのみ。この層で `UserContext` を直接読み書きしない）。
  - **履歴一覧**: `GET /web/replies/history` → **`HistoryQueryService`**。
  - セッションから **`internalUserId`** を **`AuthComponent`/`AuthService` に委ねて確定**（ゲートウェイ内でドメインロックインをしない）。
- **分割の指針（実装時）**: Ruby などでは **コントローラーをルート単位（例: UserContext／ReplyDraft／SentHistory）でファイル分割**しても、この設計書上のひとつの「Web gateway 群」とみなしてよい。

### 4. AuthComponent
- **目的**: OAuth認証と Web 経路でのユーザー同定を提供する
- **責務**:
  - Cognito Hosted UIフロー開始/コールバック処理
  - セッション発行/検証
  - **Web 経路での**未認証アクセスの遮断
  - **`sub` 等とアプリ内 `internalUserId` の対応確立（Web）**
- **非責務**: **`Slack workspace user` ↔ `internalUserId`** のリンク解決は **`LinkedIdentityResolverService`**（Slack メンバー用）
- **公開インターフェース**:
  - `GET /auth/login`
  - `GET /auth/callback`
  - `POST /auth/logout`

### 5. ReplyDraftGenerationComponent（生成パイプラインのファサード）
- **目的**: 「プロンプト組み立て → LLM 呼び出し → 出力整形」の **順序のみ** をオーケストレーションする。**入力 `ReplyDraftInput` は上流（`ReplyService`＋ Assembler）から渡されたものを使う**。
- **責務**:
  - `ReplyDraftPromptBuilder` と `ReplyDraftLanguageModelClient` と `ReplyDraftOutputNormalizer` の呼び順の固定
  - 全体の単一公開入口 `generateReplyDraft(input): ReplyDraftResult`（名前は実装と合わせてよい）
- **協調オブジェクト（同一論理コンポーネント群／別モジュール化を推奨）**:
  - **`ReplyDraftPromptBuilder`**: `ReplyDraftInput` からプロンプト断片・システム指示・モデル設定の組み立て（**コンピテンシー拘束／フォールバックのルールもここに閉じる**）
  - **`ReplyDraftLanguageModelClient`**: Bedrock／LLM API 呼び出し・リトライ／タイムアウトなど **インフラ都合のみ**
  - **`ReplyDraftOutputNormalizer`**: 文体・長さガード・空文字除外などの **後処理のみ**
- **非責務**: **`UserContext` の永続化・読取**、`History` 保存、`Slack`/HTTP の体裁

### 6. HistoryComponent
- **目的**: 変換履歴の **永続化と読クエリ実装の提供**
- **責務**:
  - `saveReplyDraftHistory` / `saveSentReplyHistory`（書込）
  - **`findSentRepliesByUser` 等の読クエリ実装（集約クエリ）は `HistoryQueryService` が利用する**。ゲートウェイが直接本コンポーネントの読APIを迂回しない運用とする。
- **公開インターフェース**:
  - `saveReplyDraftHistory(historyRecord): HistoryId`
  - `saveSentReplyHistory(historyRecord): HistoryId`
  - `listSentRepliesByUser(userId, pagination): SentReplyHistoryList`

### 7. UserContextComponent
- **目的**: **`internalUserId` に紐づくコンテキスト（コンピテンシー／自己コンテキスト等）を永続層とのみやりとりする**
- **責務**:
  - `getContext(userId)` / `upsertContext(userId, payload)` で **ドメインモデル読み書き**
- **非責務**: 「LLM が読みやすい形」の組み立て・プロンプト用テンプレ適用。**それは `ReplyDraftContextAssembler`（`ReplyService` 内協調オブジェクト／モジュール）の責務**とする
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
  - Slack メンバーと `internalUserId` を結ぶリンク用リポジトリ（命名は Functional Design で確定）
