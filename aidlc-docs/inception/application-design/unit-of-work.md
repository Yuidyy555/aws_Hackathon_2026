# Unit of Work 定義（MVP）

## スコープ宣言

- **デプロイ境界**: 単一アプリケーション（**モノリス**）。ハッカソン MVP として **1 リポジトリ・1 プロセス**を想定する。
- **本書のユニット**: 実装・テストを並行しにくい **依存順に並べた論理モジュール**。各ユニットは `components.md` / `services.md` の境界と整合させる。

## コード組織方針（グリーンフィールド）

- アプリケーション本体は **`aidlc-docs/` 以外のワークスペースルート**に置く（AI-DLC ディレクトリ規則）。
- 推奨の論理マッピング（実装時にリネーム可）:
  - **U1**: `db/migrate`, 永続化アダプタ（Repository）、`Persistence` に相当する層
  - **U2**: 認証・Slack リンク（OAuth コールバック、`LinkedIdentityResolver`、リンクテーブル操作）
  - **U3**: `UserContext` 読み書きサービスと Web のコンテキスト API
  - **U4**: `ReplyService`、`ReplyDraftContextAssembler`、ドラフト生成パイプライン（Prompt / LLM / Normalizer）、履歴の書込を伴う返信ユースケース
  - **U5**: Slack 署名、`SlackInteractionService`、`SlackReplyPresentationMapper`、Slash ルート
  - **U6**: Web の返信ドラフト／送信／履歴一覧ルート、セッション連携、UI 都合の入出力（US-01,03,07 の HTTP 側）

## ユニット一覧

### U1 — Persistence & schema foundation

- **責務**: RDB スキーマ（ユーザー、`user_contexts`、Slack リンク、ドラフト／送信履歴等）、`PersistenceComponent` に相当する **DB アクセスの単一窓口**、トランザクション境界の基盤。
- **主要アウトプット**: マイグレーション、リポジトリ／ORM 境界、他ユニットが依存する **ID 型と永続モデル**。
- **依存**: 外部（PostgreSQL 等はインフラ設計で確定）。

### U2 — Auth & linked identity

- **責務**: Cognito Hosted UI 連携（`AuthComponent`/`AuthService`）、Web セッションと **`internalUserId`**。**`LinkedIdentityResolverService`** と Slack `team_id`/`user_id` ↔ `internalUserId` の永続リンク。
- **主要アウトプット**: ログイン／コールバック／ログアウト、リンク未達時の失敗型（Slack 経路で利用）。
- **依存**: **U1**（ユーザー／リンクテーブル）。

### U3 — User context module

- **責務**: `UserContextService` / `UserContextComponent`、`GET/PUT /web/user-context`。コンピテンシー任意・共有ストア。
- **主要アウトプット**: コンテキスト API、バリデーション（長さ等は Functional Design で詳細化）。
- **依存**: **U1**, **U2**（Web 経路の認証済みユーザー解決）。

### U4 — Reply orchestration & draft generation core

- **責務**: **`ReplyService`**（文脈取得 → **`ReplyDraftContextAssembler`** → **`ReplyDraftGeneration`**（PromptBuilder / LanguageModelClient / OutputNormalizer）→ **`HistoryComponent`** 書込）。返信送信ユースケースのオーケストレーション。**チャネル非依存**。
- **主要アウトプット**: ドメイン結果 DTO、履歴へのドラフト／送信記録、**US-02 / US-05 / US-06** の中核、**US-09** の拡張ポイント（履歴参照生成はここに載せる）。
- **依存**: **U1**, **U3**（読取）。**U2** は呼び出し側が `internalUserId` を渡す前提のため、U4 本体のコード依存は最小（結合テストで U2 と接続）。

### U5 — Slack channel adapter

- **責務**: **`SlackGatewayComponent`**（署名・委譲）、**`SlackInteractionService`**（Slash 正規化・`ReplyService` 転送）、**`SlackReplyPresentationMapper`**（outcome → Slash 応答）。**`LinkedIdentityResolverService` の呼び出し**（実装は U2、ここはオーケストレーション上の利用）。
- **主要アウトプット**: Slash エンドポイント、Slack エラー表示、**US-04**。
- **依存**: **U2**, **U4**。

### U6 — Web channel adapter

- **責務**: **`WebGateway`** 相当の HTTP ルート（コンテキストは U3 へ委譲済みの経路と、返信ドラフト／送信／**`HistoryQueryService`** 経由の一覧）。入力検証と JSON 応答。
- **主要アウトプット**: **US-01, US-03, US-07** の Web 面、**US-08** の保護されたルート。
- **依存**: **U2**, **U3**, **U4**（`HistoryQueryService` は U4 と同じ永続を読むため U1 経由で間接依存）。

### U7 — Cross-cutting concerns（横断）

- **責務**: **`ErrorHandlingService`**、構造化ログ、（必要なら）相関 ID。全ゲートウェイ／サービスから利用。
- **主要アウトプット**: 例外→ユーザー向け／監視向けの一貫したポリシー（`services.md` のレイヤ規約と整合）。
- **依存**: 実装は各層にフック。**論理依存は最後に整備**してもよいが、**U5/U6 の外部公開前に最低限そろえる**ことを推奨。

## 推奨実装順

`U1` → `U2` → `U3` → `U4` →（`U7` を並行可能）→ `U5` と `U6`（相互独立、**U4 完了後**に着手）。

## 検証

- 各ユニットは **単体テスト可能な境界**（モックは隣接ユニットのインタフェース）を持つこと。
- **E2E** は U5/U6 が揃った後に、少数のクリティカルパス（ログイン → コンテキスト PUT → ドラフト POST 等）でよい。
