# Business Logic Extraction — reverse-engineering the business from the code

Most repos have no product spec. The business rules are still there — encoded,
scattered, and unlabeled. This guide is how you recover them. Code tells you
MECHANISM; you must reconstruct POLICY. Mastery is knowing which is which,
because mechanism can be refactored freely and policy cannot be touched
without a business conversation.

## The policy/mechanism split (apply constantly)

- **Mechanism** = how the system accomplishes something. Retry loops, caching,
  serialization, DI wiring, connection pools. Changing this is an engineering
  decision.
- **Policy** = a business rule the company would have to approve changing.
  "Refunds allowed within 30 days", "trial is 14 days", "orders over $500 need
  manual review", "EU users get different consent handling".
- The trap: policy hides inside mechanism. A `retryCount = 3` is mechanism. A
  `MAX_FAILED_LOGIN_ATTEMPTS = 5` is policy wearing mechanism's clothes.
  When unsure, ask: *would a product manager have an opinion about this
  number?* If yes, it's policy — catalog it.

## Where business rules physically hide (sweep all 12)

1. **Validation layers** — required fields, ranges, formats. Every constraint
   is a rule someone decided ("username 3–20 chars" = a product decision).
2. **Conditionals on domain state** — `if (order.status === 'SHIPPED')`
   gates encode the allowed lifecycle. Collect them into a state machine.
3. **Magic numbers and constants** — thresholds, limits, quotas, timeouts
   users can feel, fee percentages, tier boundaries. Each is a policy;
   name it, don't just note the value.
4. **Permission/authorization checks** — the role→capability matrix IS the
   business's model of who its users are. Reconstruct the full matrix.
5. **Pricing, billing, discount, tax, and fee code** — highest-value business
   logic in any commercial repo. Trace every branch; these are the rules the
   company literally makes money from.
6. **Scheduled jobs' cadence and cutoffs** — "runs at 02:00 daily",
   "expires after 90 days", billing cycles, reconciliation windows. Cadence
   is a business decision about acceptable staleness.
7. **State machines and status enums** — the enum values are the business's
   vocabulary for a process; the allowed transitions are the process itself.
8. **Feature flags and config toggles** — each flag is an unresolved business
   question or a staged rollout. Flag names + default values tell you what
   the business is currently betting on.
9. **Error messages and user-facing copy** — often the clearest statement of
   a rule in the whole repo, written for humans ("You can only have 3 active
   projects on the Starter plan").
10. **Tests** — test names are the closest thing to a spec most repos have.
    `it('rejects refund after 30 days')` is a business rule, verbatim.
11. **Migrations** — a column added, backfilled, then made non-null tells the
    story of a policy becoming mandatory on a specific date.
12. **Integrations with external systems** — what the business outsources
    (payments, KYC, email, tax) reveals its operating model and compliance
    posture.

## Extraction commands (adapt per stack)

```bash
# Magic numbers/thresholds that smell like policy
grep -rn "MAX_\|MIN_\|LIMIT\|THRESHOLD\|_DAYS\|_HOURS\|EXPIR\|QUOTA\|FEE\|RATE\|DISCOUNT\|TIER" --include='*.<ext>' | grep -v test | head -60

# Status/lifecycle vocabulary
grep -rn "enum .*Status\|STATUS\|State =\|_STATE" --include='*.<ext>' | head -40

# Permission matrix
grep -rn "hasRole\|can(\|authorize\|@PreAuthorize\|permission\|isAdmin\|scope" --include='*.<ext>' | head -60

# Money
grep -rniE "price|amount|currency|invoice|charge|refund|tax|coupon|subscription|billing|proration" --include='*.<ext>' -l | head -30

# Rules stated in human language
grep -rniE "you (can|cannot|must)|not allowed|only .* can|maximum of|at least" --include='*.<ext>' | head -40

# Tests as spec
grep -rn "it(\|test(\|def test_\|@Test" -A0 --include='*<testglob>*' | grep -iE "should|when|rejects|allows|only" | head -60

# Feature flags
grep -rniE "featureflag|isEnabled|launchdarkly|unleash|flagsmith|toggle" --include='*.<ext>' | head -30
```

## From rules to a business model

Once rules are collected, assemble upward — a list of rules isn't
understanding. Build:

1. **Actors** — every distinct user/system type the code recognizes (from the
   permission matrix, user types, API clients, service accounts). For each:
   what they can do, what they cannot, and what they care about.
2. **Capabilities** — the things the product lets actors accomplish, stated as
   business outcomes, not endpoints. ("Merchant issues a partial refund", not
   `POST /refunds`.) Map each capability to the flows that implement it.
3. **Lifecycles** — for each core entity, the state machine: states, allowed
   transitions, who/what triggers each, and what's irreversible. Irreversible
   transitions are where business risk concentrates.
4. **Value model** — how the business makes/keeps money or delivers value:
   pricing tiers, limits per tier, what's metered, what's free, where revenue
   is recognized. If non-commercial: what the success metric is (uptime,
   throughput, accuracy) and where it's measured.
5. **Constraints the business operates under** — compliance (GDPR/PCI/HIPAA
   signals: consent flows, data retention jobs, PII encryption, audit logs),
   contractual SLAs, regulatory cutoffs, partner requirements.
6. **Bounded contexts** — where the same word means different things
   ("Order" in fulfillment vs billing). Each boundary is a place where teams,
   or eras of the product, met.

## Rule cataloging format

Every extracted rule gets a stable ID so flow traces and risk entries can
cite it:

```
BR-014 | Refund window
Statement: A refund may be issued only within 30 days of order delivery.
Type: Policy
Enforced at: services/refund.ts:88 (guard) + DB check constraint (none)
Discovered via: conditional + test name "rejects refund after 30 days"
Confidence: [TRACED]
Exceptions: admin override path (services/admin/refund.ts:41) bypasses the
  check entirely — no audit log. → RISK-07
Owner question: is 30 days contractual or a product default?
```

Rules with **inconsistent enforcement** (checked in one path, not another)
are the single most valuable finding this whole skill produces — they are
live bugs or undocumented intentional exceptions. Every one goes to the risk
register.

## Questions the business layer must be able to answer

If your artifacts can't answer these, the business work isn't done:
- Who pays for this system, and what do they pay for?
- What is the one flow that, if broken for an hour, costs the most?
- What can a user do that is irreversible?
- Which rules would require legal/compliance sign-off to change?
- Where does the same domain word mean two different things?
- What does the product promise that the code doesn't actually enforce?