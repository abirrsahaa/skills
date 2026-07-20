# Flow Tracing Guide — how to write 05-flows/<NN>-<flow-name>.md

Flow traces are the heart of the suite. A flow trace is complete when a
developer who has read ONLY that file can (a) modify the flow safely,
(b) debug a production incident in it, and (c) predict its blast radius.

## Discovery: building the inventory first

Sweep ALL of these sources — each finds flows the others miss:
1. Route/controller registrations (HTTP surface)
2. Message/event consumer registrations (async surface)
3. Scheduler/cron definitions (temporal surface)
4. CLI/admin command registrations
5. Frontend route table + top-level user actions (forms, buttons w/ handlers)
6. Test file names — tests often name flows nobody documented
7. Docs/README claims — flows claimed but not found in code = ghost
   features → 09-risk-register
8. Webhooks and callback URLs registered with third parties

Priority rubric:
- **P0**: money moves, core domain action the product exists for, authn/z,
  data-destructive operations, anything in an SLA.
- **P1**: everything a normal user does weekly.
- **P2**: admin, backoffice, rare maintenance paths.

## The mandatory per-flow template

```markdown
---
flow: <NN>-<name>
priority: P0|P1|P2
status: Traced | Skeleton
confidence: <tag>
entry: <route/topic/cron expr/CLI cmd>
last-verified: <commit>
---
# Flow: <Name in domain language>

## 1. Trigger
What starts it, with exact anchor (route def, consumer binding, cron).
Payload/params schema. Who/what is allowed to trigger it.

## 2. Authorization & validation
Every check between trigger and core logic, in order, each with: what it
checks | where | what the caller sees on failure. Hidden business rules
concentrate here — state them as rules, not as code descriptions.

## 3. Core transformation (DOMAIN LANGUAGE)
Numbered steps of what the system DOES to the domain — "reserves inventory,
emits PendingOrder" not "calls svc.process()". Each step anchored to code.
State changes to each entity involved (tie to 03-data-model states).

## 4. Side effects
EVERY additional consequence: events published (topic + schema), other
services called, emails/notifications, cache writes/invalidations, metrics,
audit logs, files written. This section is where refactors silently break
things — it must be exhaustive. Mark each [TRACED] or [INFERRED].

## 5. Failure & edge behavior
- Partial failure at each step: what's rolled back, what isn't
- Retry behavior and idempotency (is a duplicate trigger safe? PROVE it or
  mark [UNKNOWN])
- Timeouts along the path and what the caller experiences
- Known ugly/clever code here and the war story it implies

## 6. Tests as specification
Which tests cover this flow (anchors). Rules the tests encode that appear
NOWHERE else. Any test/code disagreement (finding, not a skip).
Coverage verdict: which of sections 2–5 have no test at all → those rows
also land in 09-risk-register if this is P0.

## 7. Blast radius
If I change this flow, what else can break? (feeds from 04-dependency-
graph). Shared modules touched, events other systems consume, invariants
this flow is responsible for maintaining.

## 8. Sequence diagram
mermaid sequenceDiagram of the happy path + the most important failure path.
```

## Tracing discipline

- Trace by hand. Open every file on the path. Summarizers miss exactly the
  side effects and edge handling that make sections 4–5 valuable.
- Follow the data, not just the calls: a flow isn't done until you know
  every store it wrote to and every message it emitted.
- When a path forks on a feature flag or config value, trace BOTH branches
  or mark the untraced one [UNKNOWN] with the flag name.
- Async boundaries do not end the trace: if the flow publishes an event,
  identify the consumer(s) and either trace into them (same file, "async
  continuation" subsection) or link the sibling flow file that covers them.
- Time-box skeleton traces for P1: sections 1, 3, 7 only, status: Skeleton,
  so coverage stays honest.

## Frontend flow specifics

Section 3 becomes: state ownership (which store/context owns each piece),
data-down/events-up path, optimistic updates and their rollback. Section 4
includes: analytics events, local/session storage, URL state. Section 5
includes: loading/error UI states and race conditions on rapid re-trigger.

## Backend flow specifics

Always resolve for section 5: transaction boundaries (what's inside the DB
tx), lock acquisition, outbox/inbox patterns or their absence, exactly-once
vs at-least-once semantics on every broker interaction.