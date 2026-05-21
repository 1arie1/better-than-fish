# Corrections — the supersede pattern

When a `[HYP]` or `[FACT]` is found wrong, the most valuable thing the
system can record is **the correction**, paired with **what made the wrong
claim plausible**. Future agents (and you) face the same cognitive trap.

## The pattern

When the user corrects a claim the agent has made (whether in conversation
or already written to a note):

1. **Find the relevant note.** `grep -l <claim-keyword>` across the notes
   tree. Often the wrong claim only exists in conversation; that's fine,
   skip step 2.
2. **Mark it superseded.** Replace the wrong marker with `[SUPERSEDED →
   path/to/correct-version.md]`. **Do not delete the original text.**
3. **Write the correct claim** as a new note (or update an existing
   correct one).
4. **Append a "why I was misled" paragraph** in the new note. One or two
   sentences explaining what made the wrong claim plausible.
5. **Cross-link** both ways: the new note has a "supersedes:" reference,
   the old note has a "superseded by:" reference.

## Example

In a conversation, the agent claimed:

> `set_true_first_flag` is the general phase-hint API for theory atoms.
> theory_lra calls it on NLA literals to bias SAT toward TRUE.

The user corrected:

> `set_true_first_flag` is part of delayed theory combination. Only applies
> to equalities. Does not mean what you think it does. There are other
> mechanisms.

After the conversation, the agent finds it had written this in
`durable/phase-hint-mechanisms.md`. Fix:

### Old note (after correction)

```markdown
# Phase-hint mechanisms in z3 (SUPERSEDED)

[SUPERSEDED → ./phase-hint-mechanisms.md] Below was an early framing
based on incomplete reading. The correct API for general phase hints on
theory atoms is `force_phase`, not `set_true_first_flag`. See the
corrected note.

[HYP — wrong] `set_true_first_flag` is the general phase-hint API for
theory atoms. theory_lra calls it on NLA literals to bias SAT toward TRUE.
       → src/smt/theory_lra.cpp:2147 (the call exists)
```

### New note

```markdown
# Phase-hint mechanisms in z3

[FACT] `force_phase` is the general phase-hint API. Pokes
`m_phase_available = true; m_phase = phase` in bool_var_data; effective
under `PS_CACHING` (the default).
       → src/smt/smt_context.h:1097

```cpp
void force_phase(bool_var v, bool phase) {
    bool_var_data & d   = get_bdata(v);
    d.m_phase_available = true;
    d.m_phase           = phase;
}
```

[FACT] `set_true_first_flag` is *not* a general phase hint. It's part of
delayed theory combination — `assume_eq` registers interface equalities
between theories with this flag. Its appearance in `assume_literal`
(theory_lra:2147) is repurposed; the right tool for general theory-atom
phase hinting is `force_phase`.
       → src/smt/smt_context.cpp:4581 (assume_eq context)

[FACT] theory_seq uses `force_phase` in four places, the canonical
worked example.
       → src/smt/theory_seq.cpp:1754, 2961, 3121, src/smt/seq_axioms.cpp:77

## Why I was misled

`set_true_first_flag` *is* called from `theory_lra::assume_literal`
(line 2147), and a quick read suggests it's a polarity hint for arbitrary
NLA-emitted theory atoms. But its trail-managed registration and its
primary use site (`context::assume_eq`) reveal it's specifically for
interface-equality DTC, with a flag-based effect that happens to bias
phase. Conflating "is consulted for phase" with "is the phase API" missed
the design intent.

## Supersedes

- `phase-hint-mechanisms.md` (now SUPERSEDED) — the wrong framing.
```

## Why preserve the wrong note

Three reasons:

1. **The cognitive trap is durable.** Future agents will encounter the same
   `set_true_first_flag` call site and have the same first-glance
   interpretation. The "why I was misled" paragraph is what protects them.
2. **Provenance.** When you re-read in 3 months, "supersedes X" is much
   clearer than no trace at all of the wrong claim.
3. **Calibration.** Counting superseded notes over time tells you how
   often early framings are wrong in this domain. That's a useful
   meta-signal.

## When NOT to use the supersede pattern

- **Typos and minor edits.** Fix in place.
- **Notes that were always speculative `[HYP]` and got demoted to noise.**
  Just delete (or move to `loose-ends/archive.md` if there's a useful
  fragment).
- **Reorganization.** If a note moves directories or files, that's a
  rename, not a supersede.

The supersede pattern is for **claims that someone (you, an agent) was
relying on as true**, and turned out not to be. That's where preserving
the trail pays.

## Trigger phrasing

The user's correction may not include "wrong" or "supersede" verbatim. The
agent should recognize corrections from any of:

- "actually X"
- "wait, that's not right — X"
- "X, not Y"
- "the [API/component] doesn't work that way; it's X"
- A factual statement that contradicts a previous agent claim

When in doubt, ask: "Should I supersede the earlier note that said Y, with
the correction X?"
