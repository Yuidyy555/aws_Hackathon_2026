# Unit of Work 計画（Units Generation）

## 計画の前提（設計書・実行計画に基づく既定回答）

以下は **要件・ユーザーストーリー・アプリケーション設計（承認済み）** から採用した分解方針である。追加の `[Answer]:` 未記入タグは置かない。

| 論点 | 採用 |
|------|------|
| デプロイ単位 | **単一 Ruby アプリケーション（モノリス）**。マイクロサービス分割は MVP 外。 |
| ユニットの意味 | **論理モジュール／実装順序**（独立デプロイ単位ではない）。 |
| コード配置 | ワークスペースルートにアプリコード（`aidlc-docs/` 外）。Ruby では `app/` 配下にサービス・コントローラ・モデルを配置する方針（詳細は CONSTRUCTION の code-generation に委譲）。 |
| ストーリー網羅 | US-01〜US-09 をユニットへ割当。US-09 は U4 の拡張として扱う。 |
| Slack / Web | 共通ドメインは **U4**、入口は **U5（Slack）／U6（Web）** に分離。 |

---

## Part 1 チェックリスト（計画）

- [x] システムを開発可能なユニットへ分解する方針を文書化した
- [x] 必須成果物パスを計画に含めた（`unit-of-work.md` 等）
- [x] ストーリーとユニットの対応方針を決めた
- [x] 依存関係（実装順）を検証した

---

## Part 2 チェックリスト（生成）

- [x] `aidlc-docs/inception/application-design/unit-of-work.md` を生成した
- [x] `aidlc-docs/inception/application-design/unit-of-work-dependency.md` を生成した
- [x] `aidlc-docs/inception/application-design/unit-of-work-story-map.md` を生成した
- [x] グリーンフィールド向けコード組織方針を `unit-of-work.md` に記載した
- [x] ユニット境界と依存を見直した
- [x] 全ストーリーにユニットを割り当てた

---

## 計画承認

ユーザーから **「Units Generationに進んでください」** の指示により、上記前提で Part 2（成果物生成）まで一括で実施する。

**次のアクション（AI-DLC Step 17）**: 成果物レビュー後、ユーザーがユニット分解を承認すると **CONSTRUCTION / Functional Design** へ進む。
