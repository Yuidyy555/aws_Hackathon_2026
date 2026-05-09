# サービス定義

## サービスレイヤー方針
- 入口（Slack/Web）は **HTTP／署名検証とルーティング**に徹し、アプリ入力の正規化・本人解決・ユースケースは **サービス層に集約**する
- **Slack 応答の体裁**はドメイン結果を **`SlackReplyPresentationMapper`** に渡して組み立てる（ビジネス判断は含めない）
- 変換処理は同期実行を基本とする（Q6=A）
- 履歴参照は一覧重視（Q3=A）で最小MVPを構成する
- 命名ルールは「送信前: `reply-draft` / 送信後: `reply`」で統一する

## 例外伝播のレイヤ規約（読み取り用）
- **Domain / Service**: 検査可能な業務エラー（未リンク等は `LinkedIdentityResolverService` の失敗として型付きで返す）。
- **Gateway**: 上記を **HTTP／Slack メッセージ**に写像。監視向けメタは **`ErrorHandlingService`** で付与する。

## サービス一覧

### 1. ReplyService
- **責務**:
  - 返信文ドラフト生成・返信送信ユースケースの **チャネル中立オーケストレーション**
  - `UserContextService#getUserContext` → **`ReplyDraftContextAssembler`（協調・純関数モジュールで可）で `ReplyDraftInput` 組み立て** → **`ReplyDraftGenerationComponent#generateReplyDraft`** → `HistoryComponent#saveReplyDraftHistory`
  - ユーザー向けエラーと監視向けエラー情報の分離（上流へ投げるだけ。Slack 体裁はゲートウェイ／Mapper へ）
- **主要連携**:
  - `UserContextService`
  - **`ReplyDraftContextAssembler`（本サービスからのみ利用＝永続 `UserContext` を LLM向け入力へ写像）**
  - `ReplyDraftGenerationComponent`
  - `HistoryComponent`

### 2. LinkedIdentityResolverService
- **責務**:
  - **`slack team_id` / `user_id` から `internalUserId` を解決**する（永続リンクの参照・未リンク時は失敗結果のみ返す）
  - **Slack 特有の「誰が操作しているか」の唯一の解決口**（`SlackInteractionService` は本サービスへ委譲する）
- **非責務**: Slash の引数解釈、`ReplyService` 呼び出し
- **主要連携**:
  - **`PersistenceComponent`**（リンクテーブル／リポジトリ）

### 3. SlackInteractionService
- **責務**:
  - Slash `payload` から **`NormalizedReplyDraftInputs`（返信対象文・感情入力）** への正規化と必須チェック
  - **`LinkedIdentityResolverService` へ `internalUserId` 解決を委譲**
  - 解決済みなら **`ReplyService#createReplyDraft`**（および送信ユースケース）へ **チャネル中立の引数**で転送
  - **UserContext の読み書きは実施しない**
- **非責務**: Slack メッセージの Block 組み立て（**`SlackReplyPresentationMapper`** へ）
- **主要連携**:
  - `SlackGatewayComponent`（呼び出し元）
  - `LinkedIdentityResolverService`
  - `ReplyService`

### 4. HistoryQueryService
- **責務**:
  - 送信済み返信履歴一覧取得（日時順）
  - ページング処理、表示用DTOへの整形
  - **一覧取得の読みクエリはここを経由する**（ゲートウェイ → 本サービス → `HistoryComponent`）
- **主要連携**:
  - `HistoryComponent`

### 5. AuthService
- **責務**:
  - OAuthログインフロー管理
  - セッションライフサイクル管理
  - 認証状態検証
  - **Web 経路の `internalUserId` 確定**（ゲートウェイは本サービスまたは `AuthComponent` へ委譲）
- **主要連携**:
  - `AuthComponent`

### 6. UserContextService
- **責務**:
  - **`competencyText` を含む**コンピテンシー/自己コンテキストの取得・更新のオーケストレーション
  - **Web の `PUT /web/user-context`** は本サービス経由で `UserContextComponent` に永続化
  - **`ReplyService` の生成前読取**と **Web の GET** からの利用
  - **プロンプト向けの Assembler は持たない**（`ReplyDraftContextAssembler` は `ReplyService` 側）
- **主要連携**:
  - `UserContextComponent`

### 7. ErrorHandlingService
- **責務**:
  - 例外分類（ユーザー提示可否）
  - 監視ログ/トレース出力
  - APIエラーコード標準化
- **主要連携**:
  - すべてのGateway/Service（横断）

## 協調モジュール（サービスと同階層の命名で可・DBに直接触れない）

### ReplyDraftContextAssembler
- **責務**: 永続済み `UserContext` と今回の入力文面から **`ReplyDraftInput` を組み立てる**（生成パイプライン用 DTO）
- **所在**: `ReplyService` と同パッケージ／内部モジュール想定。単体テスト可能な純粋ロジック優先。

## オーケストレーションパターン

### 返信文ドラフト生成フロー（同期）
1. 入口: **Web** はゲートウェイで入力形式検証 → `ReplyService`。**Slack** は `SlackGateway` が署名検証 → `SlackInteractionService` が **正規化＋`LinkedIdentityResolverService`** → `ReplyService`。
2. `ReplyService` が `UserContextService#getUserContext(internalUserId)`
3. **`ReplyDraftContextAssembler` が `ReplyDraftInput` を構築**
4. `ReplyDraftGenerationComponent`（内包: PromptBuilder → LanguageModelClient → OutputNormalizer）がドラフト生成
5. `HistoryComponent#saveReplyDraftHistory`
6. **Slack のみ**: 戻り値を `SlackReplyPresentationMapper` → `SlackGateway` 応答。**Web** は JSON 応答整形。

### コンピテンシー登録フロー（Web・任意、`PUT /web/user-context`）
1. `WebGatewayComponent` が入力形式検証
2. `UserContextService` が `UserContextComponent` 経由で永続化
3. 保存結果を Web へ返却

### 返信送信フロー（同期）
1. ユーザーが返信文ドラフトを確定
2. `ReplyService` が送信処理を実行
3. `HistoryComponent` で送信済み返信履歴保存
4. 送信結果を返却（Slack 側は再び Presentation 写像の対象となり得る）

### 返信履歴一覧フロー
1. `WebGateway` が認証済み `internalUserId` を確保
2. `HistoryQueryService` が `HistoryComponent` で取得
3. 一覧DTOを返却
