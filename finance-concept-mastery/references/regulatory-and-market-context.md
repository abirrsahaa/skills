# Regulatory & Market Context — dated, current-as-of facts

This domain moves fast enough that stating a regime fact without a date is
misleading within a year. Every fact below carries the date it was true as
of, and this whole file carries a **freshness stamp** the skill should
update on re-run. When Phase 6 writes `05-regulatory-and-market-context.md`,
re-verify anything load-bearing with a fresh search rather than assuming
this file has stayed current — treat everything here as "true as of the
stamp," not "evergreen."

**Freshness stamp: verified via web search, July 2026.**

---

## Settlement cycles (T+1 / T+2)

- **United States, Canada, Mexico** — moved from T+2 to **T+1 for equities,
  corporate bonds, and ETFs on 28 May 2024**. This is now the live, settled
  regime — code assuming T+2 for US equities is out of date.
- **India** — moved to T+1 in 2023, and has since introduced a **voluntary
  T+0** settlement option, with same-day settlement being explored further.
  India is ahead of the US/EU on this transition.
- **Chile, Colombia, Peru** — confirmed to transition to T+1 in Q2 2027.
- **European Union, United Kingdom, Switzerland** — **agreed transition date
  of 11 October 2027** for T+1, still on **T+2 today**. This is a
  coordinated cross-jurisdiction move (UK's Accelerated Settlement
  Taskforce, EU's CSDR amendment, Swiss alignment via swissSPTC). A UK/EU
  industry-wide testing readiness target of January 2027 has been set.
  **Practical implication for code review:** any settlement/collateral/FX-
  funding logic in a repo touching European securities should currently
  assume T+2 and needs a clearly flagged T+1 migration path for October
  2027 — this is exactly the kind of assumption a generalist developer
  would not think to question without this context.
- Shortening settlement cycles is intended to reduce counterparty/credit
  risk and improve capital utilization, but introduces new operational
  strain — particularly FX funding timing (less time to source currency
  for cross-border settlement) and ETFs holding underlying assets in
  different time zones (so-called "T-1 funds" facing valuation timing
  mismatches).

## Reference rates: the LIBOR → risk-free-rate (RFR) transition

- **LIBOR is fully and permanently discontinued.** The last LIBOR panel
  (USD) ended 30 June 2023. A temporary "synthetic" USD LIBOR (1-, 3-, and
  6-month tenors, calculated as CME Term SOFR + a fixed ISDA spread
  adjustment) bridged legacy contracts without proper fallback language
  until it too permanently ceased publication on **30 September 2024**. As
  of that date, **all 35 LIBOR settings across all currencies have
  permanently ceased** — there is no LIBOR left in any form.
- **SOFR (Secured Overnight Financing Rate)** is now the standard USD
  reference rate, replacing USD LIBOR. Regulators (ARRC, FCA) have
  explicitly discouraged wide use of "credit-sensitive rates" as LIBOR
  substitutes, citing the same financial-stability concerns LIBOR itself
  raised.
- Non-USD equivalents are already in place: **SONIA** (GBP), **€STR** (EUR),
  replacing GBP/EUR LIBOR and EURIBOR-adjacent conventions respectively.
- **Practical implication for code review:** any code, config, or comment
  still referencing "LIBOR" as a live rate is either (a) legacy code
  handling historical contracts predating the transition, or (b) a bug/
  staleness — this needs a `[UNKNOWN]`/risk-register flag, not a quiet
  assumption that it's fine. Term SOFR (CME) is the term-rate analog to
  LIBOR's forward-looking tenors, but ARRC guidance recommends *limited*
  scope of use for it versus overnight SOFR compounded in arrears.

## Basel III Endgame (bank regulatory capital)

- The "Endgame" refers to the final, long-delayed piece of the Basel III
  post-2008 capital framework (the Basel Committee's 2017 revisions),
  covering credit risk, operational risk, market risk (FRTB), and the
  output floor.
- **United States:** the original 2023 US proposal (which could have raised
  large-bank capital requirements materially) drew heavy industry pushback
  and was never finalized. On **19 March 2026**, the Federal Reserve, OCC,
  and FDIC jointly **rescinded the 2023 proposal and issued a new,
  substantially more capital-neutral re-proposal** (three separate NPRs
  covering the Basel III Endgame itself, a standardized approach, and a
  revised G-SIB surcharge framework). The FRB board voted 6–1 to advance it.
  The **comment period closed 18 June 2026**. Overall industry capital
  levels are expected to **decrease modestly** relative to current rules if
  finalized as proposed — a reversal of the 2023 proposal's direction.
  A notable structural change: removal of the **Internal Loss Multiplier**
  from the operational-risk capital calculation, meaning a bank's own
  historical loss record will **no longer scale** its operational risk
  capital charge (effectively fixed at 1.0) — a major simplification versus
  the original Basel text.
- **European Union:** postponed its FRTB (market risk) implementation
  timeline twice — first to 1 January 2026, then to **1 January 2027** —
  citing international level-playing-field concerns, with temporary Level 2
  capital-impact mitigations under discussion for 2027–2029.
- **United Kingdom:** the PRA finalized the broader "Basel 3.1" package for
  **1 January 2027**, but separately deferred the FRTB Internal Model
  Approach specifically to **1 January 2028**, citing implementation
  complexity.
- **Practical implication for code review:** Basel implementation is now
  materially **jurisdiction-specific and non-synchronized** — a regulatory-
  capital module built assuming one global timeline/calibration is already
  wrong; check which jurisdiction's rules a given calculation actually
  targets, and treat the exact numeric calibrations (risk weights, floors)
  as provisional given the US re-proposal is still in a comment/finalization
  process as of this stamp.

## What to re-verify on every skill re-run

Because this domain moves fast, Phase 6 should re-search rather than trust
this file blindly for anything that affects a repo's actual behavior:
finalization status of the US Basel III re-proposal (comment period closed
June 2026 — a final rule may exist by the time this runs again), any
further T+1 testing/timeline updates for EU/UK/Switzerland, and any newly
announced RFR or benchmark-rate developments outside USD/GBP/EUR. Treat
this file as a strong prior, not a permanent source of truth.