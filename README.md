# サクラクダ（仮）— AWS Hackathon 2026

文脈テキストと短い感情入力から、コンピテンシーに沿ったビジネス表現へ変換するプロダクトのリポジトリです。  
ドキュメントは **AI-DLC** の規約に沿って `aidlc-docs/` に集約しています。**アプリ本体のコードはワークスペース直下**（`aidlc-docs/` 外）に置く想定です。

## AGENDA（書類選考・レビュー用）


| 内容             | パス                                                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| 要件定義書          | [aidlc-docs/inception/requirements/requirements.md](aidlc-docs/inception/requirements/requirements.md)                         |
| 要件の静的ビュー（HTML） | [aidlc-docs/inception/requirements/requirements.html](aidlc-docs/inception/requirements/requirements.html)                     |
| ユーザーストーリー      | [aidlc-docs/inception/user-stories/stories.md](aidlc-docs/inception/user-stories/stories.md)                                   |
| ペルソナ           | [aidlc-docs/inception/user-stories/personas.md](aidlc-docs/inception/user-stories/personas.md)                                 |
| アプリケーション設計     | [aidlc-docs/inception/application-design/application-design.md](aidlc-docs/inception/application-design/application-design.md) |
| ユニット／実装単位      | [aidlc-docs/inception/application-design/unit-of-work.md](aidlc-docs/inception/application-design/unit-of-work.md)             |
| 実行計画（ワークフロー）   | [aidlc-docs/inception/plans/execution-plan.md](aidlc-docs/inception/plans/execution-plan.md)                                   |
| フェーズ進捗         | [aidlc-docs/aidlc-state.md](aidlc-docs/aidlc-state.md)                                                                         |


### 補助資料

- [コンピテンシー入力のデモ用サンプル](aidlc-docs/inception/requirements/competency-examples.md)（合成の例文。特定企業の転記ではありません）
- [監査ログ（AI-DLC）](aidlc-docs/audit.md)

## ディレクトリ構成（リポジトリ全体）

```text
<リポジトリルート>          … アプリケーションコード（予定・追加時はここ）
aidlc-docs/                 … AI-DLC ドキュメント一式（アプリコードは置かない）
.aidlc-rule-details/        … AI-DLC ルール詳細（ワークフロー用テンプレ）
CLAUDE.md                   … Cursor / AI 向けプロジェクトルール
README.md                   … 本ファイル
```

## aidlc-docs の構成（ディレクトリそのまま）

Inception フェーズの成果物を中心に置いています。AI-DLC の標準では `construction/` や `operations/` にも成果物が載りますが、**現時点では未作成のディレクトリもありえます**（追加されたらこのツリーを更新してください）。

```text
aidlc-docs/
├── aidlc-state.md              … フェーズ進捗・拡張設定（AI-DLC 状態）
├── audit.md                    … 監査ログ（プロンプト・承認の記録）
└── inception/
    ├── plans/                  … 計画・ワークフロー・ストーリー／ユニット計画
    ├── requirements/           … 要件定義・レビュー用ビュー・検証メモ
    ├── application-design/     … アプリケーション設計・コンポーネント・ユニット定義
    └── user-stories/           … ユーザーストーリー・ペルソナ
        ├── personas.md
        └── stories.md
```

### フォルダごとの役割（早見）

| パス（`aidlc-docs/` 以下） | 役割 |
|---------------------------|------|
| （直下）`aidlc-state.md` / `audit.md` | 進捗トラッキングと監査 |
| `inception/requirements/` | 要件本体・HTML ビュー・検証質問・コンピテンシー例・アセット |
| `inception/user-stories/` | ストーリーとペルソナ |
| `inception/application-design/` | 全体設計、コンポーネント／サービス、ユニットと依存・ストーリーマップ |
| `inception/plans/` | 実行計画、設計／ストーリー／ユニット各計画と評価メモ |

## 技術・インフラ（要件より）

- **AWS**（例: Bedrock、Lambda / EC2、RDS、Cognito）
- **実装言語**: Ruby（要件の記載どおり）
- **インターフェイス**: Slack アプリ + Web

## ライセンス・注意

ハッカソン提出用リポジトリです。内容・ライセンスはチーム方針に従ってください。