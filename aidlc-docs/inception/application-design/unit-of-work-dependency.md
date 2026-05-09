# Unit of Work 依存関係

## 方針

- 依存は **実装・結合テストの順序**を表す（モノリス内のモジュール間）。
- **U4** がドメイン中核。**U5/U6** はアダプタであり **U4 に向かう一方向**（U4 は Slack/Web を知らない）。

## 依存マトリクス

| From \ To | U1 | U2 | U3 | U4 | U5 | U6 | U7 |
|-----------|----|----|----|----|----|----|-----|
| **U1** | — | | | | | | |
| **U2** | ✓ | — | | | | | |
| **U3** | ✓ | ✓ | — | | | | |
| **U4** | ✓ | ○ | ✓ | — | | | ○ |
| **U5** | ✓ | ✓ | ○ | ✓ | — | | ○ |
| **U6** | ✓ | ✓ | ✓ | ✓ | | — | ○ |
| **U7** | ○ | ○ | ○ | ○ | ○ | ○ | — |

凡例: **✓** = 強い依存（先に実装または同時に契約が必要）。**○** = 任意または薄い依存（ログ横断など後追い可）。

### U4 の U2 が ○ の理由

`ReplyService` は **`internalUserId` だけ**を受け取る。ID の発行は **U2/U6** 側。ユニット単体テストでは `internalUserId` を固定値で注入可能。

### U5 の U3 が ○ の理由

Slack 経路は **`UserContext` を直接編集しない**。読取は **U4 → UserContextService** 経由。U5 は U3 をコード import しない想定。

## レイヤー対応（参考）

| ユニット | 主に該当する設計要素 |
|----------|----------------------|
| U1 | `PersistenceComponent`、全 Repository |
| U2 | `AuthComponent`/`AuthService`、`LinkedIdentityResolverService` |
| U3 | `UserContextService`、`UserContextComponent`、Web コンテキストルート |
| U4 | `ReplyService`、`ReplyDraftContextAssembler`、`ReplyDraftGeneration` 群、`HistoryComponent`（書込）、`UserContextService`（読） |
| U5 | `SlackGatewayComponent`、`SlackInteractionService`、`SlackReplyPresentationMapper` |
| U6 | `WebGateway`（返信・履歴）、`HistoryQueryService` |
| U7 | `ErrorHandlingService`、ロガー |

## テキスト代替（DAG）

1. **U1** が最下流（スキーマ・永続）
2. **U2** は U1 の上（認証・リンク）
3. **U3** は U1+U2 の上（Web コンテキスト）
4. **U4** は U1+U3（+実行時は U2 が提供する ID）
5. **U5** と **U6** はいずれも **U4** の上に並列
6. **U7** は全層へ横断（最後に揃えても可）
