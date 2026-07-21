# Concept Extraction Playbook

Finance concepts hide in code the same way business rules do (see
repo-mastery's business-logic-extraction if paired), but with a domain-
specific vocabulary. This playbook is the sweep to run in Phase 1, organized
by where terms hide and by taxonomy layer.

## Where finance concepts physically hide

1. **Variable, field, and column names** — the highest-density source.
   Financial codebases are unusually literal in naming (`dv01`, `strike`,
   `notional`, `accrued_interest`) compared to typical business code, which
   makes grep unusually effective here.
2. **Constants and config keys** — day-count conventions, margin thresholds,
   curve names, benchmark rate identifiers are almost always named
   constants or config entries, not magic numbers — easier to find than
   typical business policy constants.
3. **Function/method names** — `computeDV01`, `bootstrapCurve`,
   `calculateCVA`, `applyHaircut` name the concept directly.
4. **Comments and docstrings** — quant and risk code is often *more*
   commented than typical business code, because the author knew the next
   reader might not have the math background — mine these heavily.
5. **Test names and fixtures** — `test_accrued_interest_act_360`,
   `test_var_99_confidence` state both the concept and the specific
   convention/parameter used.
6. **Schema/API field names** — trade capture schemas, FpML/FIX field
   mappings, regulatory report layouts are dense with named concepts.
7. **Embedded formulas** — literal mathematical expressions in code
   (`sqrt(t) * vol`, `notional * (1 - exp(-r*t))`) are a formula waiting to
   be matched against `finance-domain-taxonomy.md` and written up in
   `04-formula-reference.md`.
8. **External library/model imports** — imports of QuantLib, a Black-
   Scholes library, an ISDA SIMM calculator, a curve-bootstrapping package
   name the concept even before you read a line of the calling code.
9. **Regulatory report generators** — modules named after specific reports
   (CFTC swap data reporting, EMIR trade reporting, FRTB capital
   calculation) point directly at Layer 5/6 concepts.
10. **Data model relationships** — a trade linked to a "netting set" linked
    to a "CSA" IS the Layer 3/4 collateral concept graph made concrete;
    reverse-engineer the concept graph from the foreign keys.

## Grep sweep (adapt `<ext>` to the repo's language)

```bash
# Fixed income & discounting
grep -rniE "day.?count|30_360|act_360|act_365|accru|dirty.?price|clean.?price|ytm|yield.?to.?maturity" --include='*.<ext>' -l

# Rates & curves
grep -rniE "discount.?factor|bootstrap|yield.?curve|ois|forward.?rate|zero.?rate|interpolat" --include='*.<ext>' -l

# Benchmarks (watch for LIBOR-era code that may be stale — see regulatory-and-market-context.md)
grep -rniE "libor|sofr|sonia|estr|euribor|term.?rate|risk.?free.?rate|\bcsa\b" --include='*.<ext>' -l

# Duration/DV01/sensitivities
grep -rniE "dv01|pv01|duration|convexity|bpv|dollar.?duration" --include='*.<ext>' -l

# Options & Greeks
grep -rniE "black.?scholes|implied.?vol|\bdelta\b|\bgamma\b|\bvega\b|\btheta\b|\brho\b|strike|moneyness|vol.?surface|vol.?smile" --include='*.<ext>' -l

# Credit risk
grep -rniE "probability.?of.?default|\bpd\b|\blgd\b|\bead\b|expected.?loss|recovery.?rate|credit.?rating|default.?prob" --include='*.<ext>' -l

# Counterparty risk & collateral
grep -rniE "\bcva\b|\bdva\b|\bfva\b|wrong.?way|netting.?set|isda|csa|initial.?margin|variation.?margin|haircut|collateral|\bsimm\b" --include='*.<ext>' -l

# Market risk
grep -rniE "\bvar\b|value.?at.?risk|expected.?shortfall|\bcvar\b|stress.?test|scenario" --include='*.<ext>' -l

# Regulatory capital
grep -rniE "rwa|risk.?weight|cet1|tier.?1|tier.?2|leverage.?ratio|g.?sib|frtb|basel|lcr|nsfr" --include='*.<ext>' -l

# Accounting
grep -rniE "fair.?value|level.?[123]|mark.?to.?market|mark.?to.?model|hedge.?accounting|ifrs.?9|cecl" --include='*.<ext>' -l

# Trade lifecycle & ops
grep -rniE "trade.?date|value.?date|settlement.?date|affirm|confirm|novat|clearing|nostro|vostro|reconcil|isin|cusip|sedol|\blei\b" --include='*.<ext>' -l

# Regulatory/compliance frameworks
grep -rniE "dodd.?frank|volcker|mifid|emir|kyc|\baml\b|market.?abuse|swap.?data.?repository|\bsdr\b" --include='*.<ext>' -l

# Market microstructure
grep -rniE "\bfix\b.*protocol|fpml|order.?book|bid.?ask|twap|vwap|smart.?order.?rout" --include='*.<ext>' -l

# Repo/collateralized funding
grep -rniE "\brepo\b|reverse.?repo|repurchase.?agreement" --include='*.<ext>' -l
```

## After the sweep: classify each hit

For every file/symbol the sweep surfaces, record:
- **Term found** (exact string) and **location** (file:line)
- **Best-guess taxonomy entry** (from `finance-domain-taxonomy.md`)
- **Evidence type**: is this term merely present (name/comment) or does the
  surrounding code actually *implement* the concept's math/logic? This
  distinction drives whether the eventual concept file gets full-depth
  treatment (implemented) or primer-depth treatment (referenced only).

This classified list is the raw input to Phase 2's dependency walk — don't
skip straight to writing explainers before this list exists, or the
prerequisite ordering in `01-concept-map.md` will be built on an incomplete
picture.

## False-friend warning

Some finance terms collide with generic programming vocabulary and will
produce noisy grep results — filter by context, not just the string:
- `delta` (Greek) vs. delta as in "diff"/"change" in unrelated code
- `spread` (credit/bid-ask) vs. spread operator in JS
- `rate` (interest rate) vs. generic rate-limiting code
- `margin` (collateral) vs. CSS/UI margin
- `option` (derivative) vs. a config "option"
- `swap` (derivative) vs. a generic swap/exchange function
- `duration` (bond risk) vs. a generic time-duration variable
Read the surrounding code before classifying — a hit in a UI component or a
rate-limiter is not a finance concept, however tempting the string match.