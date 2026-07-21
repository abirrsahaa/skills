---
name: finance-concept-mastery
description: >-
  Deep-research engine for the financial-markets domain knowledge embedded
  in a repo's business logic. Use when the user is looking at a codebase
  (trading, risk, settlement, regulatory reporting, pricing, treasury,
  custody, or any capital-markets system) and wants to genuinely understand
  the finance concepts behind the code — the market mechanics, valuation
  math, risk framework, and regulatory context that make the logic make
  sense. Trigger on: "explain the finance concepts in this repo", "what is
  DV01 / CVA / accrued interest / RWA doing here", "help me understand the
  domain behind this trading/risk system", or any request to learn the
  market/valuation/risk/regulatory knowledge financial code assumes the
  reader has. Also trigger for partial asks ("what does this margin calc
  mean") — run the relevant phase, emit just that explainer. Pairs with
  repo-mastery's business-logic extraction but goes deeper into finance
  domain and maps prerequisite concepts so nothing is understood in
  isolation.
---

# Finance Concept Mastery

Financial codebases assume a reader who already knows the market. A variable
named `dv01`, a function called `computeCVA`, a constant `ACT_360`, a config
flag `isda_csa_threshold` — each is a compressed reference to a body of
market convention, valuation math, risk theory, or regulation that took the
original developers months or years to absorb. This skill decompresses that
reference: for every finance concept the code touches, it explains the
concept properly **and** maps the chain of prerequisite concepts a reader
needs first — so understanding is never shallow or isolated.

**The test for done:** could you defend, in a room of the desk's quants and
risk managers, why the code computes something the way it does — including
naming the market convention or regulatory rule that requires it? If not,
the explainer hasn't gone deep enough yet.

## Core principle: concepts have dependencies, not just definitions

A glossary entry ("DV01: dollar value of a 1bp yield move") is not mastery.
Mastery is knowing that DV01 depends on **bond pricing** which depends on
**discounting** which depends on **the yield curve** which depends on
**day-count conventions** — and that you cannot really understand why a
trading desk cares about DV01 without knowing what breaks (P&L, hedging,
regulatory capital) when it's wrong. Every artifact this skill produces is
ordered and cross-linked by this **prerequisite chain**, not alphabetically
and not in the order the concepts happen to appear in the code.

## Contract: what running this skill MUST produce

Output lands in one folder: **`finance-concepts/`**.

```
finance-concepts/
├── INDEX.md                    # reading order = prerequisite order, coverage, freshness stamp
├── 00-executive-briefing.md    # 1 page: what part of finance this repo lives in, at what sophistication
├── 01-concept-map.md           # the full dependency graph: every concept found + its prerequisites
├── 02-foundations/             # primers for concepts nothing else can be understood without
│   └── <NN>-<foundation>.md
├── 03-concepts/                # one file per domain concept, filed in prerequisite order
│   └── <NN>-<concept-name>.md
├── 04-formula-reference.md     # every formula/model found, derived and worked, cross-linked to concepts
├── 05-regulatory-and-market-context.md  # current-state regulatory/benchmark facts the logic depends on
├── 06-glossary.md              # every term, one line, cross-linked to its full concept file
└── 07-learning-path.md         # ordered curriculum + worked drills + a mastery exam
```

Every concept file uses the mandatory structure in
`references/explainer-template.md` — read it before writing any
`02-foundations/` or `03-concepts/` file. Read
`references/finance-domain-taxonomy.md` before Phase 1 — it is the master
map of capital-markets domains and the canonical prerequisite edges between
them; use it as the checklist and the dependency-ordering authority so you
don't invent an ad hoc taxonomy per repo. Read
`references/concept-extraction-playbook.md` before Phase 1 for the
grep/search patterns that surface finance concepts hiding in code, naming,
comments, and tests. Read `references/regulatory-and-market-context.md`
before writing 05 — it holds dated, current-as-of facts (settlement cycles,
Basel status, benchmark rates) so the artifact doesn't state stale regime
facts as current.

## Confidence discipline

Same four tags as any rigorous audit, applied to every claim:
- `[MARKET-STANDARD]` — universal convention/definition, true regardless of
  this specific repo (e.g. "DV01 = price sensitivity to a 1bp yield shift").
- `[TRACED]` — this repo's code was read end-to-end and confirmed to
  implement the concept this way.
- `[INFERRED]` — concluded from naming, structure, or comments; flag it,
  don't present it as confirmed.
- `[UNKNOWN]` — a genuine gap (e.g. the code computes something but the
  convention it assumes — which day-count, which curve — isn't visible)
  → straight into the open-questions list; a wrong guess here is worse than
  an honest gap.

Never let a repo-specific implementation detail get silently generalized
into "how the market always does it," and never let a market convention get
silently assumed to be exactly how this repo does it — the whole value of
this skill is keeping those two apart while showing where they connect.

## Execution phases

### Phase 0 — Scope (no artifact)

Infer, don't interrogate: which part of the repo is in scope (a pricing
library, a risk engine, a settlement system, a regulatory-reporting module —
whole repo if it's cohesive), and roughly what sophistication level to write
at (this skill defaults to writing for a competent generalist developer who
wants to reach desk-level fluency — not a retail-investing primer, and not
assuming an existing quant background). State the assumption and proceed;
only ask if the repo genuinely spans unrelated business lines and the user
hasn't said which one matters.

### Phase 1 — Extraction: find every finance concept in the code

Use `references/concept-extraction-playbook.md` systematically, sweeping:
variable/function/class names, constants and config keys, comments and
docstrings, test names and fixtures, schema/column names, API contract
fields, and any embedded formulas or model implementations. For each hit,
resolve it against `references/finance-domain-taxonomy.md` to identify which
domain and layer it belongs to. Log every hit into a working list — this
list becomes `01-concept-map.md`'s raw material.

Do not skip a term because it looks self-explanatory. `notional`, `strike`,
`haircut`, `accrual`, `roll` all carry precise, non-obvious market meaning
that differs by instrument — verify each against the taxonomy rather than
assuming the plain-English reading is correct.

### Phase 2 — Prerequisite mapping → `01-concept-map.md`

For every concept found in Phase 1, walk its prerequisite chain using the
taxonomy's dependency edges, back to foundational math/market concepts —
even ones NOT present anywhere in the repo's code. A repo that computes CVA
without ever mentioning "probability of default" still requires the reader
to understand PD; that prerequisite gets a foundation file regardless of
whether the literal term appears in the codebase.

Build the graph explicitly: concept → direct prerequisites → their
prerequisites → ... → foundational layer. Concepts with no unmet
prerequisites are the foundation layer (`02-foundations/`); everything else
is `03-concepts/`, numbered in a valid topological order (every concept's
prerequisites have a lower number than the concept itself). This ordering
is the single most valuable thing this skill produces — it's what turns a
pile of definitions into an actual curriculum.

### Phase 3 — Foundations → `02-foundations/<NN>-<name>.md`

Write the foundation layer first, since every later file cites it. Typical
foundations (only write the ones the concept map actually requires — see
taxonomy Layer 0/1 for the standard list): time value of money and
discounting, probability/volatility/correlation basics, day-count and
compounding conventions, core instrument mechanics (bond, equity, FX,
option, swap basics) — whatever the concept map's roots turn out to be for
THIS repo. Each uses the explainer template in full.

### Phase 4 — Concepts → `03-concepts/<NN>-<name>.md`

Walk the topological order from Phase 2, writing one file per concept using
`references/explainer-template.md` in full: definition, prerequisites
(linked, not restated), the market mechanics, the formula/model (derived,
not just stated), how it appears in THIS repo's code (anchored, confidence-
tagged), a fully worked numeric example, common misconceptions, its
regulatory/accounting tie-in if any, and what it feeds into downstream.

Depth bar: a reader should finish a concept file able to (a) compute a
worked example by hand, (b) explain why the repo's implementation matches
or diverges from the textbook version and what that divergence costs or
buys, and (c) name the concept's regulatory or P&L consequence if it's
wrong. A file that only defines the term has not met the bar — send it back
through Phase 4 rather than moving on.

### Phase 5 — Formula reference → `04-formula-reference.md`

Consolidate every formula/model across all concept files into one
reference: the formula, each variable defined, the derivation sketch (not
just "trust me"), the market convention it depends on (which day-count,
which compounding, which curve), and links back to the concept file(s) that
use it. This is the artifact a reader keeps open while reading the actual
code.

### Phase 6 — Regulatory & market context → `05-regulatory-and-market-context.md`

Use `references/regulatory-and-market-context.md` as the dated factual base
(settlement cycles, benchmark-rate regime, capital-framework status) and
tie each fact explicitly to the parts of the repo/concepts it constrains
(e.g., "this settlement module assumes T+1 — true for US equities since May
2024, but the EU/UK/CH leg of this system will need T+1 logic only from 11
October 2027"). Flag anywhere the repo's assumptions look like they encode a
regime that has since changed or is scheduled to change — this is exactly
the kind of gap a generalist developer would never spot on their own but a
domain expert would catch immediately.

### Phase 7 — Glossary & learning path → `06-glossary.md` + `07-learning-path.md`

Glossary: every term, one line, linked to its full concept file — the quick-
lookup layer for when you already know the map and just need a reminder.

Learning path: the topological order from Phase 2 turned into an actual
study plan — grouped into sessions, each with a worked drill (compute
something by hand using the formula reference), and closing with a
**mastery exam**: 20–30 questions mixing "explain this concept," "why does
the code do X and not the textbook version," and "what breaks if this
number is wrong" — each keyed to the concept file that teaches it. This is
the self-test that the depth bar in Phase 4 was actually met.

Write `INDEX.md` last: reading order (= prerequisite order), per-file
confidence summary, and the open-questions list (every `[UNKNOWN]`, phrased
as a question worth asking a desk/risk/quant colleague).

## Idempotent re-runs & partial requests

If `finance-concepts/` exists, diff new code against the stamped commit and
extend rather than regenerate — new concepts get slotted into the existing
topological order, not appended at the end. For a partial ask ("just
explain the margin calculation"), run Phase 1 scoped to that area, Phase 2
just for that concept's chain, and emit only the foundation + concept files
actually needed — using the same templates, not a shortcut version.

## Anti-patterns (hard rules)

- Never write a concept file before its prerequisites exist — a reader
  hitting an uncited term they haven't learned yet is a broken chain.
- Never state a market convention as fact without `[MARKET-STANDARD]`, and
  never state a repo's convention as universal without `[TRACED]` proof.
- Never let regulatory/benchmark facts go unstamped by date — this domain
  changes fast enough that an undated claim is actively misleading within a
  year (see the LIBOR→SOFR and T+1 transitions in the reference file).
- Never stop at definition-level for a concept the repo actually implements
  — implemented concepts get the full worked-example + code-anchor
  treatment; only concepts that are prerequisite-only and absent from the
  code can stay at primer depth.
- Never produce a flat glossary as the primary deliverable — the ordered,
  cross-linked concept files and the dependency map are the point; the
  glossary is a convenience index on top of them, not a substitute.