# Story Generation Plan

## Objective
要件定義を、実装と検証に使えるユーザーストーリーとペルソナへ変換する。

## Execution Checklist

### Part 1 - Planning
- [x] 1. User Stories実施要否を評価し、`user-stories-assessment.md`を作成する
- [x] 2. ストーリー生成アプローチの初期案を作る
- [x] 3. 文脈に応じた質問を作成する（本ドキュメント内の質問セクション）
- [x] 4. 必須成果物（`stories.md` / `personas.md`）を計画に組み込む
- [x] 5. ストーリー分解オプションと採用基準を定義する
- [x] 6. 計画を `aidlc-docs/inception/plans/story-generation-plan.md` に保存する
- [x] 7. ユーザーが全 [Answer]: を記入する
- [x] 8. 回答の欠落・矛盾・曖昧さを検証する
- [x] 9. 必要時、追質問ファイルを作成して曖昧さを解消する
- [ ] 10. 計画承認を取得する（明示承認が必須）

### Part 2 - Generation
- [ ] 11. 承認済み計画を読み込み、未完了タスクを特定する
- [ ] 12. `personas.md` を生成する
- [ ] 13. `stories.md` を生成する（INVEST + 受け入れ条件）
- [ ] 14. ストーリーとペルソナの対応関係を検証する
- [ ] 15. 本チェックリストの完了項目を [x] に更新する
- [ ] 16. User Stories完了メッセージを提示し、明示承認を取得する

## Story Breakdown Options

### A) User Journey-Based
- ユーザーの時系列行動に沿って分割
- MVPで体験フローを見せやすい

### B) Feature-Based
- 機能単位で分割
- 実装分担がしやすい

### C) Persona-Based
- ユーザー種別ごとに分割
- 価値訴求の違いを明確にしやすい

### D) Hybrid (Journey + Feature)
- 中核体験はJourney、補助機能はFeatureで分割
- ハッカソンMVPに適したバランス型

## Proposed Default (pending your answers)
- 現時点の提案: **D) Hybrid (Journey + Feature)**
- 理由: デモで体験価値を示しつつ、実装タスクへ落とし込みやすいため

## Clarification Questions

以下の各質問で、`[Answer]:` の後に選択肢を記入してください。  
どれも当てはまらない場合は、最後の `X) Other` を選び、補足を記入してください。

## Question 1
MVPの主ペルソナとして最優先で扱うユーザーはどれですか？

A) 「言いたいことはあるが、伝え方で評価を落としやすい」ビジネス職
B) 「1on1・自己評価で言語化に負荷が高い」若手社員
C) 「多忙/制約（育児・介護など）で言語調整コストを下げたい」社員
D) 「複数ロールを跨いで対人調整が多い」リーダー層
X) Other (please describe after [Answer]: tag below)

[Answer]: A ですが、ビジネス職ではなくエンジニア職です。

## Question 2
ストーリー分解の主軸はどれにしますか？

A) User Journey-Based
B) Feature-Based
C) Persona-Based
D) Hybrid (Journey + Feature)
X) Other (please describe after [Answer]: tag below)

[Answer]: D

## Question 3
ストーリー粒度はどれが良いですか？

A) デモ重視の粗め（5〜7本）
B) 標準（8〜12本）
C) 詳細（13本以上）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 4
受け入れ条件（Acceptance Criteria）の厳しさをどうしますか？

A) MVP最小限（主要正常系のみ）
B) 標準（正常系 + 主要エラー系）
C) 厳密（正常系 + エラー系 + 境界条件）
X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 5
FR-06（履歴最適化）とFR-07（依存体験設計）の扱いはMVPでどうしますか？

A) 両方MVPに含める
B) FR-06のみMVP、FR-07は将来
C) どちらも将来、今回は変換コアに集中
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6
要件には「成長を促さない」とありますが、質問票には「成長支援」記載もあります。MVP方針はどれですか？

A) 成長支援はしない（要件FR-07優先）
B) 軽い成長示唆のみ行う（明示的な育成機能はなし）
C) 成長支援機能をMVPに含める
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7
ハッカソンデモで重視する成功指標はどれですか？

A) 変換文の自然さ（自分の言葉として送れる）
B) 変換速度（10秒以内の安定応答）
C) 共感性（感情が汲み取られた実感）
D) 継続利用意向（もうこれなしでやりたくない）
X) Other (please describe after [Answer]: tag below)

[Answer]: X 全部

## Mandatory Artifacts to Generate After Approval
- `aidlc-docs/inception/user-stories/personas.md`
- `aidlc-docs/inception/user-stories/stories.md`