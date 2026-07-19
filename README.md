# dev-harness

1つの課題を「まだよく分かってない」状態から、レビュー済み PR まで **筋を見失わずに** 運ぶための [Claude Code](https://claude.com/claude-code) スキル。

コードを書く *前* に理解を固め、実装する *前* に方針を叩き、作業をフェーズに分解し、各引き継ぎを検査可能な Markdown として残す — 人間がいつでも追跡・介入できることを狙っています。

## 特徴

- **理解が先、実装が後** — align 議論・アーキ批判レビュー・explain ドキュメントで、早すぎる実装と「もっともらしいだけの方針」を潰す
- **人間ゲート式** — 各 GATE で止まり、ユーザーの OK を待つ。速さではなく筋を見失わないことが目的
- **チャットではなく成果物で引き継ぎ** — 計画・指示・レポートは `.dev/` 配下の `.md` として残る
- **セッション分離は明示的** — 重い工程は新規セッションで走らせるが、勝手に subagent を spawn しない
- **儀式は課題規模に比例** — 小さい課題はステップの統合・省略を提案してよい
- **再開可能** — `.dev/{issue}-{slug}/STATE.md` を読んで途中から再開

## インストール

Claude Code のユーザースキルとして `~/.claude/skills/` 配下に置きます。

```bash
git clone git@github.com:pipo3944/dev-harness.git ~/.claude/skills/dev-harness
```

配置後、Claude Code から `dev-harness` スキルとして認識されます。

## 使い方

新しいセッションで、次のように起動します。

```
/dev-harness {issue番号}
```

または会話の中で「この課題をハーネスで進めて」のように依頼するだけでも起動します。

再度同じ課題で起動すると `STATE.md` を読んで途中から再開します。

## 10ステップの流れ

`→ GATE` は「結果を提示して、ユーザーの明示的な OK を待つ」ポイントです。

| # | ステップ | 概要 |
|---|---|---|
| 1 | Intake | GitHub Issue を確認/作成、作業ディレクトリと `STATE.md` を用意 |
| 2 | Align | 現状・根本原因・修正方針を固め `approach.md` に草案 |
| 3 | アーキ批判レビュー | 独立した subagent が方針を批判的にレビュー（小課題はスキップ可） |
| 4 | Explain | 確定方針を HTML+canvas で可視化（任意） |
| 5 | Plan | 新規オーケストレーターセッションで `plan.md`（フェーズ計画） |
| 6 | Phase loop | フェーズごとに order → 実装 → 機械的検証 → GATE → commit |
| 7 | Self-review | 別セッションで累積 diff をレビュー（ultracode fan-out 対応） |
| 8 | Pre-PR hygiene | コメント棚卸し・文字化け・markdownlint の掃き掃除 |
| 9 | Final report | ユーザー向けの最終サマリ |
| 10 | PR | `gh pr create` で PR 作成 |

## 成果物レイアウト

1課題ぶんが `.dev/{issue#}-{slug}/` 配下にまとまります（gitignored・個人用）。

```text
.dev/{issue#}-{slug}/
├── STATE.md              # 進捗トラッカー（毎回まずこれを読む）
├── approach.md           # 現状＋方針（step 2、step 3 で洗練）
├── approach-review/      # アーキ批判レビュー（step 3）
├── docs/explain/         # 方針確定後に生成（step 4）
├── plan.md               # フェーズ計画（step 5）
├── phase-NN-{slug}/      # order.md / report.md（step 6）
└── review/               # round-N / final-report.md（step 7, 9）
```

## 課題・改善の提案

ハーネスを使う中で不便だった点・改善案は **[Issue](https://github.com/pipo3944/dev-harness/issues)** へどうぞ。「ハーネス改善」テンプレートを用意しています。

```bash
gh issue create --repo pipo3944/dev-harness
```

直したら、コミット/PR に `Fixes #NN` を入れて自動クローズします。
