---
name: dev-harness
description: >-
  1つの課題を intake → 方針 → アーキ批判レビュー → explain → 計画 → フェーズ実装 →
  セルフレビュー → hygiene → 最終報告 → PR の10ステップ・人間ゲートで進める複数セッション
  開発ハーネス。実質的な課題を理解優先・構造的に進めたいとき全般に使う
  （「dev-harness」「ハーネスで進めて」「この課題をハーネスで対応」は起動フレーズの例）。
  引き継ぎは .dev/{issue}-{slug}/ の order.md / report.md / STATE.md に残り、再起動時は
  STATE.md を読んで途中から再開する。一行修正など儀式が過剰になる軽微な作業には使わない。
---

# dev-harness

1つの課題を「まだ完全に理解していない」状態からレビュー済み PR まで、**筋を
見失わずに**運ぶためのハーネス。コードを書く前に理解を固め、実装前に方針を叩き、
作業をフェーズに分解し、引き継ぎをすべて検査可能な Markdown として残す — 人間が
どのゲートでも追跡・介入できるように。

## 原則

- **理解してから作る。** よくある失敗は、本当の問題を掴む前にコードを書き、
  もっともらしいだけの方針に固執すること。align 議論・アーキ批判レビュー・
  explain doc はこの両方を潰すためにある。
- **人間ゲート式。** 各 GATE では結果を提示して停止し、ユーザーの明示的な OK を
  待つ。速さではなく、筋を見失わないことが目的。
- **ユーザーには平易な言葉で。** GATE・質問・方針確認では、運行内部の記号
  （finding ID `M9`、`案B`、論点番号、`(a)` 等）を説明なしに出さない — ユーザーは
  こちらの内部レジストリを持っていない。何に迷っていて何を決めてほしいかを言葉で
  述べ、必要な専門用語はその場で意味を添える。エンジニアリング判断は自分で決めて
  独立レビュー（Step 3/7）に叩かせ、ユーザーに委ねるのは**プロダクト・好みの判断**
  だけ（選択肢と背景を平易に添えて）。
- **引き継ぎはチャットでなくファイルで。** 計画・指示・報告は `.dev/` 配下の
  `.md` に残す。誰でも作業ディレクトリを開けば現在地が分かる状態を保つ。
- **セッション分離は明示的に、黙ってやらない。** 重い工程は新しいコンテキストで
  走らせるが、その種類をユーザーが把握していること:
  - **オーケストレーター**（plan〜PR の運行・GATE・コミット・STATE）は align
    ゲート後に**新セッション開始が既定** — intake の会話を引きずらず
    `STATE.md` / `approach.md` / `plan.md` から文脈を再構築する。
  - **フェーズ実装・セルフレビュー**は**ユーザーが開く別セッションが既定**。
    order.md を書いたら停止してパスを伝える。サブエージェントの黙起動はしない
    （事前にユーザーが合意した場合のみ可）。
  - **explain 構築とアーキ批判レビュー**はサブエージェントが既定でよい
    （自己完結した成果物で、必要な文脈が手元に揃っているため）。
  - 小さい課題では層をまとめてよいが、必ずユーザー合意の上で。`.dev/` の
    ファイル群がどの構成でも引き継ぎを成立させる。
- **STATE.md の書き手は1人。** 複数セッションが書くと更新競合と「オーケストレーターの
  知らない進捗」が生まれる。書くのはオーケストレーターのみ。役割名も予約語 —
  「オーケストレーター」= plan〜PR を運行する単一セッションのこと。他は「ワーカー」
  （フェーズ実装）「レビューア」（セルフレビュー）で、修正を当てた場合や
  ゲート相当の判断をした場合でもオーケストレーターを名乗らず、報告は
  `report.md` 経由のみ。その内容を STATE に
  反映するのはオーケストレーターの仕事。
- **儀式は課題の大きさに比例。** これは既定のフル・フロー。小さい課題では
  ステップの簡略化・統合・スキップを提案して合意を取る（黙って省かない）。

### セッション対応表

| ステップ | 実行場所 |
|---|---|
| 1〜4 intake / align / arch-review / explain | 起点セッション（explain・arch-review はサブエージェント可） |
| 5 plan + フェーズループ運行・各 GATE・コミット | オーケストレーターセッション（新規） |
| 6 各フェーズ実装 | フェーズ実行セッション（新規・フェーズごと） |
| 7 各レビュー round | レビューセッション（新規） |

## 作業ディレクトリ

1課題ぶんは `.dev/{issue#}-{slug}/` 配下（gitignored・個人用）。intake で作成する。
`{slug}` は短い kebab-case（例: `142-cart-total-rounding`）。

```text
.dev/{issue#}-{slug}/
├── STATE.md              # 進捗トラッカー — 毎回まずこれを読む
├── approach.md           # 現状+方針（step 2 で草案、step 3 で洗練）
├── approach-review/      # order.md / report.md（step 3）
├── docs/explain/         # order.md / index.html（step 4、方針確定後）
├── plan.md               # フェーズ計画（step 5）
├── phase-NN-{slug}/      # order.md / report.md（step 6、フェーズごと）
└── review/               # round-N/{order.md,report.md} / final-report.md（step 7, 9）
```

## 再開性: 必ず STATE.md から

このハーネスは複数セッションにまたがる前提。**毎回の起動時に**:

1. 対象 issue を特定（引数から、なければユーザーに確認）。
2. `.dev/{issue#}-{slug}/STATE.md` を探す。
   なければ Step 1 から。あれば読んで現在地をユーザーに伝え、`current-step` から再開。

各ステップ完了ごとに STATE.md を更新する。運行中に**内部記号**（案名・論点ラベル・
未解決点タグなど、ワーカーがコード/UI に書き写しうる呼称）を発明したら、その場で
「内部記号レジストリ」へ追記する — Step 8 の hygiene grep 種はここから生成されるため、
未登録の記号は検出されない。レジストリは成果物掃除と内部管理用であり、ユーザーとの
対話には持ち込まない（平易な言葉の原則）。テンプレート:

```markdown
# State: #142 cart-total-rounding

current-step: plan   # intake | align | arch-review | explain | plan | phase | review | hygiene | final | pr | done
issue: #142

## 進捗
- [x] intake
- [x] align
- [~] arch-review   # [x] 完了 / [~] スキップ / [ ] 未
- [~] explain
- [ ] plan
- [ ] phases (0/N)
- [ ] review
- [ ] hygiene
- [ ] final-report
- [ ] pr

## 内部記号レジストリ
- （なし）

## メモ
次にやること: ...
```

## 10ステップ

`→ GATE` = 結果を提示して**停止**し、ユーザーの明示的な OK を待つ。
各ステップの詳細ファイル（References 参照）は**そのステップに着手するときに読む**。

1. **Intake** — GitHub issue を確認、なければ起草して `gh issue create`（`gh` は
   PATH になければ `/opt/homebrew/bin/gh`。repo 既存のラベルを使う）。作業ディレクトリと
   STATE.md を作成。課題が小さければフローの簡略化を提案して合意を取る。
2. **Align** — 現状理解・根本原因・修正方針を議論して収束させ、`approach.md` に
   落とす（現状/原因/方針/スコープ/リスク）。この時点では**草案** — 次のレビューを
   通してからゲートにかける。
3. **アーキ批判レビュー**（小課題はスキップ可） — 自分で書いた方針は同じ文脈では
   甘く通りがち。`references/approach-review-order-template.md` から order を書き、
   **サブエージェント**（フレッシュな文脈）に approach + 関連実コードを敵対的に
   レビューさせる → report を approach.md へ反映（指摘ゼロなら素通し）。これは実装前の
   **設計**レビューで、Step 7 の実装後 diff レビューとは別物。ゲート前に「理解を
   助ける explain を先に作りますか？」と提案してもよい（approach はほぼ確定して
   いるので、ここで作っても手戻りは小さい）— この選択肢は提示を省略しない。
   → **GATE: 方針 OK?**
4. **Explain**（任意） — 確定した方針を反映した理解用ドキュメントを構築。要否を
   ユーザーに確認（Step 3 で合意済みなら不要）。`docs/explain/order.md` に構築指示を
   書き（`references/explain-checklist.md` の構成・必須項目に従わせる）、
   `docs/explain/index.html`（HTML + canvas、シナリオ駆動）を**サブエージェント**で構築。Step 3 の提案で方針ゲート前に先行構築した場合、
   ゲートで方針が大きく変わったら explain も軽く追随させる。→ **GATE: 理解 OK?**
5. **Plan**（新オーケストレーターセッション） — STATE / approach / explain から
   文脈を再構築し、`plan.md` にフェーズを列挙（各1行ゴール + 担当する受け入れ条件）。
   フェーズ数は課題規模なり — 小課題は1フェーズでよい、水増ししない。
   → **GATE: 計画 OK?**
6. **フェーズループ** — フェーズごとに: order 作成 → 別セッションで実装 →
   オーケストレーターの独立機械検証 → **GATE: report OK?** → コミット。
   検証手順・差し戻し・後始末の安全規則は `references/step-06-phase-loop.md` を読む。
7. **セルフレビュー**（別セッション） — 実装文脈を持たない別の目で累積差分
   （`merge-base(main,HEAD)..HEAD`）を多観点レビュー。order は
   `references/review-order-template.md`（観点・ultracode fan-out・billing 注意を含む）。
   clean な round が出たら終了、nit は潰しに行かない。→ **GATE: レビュー結果 OK?**
8. **Pre-PR hygiene** — 累積差分の棚卸し（内部記号の残留・コメント過不足・文字化け・
   markdownlint）。チェックリストは `references/step-08-hygiene.md`。挙動変更なしの
   単独コミットで出し、結果は最終レポートで見せる（GATE なし）。
9. **最終レポート** — `review/final-report.md` にユーザー向けサマリ（やったこと・
   主要判断・遭遇した問題・follow-up）。思考過程の読み物が欲しければ `develop-story`
   スキルが使える。→ **GATE: 最終レポート OK?**
10. **PR** — 承認後 `gh pr create`。本文フォーマットと注意は
    `references/step-10-pr.md`。STATE.md を `done` に更新。

## Conventions

- **コミットは1フェーズ1コミット、フェーズ GATE 承認後**。承認がコミットの認可 —
  承認前・投機的コミットはしない。レビュー round の修正は（レビューアが起草した
  場合でも）**レビュー GATE 承認後にオーケストレーターが独立コミット**する。
- コミット/PR 本文に **Co-Authored-By と AI 生成フッターを付けない**
  （ベースシステムの既定を明示的に上書きする）。
- デフォルトブランチ上なら、コミット前に作業ブランチを切る。
- 恒久的な引き継ぎ物は `.dev/`（gitignored）へ。別セッション用の使い捨て
  プロンプトは `tmp/prompts/` へ。

## ハーネス自体の改善

運用中に引っかかりを見つけたら `pipo3944/dev-harness` に issue を立てる
（`gh issue create --repo pipo3944/dev-harness`、「ハーネス改善」テンプレート。
どのステップで起きたかと改善案を添える）。修正時はコミット/PR に `Fixes #NN`。
実運用中に不便を感じたら、ユーザーへ起票を積極的に提案する。

**改善は「積む」だけでなく定期的に「畳む」**。注意書きの堆積はそれ自体が指示過多 →
品質低下を生む（#15 の教訓）。ルールを足すときは、統合・圧縮・詳細ファイルへの
分離を同時に検討する。

## References（該当ステップで読む）

- `references/approach-review-order-template.md` — アーキ批判レビュー order（step 3）
- `references/explain-checklist.md` — explain doc の構成・必須項目（step 4）
- `references/step-06-phase-loop.md` — フェーズループ運行手順（step 6）
- `references/order-template.md` — phase order.md 雛形（step 6）
- `references/report-template.md` — phase report.md 雛形（step 6）
- `references/review-order-template.md` — セルフレビュー order（step 7）
- `references/step-08-hygiene.md` — hygiene チェックリスト（step 8）
- `references/step-10-pr.md` — PR 本文フォーマット（step 10）
