---
name: dev-harness
description: >-
  Human-gated, multi-session development harness that drives a single issue from
  intake all the way to PR through 10 checkpointed steps (intake → align/approach →
  architecture-critique review → explain doc → plan → phased implementation →
  self-review → pre-PR hygiene → final report → PR), with durable Markdown handoff
  under .dev/. Use this whenever the user wants to tackle a substantial issue or
  task in a structured, understand-first way — especially when they say
  "dev-harness", "ハーネスで進めて", "この課題をハーネスで対応", or want phased work
  managed across separate sessions with order.md / report.md handoff. Resumable:
  re-invoking on an in-progress task reads .dev/{issue}-{slug}/STATE.md and
  continues from the right step. Do not use for trivial one-line edits where the
  full ceremony would be overkill.
---

# dev-harness

A harness for taking one issue from "I don't fully understand this yet" all the
way to a reviewed PR, **without losing the thread**. The whole point is to force
understanding *before* code, pressure-test the approach *before* implementing,
decompose work into phases, and keep every handoff as an inspectable Markdown file
so a human can follow along and intervene at any gate.

## Operating principles

- **Understand before building.** The most common failure mode is writing code
  before grasping the real problem — and committing to a plausible-but-suboptimal
  approach. The align discussion, the architecture-critique review, and the explain
  doc exist to kill both.
- **Human-gated, not autonomous.** At each GATE, stop and wait for the user's OK.
  Speed is not the goal here; not losing track is.
- **Speak to the user in plain, self-contained terms.** Every user-facing message —
  GATE presentations, `AskUserQuestion`, approach confirmations — must stand on its
  own. **Never surface run-internal symbols** (finding IDs `F6`/`M9`, 論点 numbers,
  approach names `案B`, plan item labels `(a)`) as if the user knows them; the user
  doesn't hold your internal registry, and doing so reads as "answer this code I
  won't explain". Say what you're actually unsure about and what you need decided,
  in words. Introduce any needed technical term with its meaning on the spot.
  - **Narrow what you defer to the user.** Decide engineering choices yourself and
    let the independent reviews (Step 3/7) pressure-test them; bring to the user the
    **product / preference** decisions that are genuinely theirs — with options and
    background stated plainly.
- **Durable handoff over chat.** Plans, orders, and reports live as `.md` files
  under `.dev/`, not buried in chat scrollback. Anyone can open the working dir
  and see exactly where things stand.
- **Session separation is explicit, never silent.** Heavy units run in fresh
  contexts, but the *kind* of fresh context matters and the user must never be
  surprised by it:
  - The **orchestrator** (plan + phase-loop + gates + commits) runs in a **new
    session by default**, started after the align gate — it rebuilds context from
    `STATE.md` / `approach.md` / `plan.md` rather than dragging intake chat along.
  - **Phase execution** and **self-review** default to a **separate session the
    user starts**: write the `order.md`, then stop and tell the user which path to
    open. Do **not** silently spawn a subagent for these — spawn one only if the
    user agreed up front.
  - **Building the explain doc** and the **architecture-critique review** are the
    places where a spawned subagent *is* the fine default — they're bounded,
    self-contained artifacts and the context already holds what they need.
  - Collapsing layers into one session is fine for small tasks — but only with
    explicit user agreement, never by default. The `.dev/` files make any choice work.
- **One STATE writer, reserved role names.** Multiple sessions writing `STATE.md`
  causes update races and "progress the orchestrator never saw". So:
  - **Role names are reserved.** "オーケストレーター" = the single session running
    plan→PR (gates + commits + STATE). Other sessions are "ワーカー" (phase execution)
    or "レビューア" (self-review) — they must **not** call themselves orchestrator,
    even when they end up applying fixes.
  - **Only the orchestrator writes `STATE.md`.** Workers/reviewers report **only via
    `report.md`**. If a reviewer session applies fixes or makes a gate call, reflecting
    that into `STATE.md` is still the orchestrator's job (it reads the report and
    updates STATE). This keeps STATE a single, non-conflicting source of truth.
- **Proportional ceremony.** This is the default full flow. For smaller issues,
  it's fine to propose simplifying, merging, or skipping steps — just say so and
  get the user's agreement.

### Session map (which step runs where)

- intake / align / architecture-critique / explain … **起点セッション**
  (explain と arch-review は subagent に出してよい)
- plan + phase-loop 運行 + 各 GATE + commit … **オーケストレーターセッション(新規)**
- 各 phase 実装 … **フェーズ実行セッション(新規・phase ごと)**
- 各レビュー round … **レビューセッション(新規)**

## Working directory layout

Everything for one issue lives under `.dev/{issue#}-{slug}/` (gitignored,
personal). Create it at intake.

```
.dev/{issue#}-{slug}/
├── STATE.md                  # progress tracker — read this first on every invocation
├── approach.md               # current-state + approach (step 2, refined through step 3)
├── approach-review/          # architecture-critique design review (step 3)
│   ├── order.md
│   └── report.md
├── docs/explain/             # built AFTER the approach is final (step 4)
│   ├── order.md
│   └── index.html            # HTML + canvas
├── plan.md                   # phase plan (step 5)
├── phase-01-<slug>/
│   ├── order.md              # orchestrator → worker
│   └── report.md             # worker → orchestrator
├── phase-02-<slug>/ ...
└── review/
    ├── round-1/{order.md,report.md}
    └── final-report.md       # user-facing summary (step 9)
```

`{slug}` is a short kebab-case description, e.g. `142-cart-total-rounding`.

## Resumability: always start by reading STATE.md

This harness is meant to span multiple sessions. On **every** invocation:

1. Determine the target issue (from the argument, or ask the user).
2. Look for `.dev/{issue#}-{slug}/STATE.md`.
   - **Not found** → fresh start. Begin at Step 1 (Intake).
   - **Found** → read it, tell the user where things stand, resume at `current-step`.

Keep `STATE.md` updated as you complete each step — it is the single source of
truth for "where are we". **Whenever you invent an internal symbol** (approach
names, question/option labels, unresolved-point tags — anything a worker might echo
into code/UI), append it to the **内部記号レジストリ** section immediately. Step 8's
hygiene grep seeds are generated from it, so an un-logged symbol won't be caught.
Template:

```markdown
# State: #142 cart-total-rounding

current-step: plan   # intake | align | arch-review | explain | plan | phase | review | hygiene | final | pr | done
issue: #142

## 進捗
- [x] intake
- [x] align (approach draft)
- [~] arch-review   # [x] done / [~] skipped / [ ] pending
- [~] explain
- [ ] plan
- [ ] phases (0/N)
- [ ] review
- [ ] hygiene
- [ ] final-report
- [ ] pr

## 内部記号レジストリ
<!-- 本運行で発明・使用した内部記号を、生まれるたびにここへ追記する。
     Step 8 hygiene の grep 種はこのレジストリから生成される。
     例: 案A/案A'/案B（approach 呼称）, Q3/Q4（論点）, U1〜U3（未解決点）, ターゲット(a)/(b)
     注意: これは成果物 hygiene と内部管理のためのもの。ユーザーとの対話（GATE・質問）には
     持ち込まない（記号のまま質問しない。plain-terms 原則を参照）。 -->
- （なし）

## メモ
次にやること: ...
```

## The 10 steps

A `→ GATE` means: present the result, then **stop and wait** for the user's
explicit OK before continuing.

### 1. Intake

- Confirm whether a GitHub issue already exists. If not, draft one and create it
  with `gh issue create` (the `gh` binary may be at `/opt/homebrew/bin/gh` if not
  on PATH). Use the repo's existing labels.
- Create the working dir `.dev/{issue#}-{slug}/` and write the initial `STATE.md`.
- Gauge the issue size. If it's small, propose simplifying the flow (which steps to
  merge/skip) and get agreement — don't run the full ceremony silently on a trivial fix.

### 2. Align — approach draft

- Discuss and converge on the current-state understanding, the root cause, and the
  fix approach. This is where problem understanding happens (the explain doc comes
  later, once the approach is settled).
- Capture the result in `approach.md`: current state, root cause, approach, scope,
  risks. Treat it as a **draft** — it goes through the architecture-critique review
  next *before* the user gates it.

### 3. Architecture-critique review (skippable for small tasks)

A plausible approach reviewed by its own author in the same context tends to pass
too easily. So before the approach gate, get an independent adversarial design
review.

- Write `approach-review/order.md` from `references/approach-review-order-template.md`.
  **Spawn a subagent** (fresh context) given `approach.md` + the explain inputs +
  relevant real code. It checks, against the target framework's conventions and the
  repo's existing patterns: is this the best approach? better designs? missed impact
  scope? → `approach-review/report.md`.
- Reflect the findings into `approach.md`. Zero findings → pass through.
- This is a **design** review (pre-implementation), distinct from Step 7's
  post-implementation diff review. Don't conflate them.
- Skippable for small tasks (proportional ceremony) with the user's agreement.
- **Explain-first の申し出（任意）:** 方針 GATE をかける前に、
  「理解を助ける explain を**先に**作りますか？」と聞いてよい。approach は arch-review を
  通してほぼ確定しているので、ここで建てても draft 作り直しのリスクは小さい。要ると言われたら
  Step 4 を先に実施し、その explain を判断材料にして方針 GATE をかける。順序は固定しない
  （デフォルトは下記の「方針 GATE → explain」でよいが、この選択肢を明示する）。
- → **GATE: 方針 OK?** (the now-refined approach)

### 4. Explain (optional) — reflects the (near-)final approach

Built once the approach is settled, so the "how to fix" visuals show the approach
you'll actually implement (not a draft that gets revised away). May run **before the
方針 GATE** if the user takes the explain-first offer in Step 3 — the arch-reviewed
approach is already near-final, so building here is safe; if the gate then changes the
approach materially, lightly follow up the explain.

- Ask the user whether they want it (unless already agreed via the Step 3 offer).
  Respect a "不要" and skip to Step 5.
- Write `docs/explain/order.md` pointing the builder at
  `references/explain-checklist.md` — it defines both the **required content** and the
  **構成の鉄則（順序）**: lead with what's being built/fixed and a concrete scenario;
  fold internal structure/mechanism to the end. Works for both bug fixes and new
  features (read "問題" as "the pain this change removes" for features).
- Build `docs/explain/index.html` — HTML + canvas visuals (state transitions /
  sequence / data flow, scenario-driven — not an arch diagram for its own sake).
  **Spawn a subagent** (default). It must cover: what's being built/fixed, the concrete
  scenario, *why* (root cause), and *how* (the confirmed approach); *where* (file:line)
  and internals belong near the end — readable by a non-expert yet conveying the essence.
- → **GATE: 理解 OK?** Refine if not.

### 5. Plan (new orchestrator session)

Start a **fresh orchestrator session by default**: it rebuilds context from
`STATE.md` + `approach.md` + the explain doc and runs the plan + phase-loop + gates
+ commits, free of intake/align chat. Continuing in the same session is allowed only
for small tasks with explicit user agreement — never silently.

- Produce `plan.md`: an ordered list of phases, each with a one-line goal and the
  acceptance criteria it owns. Let issue size drive phase count — a small issue may
  be one phase. Don't pad.
- → **GATE: 計画 OK?**

### 6. Phase loop

For each phase, in order:

1. **Write `phase-NN-<slug>/order.md`** from `references/order-template.md`. Fill
   every section. Point the worker at `approach.md`, the explain doc, the prior
   phase's `report.md`, and tell it to read the repo — never a self-contained order
   that hides context.
2. **Execute in a separate session (default).** Stop and tell the user: "このパスを
   新規セッションで開いて実行してください" with the order path. The worker does the
   work and writes `phase-NN-<slug>/report.md` using the report skeleton **inlined in
   the order** (the worker has no skill loaded, so it can't resolve `references/...` —
   expand the skeleton into the order rather than citing the path).
   Do **not** silently spawn a subagent — only if the user agreed up front.
3. **Independent mechanical verification.** The worker's "formatted / tests green"
   is self-reported and can drift from the final saved files. So on the saved files,
   run it yourself before the gate:
   - format check (`--set-exit-if-changed`) — format-only drift → you fix it and
     fold it into the phase commit; don't bounce it back.
   - analyze (changed files)
   - the phase's tests (+ neighbor suite for regressions)

   Run these **unpiped** and check each command's **exit code** yourself: piping to
   `tail`/`head`/`grep` makes the pipeline exit code that of the last stage, so a
   failure slips through the `&&` chain — the worker and verifier sharing that blind
   spot is exactly how a defect passes the gate. Compare against the report's claimed
   results; a report claiming green that you can't reproduce is a bounce-back.

   **Cleanup safety.** If your verification does a destructive/temporary rewrite
   (e.g. a break-test: corrupt a contract value → confirm tests fail), **don't undo
   it with `git checkout` / `git restore` / `git stash`** — the phase artifacts are
   still uncommitted at verify time (commits happen after the gate), so those wipe
   the worker's uncommitted work along with your change. Reverse your own edit
   **non-destructively** (inverse `sed`/Edit). If a git-based restore is truly
   needed, snapshot the artifacts first (as with `report-r<N>.md`).

   Logic drift (not just formatting) → send back or open a small fix phase.

   **When measurement/experiment is the phase's deliverable (spike-type runs):**
   "tests green" ≠ "the measured conclusion is correct". So additionally, **the
   claims that decide the outcome** (anything an ADR or design decision rests on)
   must be **independently reproduced by the orchestrator in a minimal setup that
   does not depend on the worker's implementation** — at least one such claim. This
   catches things no checklist item would (e.g. a worker's "foreignObject taints the
   canvas, disqualified" turned out to have a `data:` escape hatch on independent
   repro). If you can't reproduce it (environment limits etc.), say so in the GATE —
   include what you tried and how.
4. → **GATE: report OK?** Present the report *and* your verification result.
   **On a bounce-back (差し戻し):** before writing the follow-up order, **snapshot the
   current `report.md` to `report-r<N>.md`** (round number) — `.dev/` is gitignored, so
   the pre-bounce report (incl. measurement numbers the worker will overwrite) is
   otherwise lost, and the bounce is itself the record of "review changed the
   conclusion". Write the follow-up as an appended "追補 N" section at the end of
   `order.md` (don't rewrite the original order).
5. On approval, **commit this phase** (see Conventions). Update `STATE.md`
   (`phases (k/N)`).

### 7. Self-review (separate session)

A different pair of eyes, not carrying the implementation context, catches what the
author can't. Default to a **separate session** (don't silently spawn a subagent).

- Write `review/round-N/order.md` from `references/review-order-template.md`. Review
  the **cumulative diff** (`merge-base(main,HEAD)..HEAD`), not files in isolation.
- Dimensions (core + adjust per task): correctness / acceptance criteria /
  regressions / simplicity-dup-altitude / **language-framework idioms** /
  **security** / **performance**. Keep the skill language-agnostic — the reviewer
  detects the target language/framework from the diff and checks against its
  conventions and the repo's existing style.
- **ultracode fan-out** (when the user uses ultracode): per-dimension parallel
  subagents → structured findings → adversarially verify blockers/majors (refute) →
  dedup + prioritize → integrated report + verdict. Mark items not verified in
  practice; don't silently drop them.
- **Billing:** ultracode / Workflow is user-triggered. The orchestrator writes the
  order only and does **not** launch it — prompt the user to open a new session and
  run ultracode there.
- Stop when a round comes back clean. Don't grind on nits.
- → **GATE: レビュー結果 OK?**

### 8. Pre-PR hygiene pass

A quick sweep over the cumulative diff before the PR, against this checklist:

1. **Session/run-only jargon in comments** — review finding IDs (`M9`/`N2`),
   approach names (`案B`), phase numbers, and any run-specific symbol — rewrite to
   reason-based. The grep seed = **fixed patterns + STATE's 内部記号レジストリ**:
   start from `（[MN][0-9]+）`, `案[A-Z]`, `Phase ?[0-9]`, then add a pattern for each
   registered symbol (fixed patterns alone miss run-invented ones like `Q1c`/`U1〜U3`).
   Judge by meaning, touch only this task's diff.
2. **Over-commenting** (restating the obvious) → trim.
3. **Missing comments** where a non-obvious invariant or "why" needs one → add.
4. **Text/markdown defects** — mojibake (U+FFFD) via
   `LC_ALL=C grep -nP $'\xef\xbf\xbd' <files>`; markdownlint (e.g. MD040: fenced
   block missing a language); over-assertive wording ("確実に成功する" → soften).

Output a small standalone commit (e.g. `docs(scope): コメント棚卸し`); no behavior or
artifact change. Orchestrator-direct or a light subagent. Show the result in the
final report (no separate gate).

### 9. Final report

- Write `review/final-report.md` — a user-facing summary: what was done, key
  decisions, problems hit, and any follow-ups (including the hygiene commit). The
  `develop-story` skill can help if a narrative of the thought process is wanted.
- → **GATE: 最終レポート OK?**

### 10. PR

On approval, create the PR with `gh pr create`. Body uses this standard format —
keep implementation detail (function names / line numbers / internal design) **out**
of the body; those live in the diff.

```markdown
<1〜2文で超ざっくり要約：何の問題を解消するか>

Closes #<issue>

### 新機能
- <ユーザー目線・プレーンな言葉のリスト>

### 改善
- <リスト>

### 検証
- [x] <何を確認したか（受入条件・セルフレビュー等）>

<任意: follow-up を1行>
```

- **No AI-generated footer/signature** in the body (e.g. "🤖 Generated with Claude
  Code"). This explicitly overrides the base system default that appends one — same
  spirit as no `Co-Authored-By`.
- Update `STATE.md` to `done`.

## Conventions (bake these in)

- **Commits per phase**, after the phase's GATE approval. The gate approval *is* the
  authorization — never commit before it. One commit per phase keeps history
  reviewable and squashable.
- **No `Co-Authored-By` line** in commit messages.
- **No AI-generated footer/signature** in PR bodies or commits (overrides base
  defaults).
- **Review-round fixes are the orchestrator's commit.** Fixes for self-review
  (Step 7) findings are committed by the **orchestrator as an independent commit
  after the レビュー GATE approval** — even if a reviewer session drafted them. The
  reviewer reports; the orchestrator commits and updates `STATE.md` (see the
  one-STATE-writer principle).
- **No auto-commit** outside the per-phase rule above; never commit speculatively.
- If on the default branch, create a working branch before committing.
- **Handoff files**: persistent harness artifacts go under `.dev/` (order, report,
  plan, approach, explain). Ephemeral one-off prompts for a side session go to
  `tmp/prompts/`. `.dev/` is gitignored.

## Backlog: improving the harness itself

This harness evolves through use. When you (or the user) hit a rough edge while
running it, capture it as a **GitHub Issue** on the harness repo
(`pipo3944/dev-harness`) so anyone can propose and track improvements:

- File it with `gh issue create --repo pipo3944/dev-harness` (or the web UI),
  using the **ハーネス改善** issue template — note where it surfaced (which step)
  and a fix idea.
- When fixed, reference the issue in the fixing commit/PR (e.g. `Fixes #NN`) so
  it closes automatically.

Proactively suggest filing an issue when something in the flow felt clunky
during a real run.

## References

- `references/order-template.md` — phase order.md schema (orchestrator → worker)
- `references/report-template.md` — phase report.md schema (worker → orchestrator)
- `references/approach-review-order-template.md` — architecture-critique design review (step 3)
- `references/review-order-template.md` — self-review order, dimensions + ultracode fan-out (step 7)
- `references/explain-checklist.md` — required content for the explain doc (step 4)
