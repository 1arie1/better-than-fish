# Example: what a notes directory looks like after one week

The minimum viable starter for a new project. Approximate sizes shown.

```
~/<project>/research/
├── CLAUDE.md                                       (60 lines, conventions only)
├── papers/
│   └── primary-reference.pdf                       (source corpus)
├── durable/
│   ├── architecture-overview.md                    (FACT, paper §-cited)
│   ├── core-API.md                                 (FACT, source-grounded)
│   ├── tooling-cheatsheet.md                       (FACT)
│   └── experimental-pattern-X.md                   (FACT, recorded design we tried)
├── empirical/
│   └── claim-Y-on-benchmark-family-Z.md            (EMP, version-stamped)
├── journal/
│   └── 2026-04/
│       ├── 2026-04-22-first-investigation.md       (OBS+HYP)
│       ├── 2026-04-23-correction-on-X.md           (FACT post-correction)
│       ├── 2026-04-25-deep-dive-on-Y.md            (OBS+HYP)
│       └── 2026-04-26-experiment-with-Z.md         (OBS, the headline finding)
├── weekly/
│   └── 2026-W17.md                                 (75 lines, distilled)
├── loose-ends/
│   └── parked.md                                   (4 entries)
└── <concept>/                                      (optional; e.g. tooling/
    ├── README.md                                    that cuts across
    └── ...                                          investigations)
```

## What's in CLAUDE.md

The notes-dir CLAUDE.md is short and project-specific. Example:

```markdown
# Project: Z3 NLA research notes

These notes live at `~/ag/z3/z3-research/`.

The better-than-fish skill governs how to add and maintain notes.
Re-read its SKILL.md for conventions.

## Project-specific triggers

- "ramon" — pull a benchmark run via the lemur ramon design (TODO).
- "trace M_s0" — capture an nla_solver trace for default master, seed 0.

## Code locations to anchor citations

- Worktree: `~/ag/z3/wt-mono-sandwich/`
- NLA core: `src/math/lp/nla_*.{h,cpp}`
- theory_lra: `src/smt/theory_lra.cpp`
- Trace tags: `src/util/trace_tags.def`

## Active investigations

Listed in `loose-ends/parked.md`. Quick view:

- Wide eval of patched-pseudo-lin + G50 vs master-1472 (~half-day)
- Probe AProVE family with sandwich+mccormick (~30 min)
- Stroeder family is uniformly bad with all PR features — investigate

## Last weekly digest

`weekly/2026-W17.md` — covers Mon-Fri investigation of the
post-rebase eval and the Gröbner-quota finding.
```

## What's in durable/architecture-overview.md

Short. Cites the paper. Provides anchor points.

```markdown
# z3 arithmetic solver — high-level architecture

[FACT] z3's arithmetic solver runs a waterfall: linear-real → linear-integer
→ non-linear. The non-linear stage uses a sub-waterfall over patch monomials,
bounds propagation, Gröbner reduction, incremental linearization, and NLSat.
       → z3-arith-paper §1, Fig. 1

[FACT] Each non-linear component can feed bounds, equalities, or lemmas
back to the LP simplex tableau. The components are individually partial;
overall closure depends on cumulative effect.
       → z3-arith-paper §5, §7

## Code anchors

- Waterfall driver: `src/math/lp/nla_core.cpp::check`, around line 1297
- Component order: `monomial_bounds → pseudo_linear → {horner, grobner,
  bounds} → nlsat (check_assignment) → bounded_nlsat → basics → divisions
  → {order, monotone, tangent}`

## See also

- ./grobner-defaults.md
- ./pseudo-linear.md
- ./nlsat-role.md
```

## What's in journal/2026-04-26-experiment-with-Z.md

Detailed. Reproducible. Has commands and trace excerpts.

```markdown
# G50 ramped Gröbner closes 2 extra seeds on Stroeder Choose

[OBS 2026-04-26] On `Stroeder_15__Choose.c__p22179_terminationG_0.smt2`,
ramping `arith.nl.gr_q` from 10 to 50 and `grobner_eqs_growth` from 10 to
50, plus `grobner_exp_delay=false grobner_frequency=1`, closes 6/8 seeds
versus 4/8 with defaults. Closing time on already-closed seeds is
unchanged (3.06s default → 3.05s G50).

## Reproduce

```bash
F=~/ag/z3/bench/inputs/QF_NIA_small/Stroeder_15__Choose.c__p22179_terminationG_0.smt2
GR="smt.arith.nl.gr_q=50 smt.arith.nl.grobner_eqs_growth=50 \
    smt.arith.nl.grobner_exp_delay=false smt.arith.nl.grobner_frequency=1"

lemur sweep "$F" --seeds 0-7 --timeout 30 --jobs 4 --tally \
  --z3 ~/ag/z3/wt-mono-sandwich/build/z3 \
  --config "default:" --config "G50: $GR"
```

## Lemma-mix change

[OBS] Closing seed 0, default vs G50 (lemur nla output):

| strategy   | default | G50    | Δ     |
|------------|--------:|-------:|------:|
| TOTAL      |    2408 |   1887 | -22%  |
| nlsat      |     849 |    551 | -35%  |
| hi<val     |     886 |    424 | -52%  |
| ord-binom  |      32 |    254 |  +8×  |

[HYP] G50 makes Gröbner produce more linear rows in LP, which reduces
hi<val (LP can refute directly without bound-correction lemmas) and nlsat
(LP-level closure replaces algebraic refutation). The increase in
ord-binom is a side-effect of richer LP state, NOT cascade indication —
total work drops despite ord-binom rising.

## Open

[OPEN] Does this generalize? Loose-ends entry: "Verify G50 holds on a
non-Lyapunov benchmark" — see loose-ends/parked.md.

[OPEN] Does G50 stack with sandwich/tangent-box? Loose-ends:
"Wide eval of G50 + PR features".
```

## What's in weekly/2026-W17.md

Short, interpretive. Already shown in `references/distillation.md`.

## What's in loose-ends/parked.md

Each entry is the five-field format. Example:

```markdown
## Probe AProVE family with sandwich+mccormick
**Status:** parked 2026-04-26
**Context:** Wide eval (run-1485) showed sandwich+mccormick is +3 net only
   on From_AProVE_2014 (vs negative everywhere else). Stroeder Choose
   probe showed McCormick barely fires (3 tan-plane lemmas/run).
   Need to confirm McCormick actually does deterministic work on AProVE
   benchmarks where the wide eval suggests it does.
**Why parked:** G50 finding (4/8 → 6/8 closure with no code) is bigger
   lever; pursuing that first.
**To resume:** Pick juHashMap* instance from the AProVE-gain set. Run
   `lemur sweep --seeds 0-7 --trace nla_solver` for default vs SM. Compare
   tan-plane lemma counts.
**Effort estimate:** ~30 min once benchmark is picked.
**References:** journal/2026-04/2026-04-26-experiment-with-Z.md
```
