# Order: self-review round-N（実装後 差分レビュー / step 7）

> オーケストレーター → レビューセッション。実装*後*の累積差分を多観点で見る。
> 別セッション推奨（実装文脈を引きずらない別の目）。ultracode を使う場合は
> 観点ごとに並列サブエージェントで fan-out する。

## 対象差分
- `merge-base(main, HEAD)..HEAD` の累積差分（ファイル単体でなく全体）
- issue: #<issue> / 受け入れ条件: approach.md・各 phase report.md 参照

## 観点
コア + 課題に応じて増減。対象の言語/フレームワークは diff から判定すること（skill は言語非依存）。

1. correctness — 正しく動くか、エッジ/境界条件
2. 受け入れ条件の充足 — issue/approach の条件を満たすか
3. regressions — 既存挙動を壊していないか
4. simplicity / 重複 / altitude — 過剰・重複・抽象レベルのズレ
5. 言語/フレームワークのイディオム — その言語/FW の流儀・repo の既存流儀に沿う組み方か
6. security — 入力検証・権限・秘匿情報・インジェクション等
7. performance — 不要な再計算/再取得・N+1・重い同期処理等

## ultracode fan-out（ユーザーが ultracode を使う前提のとき）
- 観点ごとに並列サブエージェント → 各 finding を構造化
  （dimension / severity / file:line / 根拠 / 修正案）
- blocker/major は敵対的検証（refute させる）で誤検知を除去
- dedup・優先度付け → 統合 report + verdict
- 実機で未検証の項目は明示（黙って省かない）

## 出力（review/round-N/report.md）
- finding 一覧（上記の構造）＋ verdict（このラウンドで clean か / 要修正か）
- clean なら「指摘なし」。nit は潰しに行かない。

## 注意（billing）
- ultracode/Workflow は**ユーザートリガー**。オーケストレーターは order を書くだけで
  起動しない。「このパスを新規セッションで開いて ultracode で実行して」と促す。
