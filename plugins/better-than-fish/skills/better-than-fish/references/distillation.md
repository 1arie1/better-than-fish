# Distillation

The practice that makes the system work. Without it, the journal grows
indefinitely and nobody reads it.

## Cadence

- **Weekly digest:** end of each calendar week, or when the user types
  "digest" / "digest weekly". Read all `journal/YYYY-MM/` entries from the
  past 7 days. Write `weekly/YYYY-Www.md`.
- **Monthly digest:** first Monday of each new month, or "digest monthly".
  Read the 4 weekly digests of the prior month. Write `monthly/YYYY-MM.md`.
- **Durable promotion:** any time during a digest. If a finding has been
  referenced in multiple digests *and* withstood scrutiny, propose moving
  it to `durable/`.

The agent should remind the user that a digest is overdue if the journal
has > 7 entries since the last weekly. One line at session start: "Last
weekly digest: 2026-W17. Journal has 11 unread entries. Propose digest?"

## What goes in a weekly digest

A weekly is **not** a chronological list of journal entries. That's a
journal index, not a digest. A weekly is an interpretation:

- **What did we learn?** Two or three durable insights from the week.
  Each one cites the journal entry that supports it.
- **What changed?** `[HYP]`s that got promoted, demoted, or superseded.
  Brief explanation of why.
- **What's still open?** `[OPEN]`s that are now actionable, plus any new
  loose ends parked during the week.
- **What's next?** One or two top-of-mind items the user might pick up.

A good weekly is one page. If it's longer, you're listing instead of
distilling.

### Example shape

```markdown
# Week 2026-W17

## Promoted insights

[FACT → durable/grobner-quotas.md] Gröbner's default `gr_q=10` and
`grobner_eqs_growth=10` are too conservative for Lyapunov-style benchmarks.
Verified across Stroeder Choose (4/8 → 6/8 closure with `gr_q=50`),
no overhead on already-closing seeds.
  → 2026-04-26-grobner-q50-stroeder.md

[FACT → durable/j-vars-unstable.md] j-variable IDs are not stable across
nlsat invocations. Use `-tr:nra` x-form for stable nlsat-question
fingerprints; `-tr:nla_solver` for lemma flow.
  → 2026-04-25-jvar-renumbering.md

## Superseded

[SUPERSEDED] Earlier framing "constraint-pool substitution is missing" —
not actually missing. Gröbner's Linear Propagation post-processing already
adds substituted linear rows to the Simplex tableau. The issue was
quota-bound, not pipeline-bound.
  → corrected by 2026-04-26-paper-clarifies-layering.md

## Loose ends added

- Probe AProVE family with sandwich+mccormick (parked: G50 finding
  prioritized; ~30 min to verify)
- Wide eval of patched-pseudo-lin + G50 vs master-1472

## What's next

The G50 finding is real on Stroeder Choose. Next experiments:
1. One non-Lyapunov benchmark to test generality
2. Wide eval if (1) confirms
```

Notice what's NOT in the digest:
- Chronological listing of every experiment
- Specific numbers from each run (those live in journal entries)
- Commands or trace excerpts (also journal-only)
- Speculation that hasn't graduated to a `[HYP]` worth tracking

## What goes in a monthly digest

A monthly is the durable interpretation of the month's work. Same shape as
weekly but coarser. Promote whatever appeared in 2+ weekly digests AND has
operational consequences.

By the time something reaches `monthly/`, it should be close to
`durable/`-quality.

## Promotion criteria

A claim moves up when it satisfies:

| from → to | criterion |
|---|---|
| journal `[OBS]` → weekly | referenced or built upon during the week |
| weekly → monthly | referenced in 2+ weeklies; still load-bearing |
| journal `[OBS]` (or weekly) → **`empirical/` `[EMP]`** | observed across **multiple investigations** without contradiction; has clear operational shape |
| `empirical/` `[EMP]` → `durable/` `[FACT]` | becomes source-grounded (paper §, file:line proof) **or** time-stable across z3 versions and benchmark families |
| monthly → `durable/` | (alternative path) survived a month of work with no contradiction; has clear operational consequence; either source-grounded or codified as a recorded design pattern |

The criteria are **necessary but not sufficient**. The agent should
propose; the user decides. Don't promote silently.

### empirical/ vs durable/ — the key distinction

A claim earns `durable/`-status when it's **time-invariant** in some
form:
- A math fact (McCormick envelope properties)
- A paper claim (the z3 NLA waterfall structure)
- A source-grounded API (`force_phase` exists at `smt_context.h:1097`)
- A recorded design pattern (the experimental ineq.prefer() shape)

A claim is `empirical/` when it's **stable but version-bound**:
- "Ramped Gröbner closes more on Lyapunov-style benchmarks (z3@b0956429)"
- "PR9391 features show universal trade-off shape on QF_NIA_small (z3@b0956429)"
- "tangents.box_corners is the only PR feature net-positive on QF_NIA_small"

When the underlying code changes, an `empirical/` note may need
re-verification. A `[FACT]` doesn't (the API may move, the location
file:line may need updating, but the conceptual fact remains).

## What the digest excludes

- **Reproducible setup**: commands, env vars, file paths. These belong in
  the journal entry.
- **Quotes from papers**: cite, don't transcribe.
- **Single-data-point findings**: a result on one benchmark is a `[HYP]`
  candidate, not a digest item, until it's verified more broadly.
- **Commentary about the writing process itself**: "we explored several
  framings" — irrelevant to durable understanding.

## Quality check

After writing a weekly, run this filter on each item:

> Could a future agent, six months from now, take this insight and apply
> it without re-doing the work?

If no, the item isn't ready. Demote it back to a journal entry or rewrite
it more concretely.
