# Output Templates — required structure for every artifact

Every artifact begins with this header block:

```markdown
---
artifact: <NN-name>
repo: <name> @ <commit-hash>
generated: <ISO date>
scope: <whole repo | package/service name>
overall-confidence: <Verified | Traced | Inferred | Mixed>
open-unknowns: <count, each listed in body with [UNKNOWN]>
---
```

Confidence tags used inline throughout every artifact:
- `[VERIFIED]` — executed, tested, or observed at runtime
- `[TRACED]` — full code path read end-to-end
- `[INFERRED]` — concluded from structure, history, or naming; could be wrong
- `[UNKNOWN]` — a real gap; listed, never papered over

---

## 00-executive-summary.md  (max 1 page)

```markdown
# <Repo> — Executive Summary
## What it is
One paragraph: the product/system purpose in domain language, its users,
and its runtime shape (monolith / services / monorepo of N packages).
## Tech identity
Stack table: language(s), framework(s), datastore(s), broker(s), infra.
## Health at a glance
3–6 bullets: test coverage posture, doc posture, top 3 risks (link 09),
overall code age & activity (from git history).
## How to use this suite
2–3 sentences pointing at INDEX.md reading order.
```

## 01-system-map.md

```markdown
# System Map
## Component diagram
ASCII or mermaid: every component + every external system, arrows labeled
with protocol (HTTP/gRPC/topic-name/SQL).
## Component inventory
Table per component: name | responsibility (one line) | code location |
owner signals (CODEOWNERS/git) | confidence.
## Directory semantics
Top-level (and package-level for monorepos) directory table: path | what
lives here | what does NOT live here despite the name suggesting it might.
## Build & deploy reality
What CI actually does, stage by stage. Where deploys go. [TRACED] from CI
config, not from README claims.
```

## 02-runtime-architecture.md

```markdown
# Runtime Architecture
## Entry point census
Table: entry mode (HTTP/consumer/cron/CLI/webhook/route) | count | where
registered | representative example.
## Lifecycle traces (one per entry mode)
For each: numbered hop list bootstrap→completion, with every crossed layer
(auth, validation, rate-limit, tracing, flags, error translation) named and
its code location anchored.
## Process & concurrency model
Threads/event loop/goroutines/workers; what's sync vs async; where
backpressure exists or is missing [UNKNOWN if unclear].
## Configuration surface
How config enters (env/files/flags/remote), precedence order, and the
config keys that change behavior most dangerously.
```

## 03-data-model.md

```markdown
# Data Model
## Entity catalog
Per core entity: fields (required/optional/nullable + WHY nullable),
relationships, lifecycle states if any.
## ER diagram
mermaid erDiagram or ASCII.
## Invariants table
Invariant | enforced by (DB constraint / app code path / NOTHING) |
confidence. "NOTHING" rows are auto-copied to 09-risk-register.
## Migration timeline
Chronological digest of migrations with the story they tell — pivots,
renames, backfills. Feed pivots to 08-decision-log.
## Data at rest vs in flight
Serialization formats, event schemas, cache shapes, TTLs.
```

## 04-dependency-graph.md

```markdown
# Dependency Graph & Seams
## Internal coupling
Graph or table of module→module deps. Cycles listed individually with the
files involved. God modules: name | % of codebase importing it | blast
radius summary | link to 09 entry.
## Layering rules
The implicit import hierarchy, stated explicitly. Known violations listed.
## External seams
One subsection PER seam (DB, broker, each third-party API, each sibling
service): protocol | client location | timeout | retry | idempotency |
fallback | behavior when down. Any [UNKNOWN] cell → 09-risk-register.
```

## 05-flows/_flow-inventory.md

```markdown
# Flow Inventory
| # | Flow | Trigger | Priority (P0/P1/P2) | Why this priority | Status (Traced/Skeleton/Inventoried) | File |
Coverage: X/Y P0 traced (NN%), X/Y P1, X/Y P2.
Discovery sources used: routes, consumers, schedulers, UI actions, test
names, docs. Note any source that suggests flows you could NOT find code
for (ghost features) → 09.
```

(Per-flow file template lives in flow-tracing.md.)

## 06-patterns-conventions.md

```markdown
# Patterns & Conventions
## Named patterns
Per pattern: name | where used (anchors) | where conspicuously absent +
verdict (evolution vs deliberate, with evidence) | the LOCAL problem it
solves | the cost it pays.
## House rules (unwritten conventions)
Error handling idiom / logging & observability idiom / config & secrets /
DI style / naming / test structure / commit style. Each with a "do this,
not that" example pair lifted from real repo code style (paraphrased).
## Idiom checklist for new code
A literal checklist a contributor runs before opening a PR so their code
reads native.
```

## 07-glossary.md

```markdown
# Glossary
| Term | Authoritative definition | Code anchor (file/type) | Drift (places the term is used inconsistently) |
Sorted alphabetically. Drift column is mandatory — empty means you checked
and found none, not that you didn't look.
```

## 08-decision-log.md

```markdown
# Decision Log
Per decision:
### D<NN>: <decision title>
- What was decided:
- When (commit/PR/ADR ref or era):
- Why — SOURCED: <quote/paraphrase + reference>   ← or:
- Why — [INFERRED]: <reconstruction + the evidence pattern behind it>
- Status: still valid / superseded by D<NN> / worth revisiting (link debate)
Sourced and inferred are never mixed in one field.
```

## 09-risk-register.md

```markdown
# Risk Register
| ID | Risk | Severity (Critical/High/Med) | Evidence | Blast radius | Pre-touch checklist |
Categories to sweep: god modules, cycles, seams w/ unknown failure modes,
unenforced invariants, ghost/dead-looking code, env-coupled config, P0
flows without tests, secrets handling, upgrade landmines (pinned ancient
deps). "Pre-touch checklist" = what a dev must verify before modifying.
```

## 10-onboarding-path.md

```markdown
# Onboarding Path
## Reading order (Day 1 → Day 5)
Which artifacts + which code, in order, with time boxes.
## Guided first contribution
A concrete small change (chosen from real repo needs), walked through the
house patterns step by step, with predicted blast radius stated up front.
## Mastery exam (15–25 questions)
Questions an original developer answers cold. Each keyed: "→ taught by
<artifact>". Mix: architecture, flow behavior, failure modes, domain
language, "what breaks if". No trivia — every question maps to a real
change a dev might make.
```

## INDEX.md

```markdown
# Repo Mastery Suite — <repo>
Generated <date> @ commit <hash>. Re-run the skill to refresh; it diffs
against this hash and updates only affected artifacts.
## Reading order
00 → 01 → 02 → 05 (P0 flows) → 03 → 04 → 06 → 07 → 08 → 09 → 10.
## Coverage & confidence dashboard
| Artifact | Confidence | Open [UNKNOWN]s |
Flow coverage: <NN>% of P0, <NN>% of P1.
## Open questions for the team
Every [UNKNOWN] worth asking a human about, phrased as askable questions.
```