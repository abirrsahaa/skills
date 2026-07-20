---
name: repo-mastery
description: >-
  Production-grade codebase mastery engine. Run this skill whenever the user
  points at a repo they don't fully know — frontend, backend, monorepo, or
  monolith — and wants to be onboarded, "understand this codebase", "explain
  the architecture", "map every flow", "document this repo", "make me a
  developer of this project", audit before a refactor, or take over ownership.
  It systematically reverse-engineers the repo and GENERATES a complete docs
  suite (numbered markdown files — system map, runtime architecture, data
  model, dependency graph, per-flow traces, patterns, glossary, decision log,
  risk register, onboarding path) that makes the reader operate like an
  original developer. Trigger even for partial asks ("what does this service
  do", "trace the checkout flow", "what patterns does this use") — run the
  relevant phase and emit its artifact. Do not wait for the words "master" or
  "documentation".
---

# Repo Mastery — Production Codebase Mastery Engine

This skill turns an unfamiliar repository into a **complete, verifiable docs
suite** whose reader can extend, debug, and debate the system like one of its
original developers. It is not a summarizer. Every artifact it produces must
answer "why is it shaped this way and what breaks if I touch it" — not just
"what does it do".

## Contract: what running this skill MUST produce

All output goes into one folder: **`repo-mastery/`** (inside the repo if you
have write access and the user wants it committed, otherwise as deliverable
files for the user). The suite is:

```
repo-mastery/
├── INDEX.md                  # reading order, coverage %, freshness stamp
├── 00-executive-summary.md   # 1 page: what this system is, for whom, in what shape
├── 01-system-map.md          # components, boundaries, diagram, tech inventory
├── 02-runtime-architecture.md# entry points, processes, request/event lifecycles
├── 03-data-model.md          # entities, relationships, migrations timeline, invariants
├── 04-dependency-graph.md    # module coupling, god modules, cycles, external seams
├── 05-flows/                 # ONE FILE PER FLOW — the heart of the suite
│   ├── _flow-inventory.md    # every discovered flow, prioritized, coverage tracked
│   └── <NN>-<flow-name>.md   # full trace using the flow template
├── 06-patterns-conventions.md# named patterns w/ local why + cost; unwritten house rules
├── 07-glossary.md            # domain language, code anchors, naming drift
├── 08-decision-log.md        # the "why" archive: sourced vs inferred, clearly marked
├── 09-risk-register.md       # dragons, debt, blast-radius map, "do not touch blind" list
└── 10-onboarding-path.md     # first-week plan + first-contribution walkthrough + mastery exam
```

Every file starts with a standard header (see `references/output-templates.md`):
scope, confidence level (Verified / Traced / Inferred / Unknown), source
evidence, and last-verified commit hash. **Never present an inference with the
same confidence as a traced fact** — the suite's credibility depends on this.

Read `references/output-templates.md` before writing any artifact — it holds
the exact required structure for every file above. Read
`references/stack-playbooks.md` when you reach Phase 1 to get the right
commands for the detected stack. Read `references/flow-tracing.md` before
writing any file in `05-flows/`.

## Operating principles

1. **Top-down, then drill.** System shape → module coupling → line-level only
   where risk concentrates. Never file-by-file from the entrypoint outward.
2. **Explain by decisions, not syntax.** Every artifact answers: why this
   shape, what constraint forced it, what breaks if removed.
3. **Write as you go.** Each phase ends by writing/updating its artifact
   BEFORE the next phase starts. If the session dies mid-run, everything
   completed so far is already on disk. Never hold the suite in your head
   for one big write at the end.
4. **Evidence discipline.** Four confidence tags, used everywhere:
   `[VERIFIED]` (ran it / tested it), `[TRACED]` (read the full code path),
   `[INFERRED]` (concluded from shape/history), `[UNKNOWN]` (flagged gap).
   An honest `[UNKNOWN]` is worth more than a confident guess.
5. **Coverage is tracked, not assumed.** `_flow-inventory.md` lists every
   discovered flow; INDEX.md reports what % are traced. "Mastery" without a
   number is a vibe.
6. **Idempotent re-runs.** If `repo-mastery/` already exists, diff the repo
   against the stamped commit hash, update only affected artifacts, and
   refresh the freshness stamps — don't regenerate from scratch.

## Execution phases

Run phases in order. Each lists its goal, method, and the artifact it emits.
For a partial user ask (e.g. "just trace the checkout flow"), run Phase 0
lightweight + the relevant phase, and emit just that artifact using the same
templates.

### Phase 0 — Scoping (no artifact; 5 minutes max)

Determine: goal (contribute / audit / takeover / due diligence), boundary
(whole repo vs. specific package in a monorepo), and depth budget. Infer from
context; ask only what you genuinely can't infer. Default assumption if user
just said "master this repo": *contribute confidently, whole repo (or the
primary service of a monorepo), full suite*. State the assumption, don't
block on it.

Monorepo rule: run Phases 1–4 once at workspace level, then Phases 2–9 per
in-scope package. One model per bounded context + a thin map connecting them.
Never one flat model across unrelated services.

### Phase 1 — Recon → `00-executive-summary.md` + `01-system-map.md`

Detect the stack, then use the matching section of
`references/stack-playbooks.md` for concrete commands. Universal moves:

- Size/shape: file counts, LOC by language, top-level directory semantics.
- Read IN FULL: README, CONTRIBUTING, ARCHITECTURE, `docs/`, all ADRs. ADRs
  are the highest-value file type in any repo; their absence is itself data
  (log it in 08-decision-log as "no documented decisions — all whys below
  are reconstructed").
- CI/CD configs (`.github/workflows`, Jenkinsfile, etc.) — the *actual*
  build/test/deploy truth, more honest than any doc.
- Infra-as-code, Dockerfiles, k8s manifests, env/config templates — the
  runtime topology and every external dependency the system admits to.
- Workspace configs (turbo/nx/lerna/pnpm-workspace/go.work/maven modules) —
  monorepo structure and build graph.

Emit both artifacts using their templates. The executive summary is written
LAST in this phase but placed first in the suite.

### Phase 2 — Entry points & lifecycles → `02-runtime-architecture.md`

Enumerate EVERY way control enters the system: HTTP routes, message/event
consumers, cron/schedulers, CLI commands, admin endpoints, webhooks, signal
handlers, frontend routes + global providers. Monoliths hide 3–6 entry
modes in one process — find all of them before assuming one.

Hand-trace ONE representative request/action per entry mode from bootstrap
to completion, recording every layer crossed (auth, validation, rate limit,
tracing, feature flags, error translation). These layers seed Phase 6's
cross-cutting conventions. Do not delegate this trace to a summarizer — the
value is in the layers you're forced to notice.

### Phase 3 — Data model → `03-data-model.md`

Schemas, migrations, ORM models, proto/GraphQL defs, core type definitions.
For each core entity: required vs optional fields, relationships, nullability
*with reasons* (nullable often = historical pivot), and **invariants** — what
must always be true, and which code enforces it (DB constraint vs app code vs
nothing — "nothing" goes straight into 09-risk-register).

Read migration history **in chronological order** — it is the team's diary of
how their domain understanding changed. Pivots found here feed 08-decision-log.
Start `07-glossary.md` now; every domain noun gets an entry the moment you
meet it.

### Phase 4 — Coupling & seams → `04-dependency-graph.md`

Run the stack's dependency analyzer (see playbooks). Report:
- **Cycles** (compromised boundaries — usually deadline scars),
- **God modules** (imported by >30% of the codebase; each gets a blast-radius
  entry in 09-risk-register),
- **External seams** (every DB, broker, third-party API, other service):
  for each seam document protocol, timeout, retry, idempotency, fallback,
  and what happens when it's down. `[UNKNOWN]` on failure behavior at a seam
  is a top-priority risk-register entry.
- **Layering rules** — the implicit "who may import whom", and every
  violation of it you can find.

### Phase 5 — Flow traces → `05-flows/` (the heart; biggest time share)

First build `_flow-inventory.md`: EVERY flow you can discover — from routes,
consumers, schedulers, UI actions, test names, and docs — prioritized P0
(revenue/core-domain/security) → P2 (admin/rare). This inventory is the
coverage denominator.

Then trace flows in priority order, one file each, using the mandatory
template in `references/flow-tracing.md` (trigger → authz/validation → core
transformation *in domain language* → side effects → failure/edge handling →
tests-as-spec cross-check → blast radius). Full-suite target: 100% of P0,
all P1 at least skeleton-traced, P2 inventoried. Track % in INDEX.md.

Cross-check every trace against its tests — tests are executable spec and
encode rules nobody wrote down. Any test/code disagreement is a finding, not
a skip.

### Phase 6 — Patterns & conventions → `06-patterns-conventions.md`

Only NOW name patterns — after real logic, so names fit reality instead of
being force-fit. Per pattern: name, where used, where conspicuously NOT used
(evolution vs deliberate exception — determine which), the *local* problem it
solves (not the textbook one), and the cost it pays. Plus the unwritten house
rules: error-handling style, logging/observability idiom, config & secrets
handling, DI style, naming, test structure, commit conventions. Violating
these is how new contributors write code that "works" but reads foreign.

### Phase 7 — Consolidate knowledge → `07-glossary.md` + `08-decision-log.md`

Finalize the glossary (term → definition → code anchor → naming drift
locations). Build the decision log from ADRs, commit messages
(`git log --grep "fix\|hack\|workaround\|revert"`), PR references, and
inference from code shape — with sourced vs `[INFERRED]` rigorously separated.

### Phase 8 — Risk → `09-risk-register.md`

Everything dangerous in one place: god modules with blast radii, cycles,
seams with unknown failure behavior, unenforced invariants, dead-looking code
that might not be dead, config coupled across environments, test gaps on P0
flows. Each entry: severity, evidence, and what a developer must check before
touching it.

### Phase 9 — Validate, then enable → `10-onboarding-path.md` + `INDEX.md`

Mastery is a hypothesis until tested:
1. **Predict-then-check**: pick a small real change; write the predicted
   blast radius from artifacts 04/05/09; make the change (or dry-run it if
   no write access); run tests; compare. A mismatch = model gap → fix the
   artifact, then the change.
2. Write `10-onboarding-path.md`: day-by-day reading order through the suite,
   a guided first contribution using the house patterns exactly, and the
   **mastery exam** — 15–25 questions a true developer of this repo could
   answer cold (each keyed to the artifact that teaches it). This is the
   user's self-test that the suite worked.
3. Write `INDEX.md` last: reading order, per-artifact confidence summary,
   flow coverage %, list of open `[UNKNOWN]`s, repo commit hash + date stamp.

## Debate rules (when the user wants to change the system)

Only after Phase 9 validation. Structure every change argument as:
current approach → why it was likely chosen (cite 08-decision-log) → what has
changed since → the alternative → what the alternative costs. Proposing
"should be X" without stating why it's currently not-X is the signature move
of someone who hasn't done the reading. The suite exists so you never have to
make that move.

## Anti-patterns (hard rules)

- No AI summary substitutes for a Phase 5 hand trace.
- No change proposals before Phase 9 validation.
- No "fixing" inconsistent pattern use before determining oversight vs
  deliberate exception.
- No unmarked inferences — every claim carries its confidence tag.
- No skipping the glossary; domain-language drift is the #1 new-contributor
  bug source.
- No giant end-of-run write; artifacts land per-phase.