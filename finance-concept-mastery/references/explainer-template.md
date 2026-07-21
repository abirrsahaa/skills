# Explainer Template — required structure for every foundation/concept file

A concept file that only defines its term has not met the skill's depth
bar. Every file in `02-foundations/` and `03-concepts/` uses this structure
in full. Foundation files may trim sections 5–7 if the concept is genuinely
absent from the repo's code (primer-depth is acceptable ONLY when Phase 1
found no implementation of it — see the extraction playbook's evidence-type
distinction); concept files that ARE implemented in the repo get every
section.

```markdown
---
concept: <NN>-<name>
layer: <taxonomy layer #>
status: <Foundation | Implemented-in-repo | Referenced-only>
confidence: <tag or Mixed>
prerequisites: [<links to other concept files, by number+name>]
feeds-into: [<concepts downstream of this one, filled in as discovered>]
---

# <Concept Name>

## 1. The one-sentence essence
What this concept is, in a sentence a smart non-specialist could repeat
back correctly. This is the sentence that goes in the glossary too.

## 2. Prerequisites — what you need first
Explicit list, each with ONE sentence on why it's needed here specifically
(not just "see concept X") — e.g. "Discounting (02-foundations/01), because
DV01 is a derivative of the bond-pricing formula with respect to yield, and
bond pricing requires discounting cash flows."

## 3. The market mechanics
What this concept means and why it exists *in the market*, before any
formula. Who uses it, on which desk, for what decision. This is the part a
formula alone never communicates — the "why does anyone care."

## 4. The formula / model (derived, not asserted)
State the formula, define every variable, and sketch the derivation or the
intuition for where it comes from (not a proof, but not "just trust the
formula" either). If multiple conventions exist (e.g. different day-count
bases), state which one(s) and why it matters which is used.

## 5. How this repo implements it — `[TRACED]` or `[INFERRED]`, always tagged
Code anchor(s): file/function/class. What matches the textbook version
exactly, what diverges, and — if it diverges — the most likely reason
(performance, a specific regulatory requirement, a legacy convention, or
just technical debt — say which, and mark your confidence). If genuinely
`Referenced-only` (concept needed for understanding but not implemented
here), say so explicitly instead of forcing a code anchor that doesn't
exist.

## 6. Worked example
A fully worked numeric example, computed by hand, from realistic inputs to
a specific numeric output. This is the single most load-bearing section —
a reader who can reproduce this calculation independently has genuinely
learned the concept, not just recognized its name.

## 7. Common misconceptions & pitfalls
The 2–4 ways people (including experienced developers new to this specific
desk/product) get this wrong. Where relevant, name the specific bug pattern
this produces in code (e.g. "using ACT/365 where the contract specifies
30/360 understates accrued interest by roughly X%").

## 8. Regulatory / accounting tie-in (if any)
Which regulation, capital rule, or accounting standard depends on getting
this right, and what the consequence of getting it wrong is (capital
miscalculation, reporting breach, audit finding, mis-hedged P&L). Omit this
section only if genuinely no regulatory/accounting angle exists.

## 9. What this feeds into
Which downstream concepts in this repo's concept map depend on this one,
and — one sentence each — why. This is what makes the map a graph a reader
can navigate in either direction, not just a linear reading list.
```

## Notes on filling this in well

- **Section 6 is not optional and not decorative.** A concept file without
  a worked numeric example is an unfinished concept file, full stop.
- **Prerequisites link, they don't repeat.** If duration is a prerequisite,
  don't re-explain duration here — link to its file. Repetition across
  files is exactly what makes a docs suite rot; the dependency graph exists
  so nothing has to be said twice.
- **"Feeds into" is filled in retrospectively** as later concepts are
  written — when you write concept 14 and it depends on concept 6, go back
  and add the link to concept 6's "feeds into" section so the graph is
  navigable from either end. This is a real edit to an earlier file, not
  optional bookkeeping.
- **Confidence tagging in section 5 is the section most likely to be
  rushed — don't rush it.** The difference between "this repo computes
  DV01 by full reprice (bump-and-reprice) rather than the closed-form
  duration approximation, confirmed by reading the function" `[TRACED]`
  and "this repo appears to use bump-and-reprice based on the function name"
  `[INFERRED]` is the difference between documentation and a guess wearing
  documentation's clothes.