# Finance Domain Taxonomy — the master concept map

This is the checklist and dependency authority for Phase 1–2. It organizes
capital-markets domain knowledge into layers ordered by prerequisite depth:
each layer generally depends on the ones above it. Use it to (a) recognize
what a piece of code is really referring to, and (b) place every found
concept at its correct point in the topological order. It is not exhaustive
of every product variant in existence — it's the backbone a generalist
developer needs to reach genuine fluency across a typical capital-markets
codebase (trading, risk, settlement, regulatory reporting, treasury).

Format per entry: **Concept** — one-line essence — *prerequisites*.
Prerequisites listed are direct dependencies only (their own prerequisites
are listed under their own entry) — this is what makes it a graph, not a
flat list; Phase 2 walks the chain transitively.

---

## Layer 0 — Mathematical & statistical foundations

Needed before almost anything else in Layers 2–4 makes sense.

- **Time value of money** — a dollar today is worth more than a dollar
  later; the basis for all discounting. *prerequisites: none*
- **Compounding & discounting** — simple vs compound interest, continuous
  compounding (`e^(rt)`), present value / future value. *prereq: time value
  of money*
- **Day-count conventions** — how the fraction of a year between two dates
  is measured (30/360, ACT/360, ACT/365, ACT/ACT) — the single most common
  source of off-by-a-few-basis-points bugs in fixed income code. *prereq:
  none, but meaningless without compounding*
- **Probability distributions** — normal, lognormal, fat tails/kurtosis;
  why lognormal (not normal) models asset prices (prices can't go negative).
  *prereq: none*
- **Volatility & standard deviation** — dispersion of returns; annualizing
  a daily/monthly vol (`σ_annual = σ_daily × √252`). *prereq: probability
  distributions*
- **Correlation & covariance** — how two variables move together; the basis
  of portfolio diversification and multi-asset risk. *prereq: probability
  distributions*
- **Regression & statistical estimation basics** — beta, R², least squares
  — used throughout risk factor models and hedge ratios. *prereq:
  correlation*
- **Stochastic processes primer** — random walk, Brownian motion, geometric
  Brownian motion (GBM) — the assumed process behind most derivatives
  pricing models. *prereq: probability distributions, volatility*

## Layer 1 — Instrument & market mechanics

The vocabulary of what is actually being traded.

- **Equities** — shares, par value, dividends, voting rights, market cap.
  *prereq: none*
- **Corporate actions** — splits, reverse splits, mergers, spin-offs, rights
  issues — events that change an equity position without a trade. *prereq:
  equities*
- **Fixed income / bonds** — face value, coupon, maturity, issuer, seniority.
  *prereq: time value of money*
- **Accrued interest & clean vs. dirty price** — a bond's quoted ("clean")
  price excludes interest accrued since the last coupon; the "dirty" price
  (what actually settles) includes it. *prereq: day-count conventions, bond
  mechanics*
- **Yield** — current yield, yield-to-maturity (YTM), yield-to-call; the
  discount rate that equates a bond's price to its discounted cash flows.
  *prereq: discounting, bond mechanics*
- **FX (foreign exchange)** — spot rate, base/quote currency convention,
  pips, cross rates. *prereq: none*
- **FX forwards & swaps** — locking a future exchange rate; covered interest
  rate parity (the forward rate is implied by the interest-rate differential
  between the two currencies, not a market guess). *prereq: FX, discounting,
  time value of money*
- **Futures** — exchange-traded, standardized, daily mark-to-market
  ("variation margin"), margining via a clearinghouse. *prereq: forwards
  concept below, or introduce together*
- **Forwards** — OTC, bilateral, customizable, settled only at maturity (no
  daily margining unless collateralized). *prereq: time value of money*
- **Options** — the right (not obligation) to buy (call) or sell (put) at a
  strike price; premium, moneyness (ITM/ATM/OTM), exercise style
  (American/European/Bermudan). *prereq: probability distributions (to reason
  about payoff uncertainty)*
- **Swaps — interest rate (IRS)** — exchanging fixed for floating interest
  payments on a notional; the workhorse derivative of fixed income desks.
  *prereq: discounting, floating-rate mechanics*
- **Swaps — credit default swap (CDS)** — insurance-like contract paying out
  on a credit event; premium leg vs. protection leg. *prereq: probability of
  default (Layer 4), discounting*
- **Repo (repurchase agreement)** — a collateralized short-term loan
  structured as a sale + agreement to repurchase; the plumbing of short-term
  funding markets. *prereq: bonds/collateral, time value of money*
- **Structured products** — payoffs built from combinations of the above
  (e.g. a note whose return depends on an equity index with capped
  upside); understood by decomposing into the underlying instruments.
  *prereq: options, bonds*
- **Reference/benchmark rates** — the rate an instrument's floating leg or
  discounting is tied to (SOFR, €STR, SONIA, and their legacy predecessor
  LIBOR). See `regulatory-and-market-context.md` for the current-state
  facts — this area changed materially in 2023–2024 and code written before
  then may still assume LIBOR-era conventions. *prereq: none, but only
  meaningful with IRS/discounting*

## Layer 2 — Valuation & pricing

How a price or value gets attached to Layer 1 instruments.

- **Yield curve construction** — building a curve of rates across
  maturities from observable instruments (deposits, futures, swaps);
  bootstrapping. *prereq: discounting, IRS*
- **Discount factors & curve interpolation** — converting a yield curve into
  the discount factors actually used to present-value cash flows; choice of
  interpolation method affects pricing at the margin. *prereq: yield curve
  construction*
- **OIS discounting** — post-2008 standard of discounting collateralized
  cash flows using the overnight-index-swap curve rather than LIBOR/IBOR,
  reflecting the true cost of funding a collateralized position. *prereq:
  discount factors, repo/collateral*
- **Bond pricing** — present value of coupon + principal cash flows at the
  appropriate yield/curve. *prereq: discounting, day-count, accrued
  interest*
- **Duration** (Macaulay & modified) — weighted-average time to receive cash
  flows; modified duration ≈ % price change per 1% yield change. *prereq:
  bond pricing*
- **Convexity** — the curvature correction to duration's linear
  approximation; matters more for large yield moves. *prereq: duration*
- **DV01 / PV01** — dollar value of a 1 basis-point yield move; the
  practical, tradeable form of duration risk managers actually quote.
  *prereq: duration, bond pricing*
- **Credit spread & z-spread** — the extra yield over the risk-free curve
  compensating for default risk; z-spread is the constant spread added to
  every point on the curve to match a bond's market price. *prereq: yield
  curve, bond pricing, probability of default (Layer 4)*
- **Black-Scholes-Merton model** — closed-form option pricing under
  lognormal price assumptions; inputs (spot, strike, rate, vol, time) and
  what each drives. *prereq: GBM, discounting, options*
- **The Greeks** — Delta (price sensitivity to underlying), Gamma (rate of
  change of Delta), Vega (sensitivity to volatility), Theta (time decay),
  Rho (rate sensitivity) — the risk managers actually hedge against.
  *prereq: Black-Scholes, calculus intuition (rates of change)*
- **Implied volatility & the vol surface/smile** — inverting the pricing
  model to back out the market's volatility assumption; the "smile" is the
  empirical pattern that implied vol differs by strike, contradicting
  Black-Scholes' flat-vol assumption. *prereq: Black-Scholes, Greeks*
- **Swap valuation** — present-valuing the fixed and floating legs
  separately using the appropriate curves; the swap's value is the
  difference. *prereq: discount factors, OIS discounting, IRS*
- **Mark-to-market (MTM) vs. mark-to-model** — MTM uses observable market
  prices; mark-to-model is used when no liquid market price exists (uses a
  pricing model with model risk). *prereq: whichever pricing model applies*
- **P&L attribution** — decomposing a position's daily P&L into causes: new
  trades, time decay (theta), market moves (delta/gamma/vega), and
  "unexplained" residual — a risk-management and audit staple. *prereq: the
  Greeks, MTM*

## Layer 3 — Trade lifecycle & operations

The plumbing that turns a trade decision into a settled, reconciled
position — where most of a repo's non-pricing business logic actually lives.

- **Front / middle / back office** — the organizational split: front office
  trades, middle office manages risk/valuation/confirmation, back office
  settles/reconciles/reports. Most codebases map cleanly onto one of these.
  *prereq: none*
- **Trade capture & booking** — recording the economic terms of a trade
  (notional, trade date, value/settlement date, counterparty, instrument
  reference) into a system of record. *prereq: relevant instrument (Layer
  1)*
- **Confirmation & affirmation** — the two counterparties agreeing the
  trade's economic terms match before it moves downstream; a mismatch here
  is a "trade break." *prereq: trade capture*
- **Clearing & CCPs (central counterparties)** — a CCP interposes itself
  between both sides of a trade (novation), becoming the buyer to every
  seller and the seller to every buyer; this is what "cleared" vs.
  "bilateral/OTC" means and why clearing reduces counterparty risk.
  *prereq: trade capture, counterparty credit risk concept (Layer 4)*
- **Netting** — combining multiple offsetting obligations between the same
  parties into a single net obligation; close-out netting activates on
  default. Central to why counterparty exposure is smaller than gross
  notional would suggest. *prereq: clearing/CCPs*
- **Settlement & settlement cycles (T+1/T+2)** — the actual exchange of
  cash for securities on the value date. **Regime fact (dated) — see
  regulatory-and-market-context.md:** US equities moved to T+1 in May 2024;
  EU/UK/Switzerland are moving to T+1 on 11 October 2027, still on T+2
  until then. *prereq: trade capture, custody*
- **Custody & safekeeping** — the holding of securities by a custodian on
  behalf of the beneficial owner; nominee vs. direct holding. *prereq:
  settlement*
- **Corporate action processing** — operationally applying Layer-1 corporate
  actions to every affected position (entitlement calculation, election
  deadlines for elective actions). *prereq: corporate actions (Layer 1),
  custody*
- **Reconciliation** — comparing internal records against external sources
  (custodian, exchange, counterparty) to catch breaks; nostro/vostro
  account reconciliation for cash. *prereq: settlement, custody*
- **Reference data** — the static identifiers and attributes that everything
  else keys off: ISIN, CUSIP, SEDOL (security identifiers), LEI (Legal
  Entity Identifier for counterparties), instrument static data. Bad
  reference data is one of the most common root causes of trade breaks.
  *prereq: none, but load-bearing under everything else in this layer*
- **Collateral management operations** — the operational process of calling,
  posting, and reconciling collateral (distinct from the risk *concept* of
  margin in Layer 4). *prereq: netting, margin (Layer 4)*

## Layer 4 — Risk management

The frameworks for measuring and controlling exposure to loss.

- **Market risk & VaR (Value at Risk)** — the loss a portfolio would not
  expect to exceed over a given horizon at a given confidence level (e.g.
  "1-day 99% VaR"); computed historically, parametrically, or via Monte
  Carlo. *prereq: volatility, correlation, probability distributions*
- **Expected Shortfall (CVaR)** — the average loss *given* that the VaR
  threshold is breached; addresses VaR's blindness to tail severity beyond
  the threshold. Increasingly the regulatory-preferred market-risk metric
  under FRTB. *prereq: VaR*
- **Stress testing & scenario analysis** — applying historical or
  hypothetical extreme scenarios to a portfolio, outside the statistical
  assumptions VaR relies on. *prereq: VaR, sensitivities (Greeks)*
- **Probability of Default (PD)** — the likelihood a counterparty/issuer
  defaults over a horizon; derived from credit ratings, market spreads (CDS-
  implied PD), or internal models. *prereq: probability distributions*
- **Loss Given Default (LGD)** and **Exposure at Default (EAD)** — the other
  two components of expected loss (`EL = PD × LGD × EAD`); LGD is `1 -
  recovery rate`. *prereq: PD*
- **Credit risk / expected & unexpected loss** — expected loss is provisioned
  for (accounting); unexpected loss is what regulatory capital exists to
  absorb. *prereq: PD, LGD, EAD*
- **Counterparty Credit Risk (CCR) & CVA** — Credit Valuation Adjustment: the
  market price of counterparty default risk on a derivative, computed from
  the exposure profile (how much you're owed over time) times the
  counterparty's PD. DVA is the mirror (your own default risk, from the
  counterparty's perspective); FVA is the funding cost adjustment. *prereq:
  PD, discounting, netting, derivative pricing (Layer 2)*
- **Wrong-way risk** — when exposure to a counterparty increases exactly
  when that counterparty's credit quality worsens (the correlation nobody
  wants). *prereq: CVA*
- **Collateral & margin (the risk concept)** — Initial Margin (IM, collateral
  posted to cover potential future exposure) vs. Variation Margin (VM,
  collateral posted to cover current mark-to-market exposure); haircuts
  (a discount applied to collateral value for its own risk). *prereq: MTM,
  CVA*
- **ISDA Master Agreement & CSA (Credit Support Annex)** — the standard
  legal contract governing OTC derivatives between two parties and the
  annex specifying collateral terms (threshold, minimum transfer amount,
  eligible collateral). *prereq: netting, margin*
- **ISDA SIMM (Standard Initial Margin Model)** — the industry-standard
  methodology for computing initial margin on non-cleared derivatives, built
  from risk-factor sensitivities (delta, vega, curvature) bucketed by asset
  class and tenor. *prereq: the Greeks, margin*
- **Liquidity risk** — funding liquidity risk (can the firm meet its cash
  obligations) vs. market liquidity risk (can a position be exited without
  moving the price); LCR (Liquidity Coverage Ratio) and NSFR (Net Stable
  Funding Ratio) are the Basel III metrics for the former. *prereq: none
  directly, but meaningless without balance-sheet/funding context*
- **Operational risk** — loss from failed processes, people, systems, or
  external events; measured under Basel via a Business Indicator-based
  approach. **Regime fact (dated):** the March 2026 US Basel III re-proposal
  removes the Internal Loss Multiplier (a firm's own loss history no longer
  scales its capital charge) — see regulatory-and-market-context.md.
  *prereq: none*
- **Model risk** — the risk that a pricing/risk model itself is wrong or
  misapplied; governed by model validation and independent price
  verification (IPV) functions. *prereq: whichever model is in question*

## Layer 5 — Regulatory capital & accounting

How risk gets translated into required capital and reported earnings.

- **Risk-Weighted Assets (RWA)** — an exposure's notional adjusted by a risk
  weight reflecting its riskiness; the denominator regulatory capital ratios
  are measured against. *prereq: credit risk (PD/LGD/EAD), market risk*
- **Regulatory capital tiers** — CET1 (Common Equity Tier 1), Tier 1, Tier 2;
  the quality hierarchy of capital a bank must hold. *prereq: RWA*
- **Basel III / Basel III Endgame** — the international bank-capital
  framework; "Endgame" refers to the final, still-being-finalized piece of
  Basel III implementation. **Regime fact (dated) — see
  regulatory-and-market-context.md:** as of mid-2026 US regulators
  re-proposed the framework in March 2026 with a comment period ending June
  18, 2026, materially softer than the original 2023 proposal; the EU/UK are
  on separate, later timelines (FRTB deferred to 2027–2028 in various
  jurisdictions). *prereq: RWA, capital tiers*
- **Leverage ratio & G-SIB surcharge** — a non-risk-weighted capital backstop,
  and an additional capital requirement for globally systemic banks.
  *prereq: capital tiers*
- **FRTB (Fundamental Review of the Trading Book)** — the Basel market-risk
  capital framework, replacing VaR-based capital with Expected-Shortfall-
  based standardized and internal-model approaches. *prereq: VaR, Expected
  Shortfall, RWA*
- **Fair value hierarchy (Level 1/2/3)** — accounting classification of how
  observable a valuation input is: Level 1 (quoted market price), Level 2
  (observable inputs other than quoted prices), Level 3 (unobservable,
  model-driven). Directly tied to mark-to-market vs. mark-to-model.
  *prereq: MTM/mark-to-model*
- **IFRS 9 vs. US GAAP CECL** — the two dominant expected-credit-loss
  provisioning regimes; IFRS 9 uses a three-stage model, CECL requires
  lifetime expected loss from origination. *prereq: PD, LGD, EAD, expected
  loss*
- **Hedge accounting** — the accounting treatment allowing a derivative's
  P&L volatility to be matched against the hedged item's, avoiding
  artificial earnings volatility; requires documented hedge effectiveness.
  *prereq: MTM, the instrument being hedged*

## Layer 6 — Regulatory & compliance framework

The rules governing market conduct, reporting, and structure (distinct from
capital rules in Layer 5).

- **Dodd-Frank Act** (US) — post-2008 reform mandating central clearing for
  standardized swaps, swap data reporting, and the Volcker Rule. *prereq:
  clearing/CCPs*
- **Volcker Rule** — restricts US banks' proprietary trading and certain
  fund investments. *prereq: none directly*
- **MiFID II / MiFIR** (EU) — governs best execution, pre/post-trade
  transparency, transaction reporting, and product governance across EU
  markets. *prereq: trade lifecycle basics*
- **EMIR** (EU) — mandates clearing obligations and trade reporting for
  derivatives, parallel to Dodd-Frank's swap provisions. *prereq: clearing,
  derivatives*
- **Market Abuse Regulation (MAR)** — prohibits insider dealing and market
  manipulation; underlies surveillance logic in trading systems. *prereq:
  none*
- **KYC / AML** — Know Your Customer and Anti-Money Laundering checks
  gating onboarding and transaction monitoring. *prereq: none*
- **Trade & transaction reporting** — the operational requirement to report
  trade details to regulators/trade repositories under Dodd-Frank/EMIR/
  MiFIR; often the direct consumer of reference data and trade capture
  fields. *prereq: reference data, trade capture, EMIR/Dodd-Frank/MiFID*

## Layer 7 — Market microstructure & data infrastructure

The technical substrate all of the above runs on.

- **Order types & order book mechanics** — limit/market/stop orders, the
  bid-ask spread, price-time priority matching. *prereq: none*
- **FIX protocol** — the standard messaging protocol for trade orders and
  executions between market participants; field-tag structure (e.g. Tag 55
  = Symbol, Tag 54 = Side). *prereq: order types*
- **FpML** — the XML standard for representing OTC derivative trade
  economics, commonly the wire format behind trade capture/confirmation
  systems. *prereq: relevant derivative (Layer 1)*
- **Benchmark/reference rates infrastructure** — how SOFR/€STR/SONIA are
  published, and the mechanics of term-rate construction (e.g. CME Term
  SOFR) versus the overnight rate itself. *prereq: reference/benchmark
  rates (Layer 1)*
- **Algorithmic & electronic trading basics** — smart order routing, TWAP/
  VWAP execution algorithms, latency sensitivity. *prereq: order book
  mechanics*

---

## Using this taxonomy

1. Every concept found in Phase 1 gets matched to exactly one entry above
   (or, if genuinely absent, added as a new entry with its own prerequisite
   analysis — the taxonomy is a checklist, not a cage).
2. Its prerequisites are pulled transitively until you hit entries with no
   unmet prerequisites — that set is the repo-specific foundation layer.
3. The remaining concepts are topologically sorted (every prerequisite
   numbered lower than what depends on it) to produce the numbering for
   `02-foundations/` and `03-concepts/`.
4. Layer numbers above are a strong prior for ordering, not a substitute for
   doing the actual dependency walk per concept — a repo that never touches
   derivatives can skip straight from Layer 1 bonds to Layer 3 settlement
   without ever needing Layer 2's option-pricing machinery.