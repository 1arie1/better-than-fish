# Example: a correction walkthrough

A real correction from a z3 NLA investigation, walked through end-to-end.
Use this as a template for your own corrections.

## Conversation moment

The agent had been reasoning about how to bias SAT decisions for new
theory atoms emitted by pseudo-lin lemmas. It proposed:

> Drafting the minimal patch — three small edits.
> [reads `theory_lra.cpp::assume_literal`, sees `ctx().set_true_first_flag(lit.var())`]
> ...So `set_true_first_flag` is the general-purpose mechanism: pokes the
> bool_var_data directly... For pseudo-lin specifically: the equality
> disjunct evaluates TRUE algebraically by construction. So the natural
> hints would be `set_true_first_flag` on the equality literal...

The user replied:

> going too fast. set_true_first_flag is part of delayed theory combination.
> only applies to equalities. does not mean what you think it does. there
> are other mechanisms to communicate preferred phase.

## Steps the agent took

### 1. Re-read the source

```bash
grep -n "set_true_first_flag\|assume_eq" src/smt/smt_context.{cpp,h}
```

Found `set_true_first_flag` defined at smt_context.cpp:4575 and called
mainly from `context::assume_eq` (line 4605, 4621). Confirmed: it's used
for interface equalities, with the flag side-effecting phase guess at
smt_context.cpp:1809. The call from theory_lra at line 2147 is a
*reuse* of the flag's phase-side-effect, not its design API.

### 2. Found the right mechanism

```bash
grep -nE "force_phase|set_phase|phase_hint" src/smt/smt_context.{cpp,h}
```

`force_phase` at smt_context.h:1097 is the general API. It directly sets
`m_phase_available = true; m_phase = phase`. Comment notes it has no
effect if phase caching is disabled, but caching is on by default.

### 3. Found the canonical user

theory_seq uses `force_phase` in four places, all in the
`mk_literal/force_phase/mark_as_relevant` triplet pattern:

```cpp
literal lit = mk_eq(...);
ctx.force_phase(phase ? lit : ~lit);
ctx.mark_as_relevant(lit);
```

→ src/smt/theory_seq.cpp:1754, 2961, 3121, src/smt/seq_axioms.cpp:77

### 4. Wrote the corrected note

[New file] `durable/phase-hint-mechanisms.md` with `[FACT]` markers,
citing both the API definition and the canonical user.

### 5. Recorded "why I was misled"

The "Why I was misled" paragraph in the corrected note:

> `set_true_first_flag` *is* called from `theory_lra::assume_literal`
> (line 2147), and a quick read suggests it's a polarity hint for
> arbitrary NLA-emitted theory atoms. But its trail-managed registration
> and primary use site (`context::assume_eq`) reveal it's specifically
> for interface-equality DTC, with a flag-based effect that happens to
> bias phase. Conflating "is consulted for phase" with "is the phase API"
> missed the design intent.

### 6. Linked back from the (in-conversation) wrong claim

Since the wrong claim was only in conversation, no on-disk note needed
the `[SUPERSEDED]` marker. If it had been written down, the old file
would have been edited to start with:

```markdown
[SUPERSEDED → ./phase-hint-mechanisms.md]

[Original text preserved below for the lesson it teaches.]
```

## What this example illustrates

- **The user's correction is the highest-value input.** The conversational
  exchange "going too fast" → re-grounded the agent → produced a correct
  durable note.
- **Corrections require code-grounding, not just rewording.** The agent
  ran two `grep`s and read 3-4 line snippets. Without that, the new note
  is just a different opinion.
- **The "why misled" paragraph is the durable artifact.** Anyone reading
  this code in the future will face the same first-glance impression. The
  paragraph protects them.

## Anti-pattern: silent fix

A worse version of this would be: agent re-reads, realizes mistake, just
edits the wrong claim out of the conversation context, never writes the
note. The user's correction is then forgotten by the next session and the
agent walks into the same trap again.

The supersede pattern is what makes corrections **durable**, not just
**immediate**.
