# Format conventions

## Markers

Every claim in a note begins with one of:

```
[FACT]               math/paper/source/design-pattern; lives in durable/
[EMP]                repeated empirical claim; lives in empirical/
                     carries "Verified against" version stamp
[OBS YYYY-MM-DD]     single empirical observation, dated; lives in journal/
[HYP]                hypothesis, untested or partial; lives in journal/
[OPEN]               known unknown; corresponding entry in loose-ends/
[SUPERSEDED → path]  trail to corrected version; never deleted
```

A note can mix markers. The first paragraph's marker sets the note's
"primary status" — that determines which directory it belongs in.

### Tier discipline

- `durable/` notes hold `[FACT]`s. A note with mostly `[OBS]` or `[EMP]`
  belongs elsewhere. Mixed `[FACT]+[OPEN]` is fine.
- `empirical/` notes hold `[EMP]`s. Each one starts with a header block:

  ```
  **Verified against:** z3 master at commit b0956429
  **Last verified:** 2026-04-26
  **Status:** single-benchmark / repeated / wide-eval-confirmed
  ```

  This makes the version-binding visible and prompts the next agent to
  re-verify before relying.

- `journal/` notes are dated and exploratory. `[OBS]`, `[HYP]`, and
  follow-up `[OPEN]`s mix freely.

### Source citations

`[FACT]` claims must cite a source. Acceptable forms:

```
[FACT] Z3's Gröbner adds derived linear equations directly to the
       Simplex tableau via "Linear Propagation" post-processing.
       → z3-arith-paper §5.4
       → src/math/lp/nla_grobner.cpp:701 (m_check_feasible = true after
         add_term)
```

Multiple sources are fine. Code citations should be `path:line` plus a
1-2 line snippet so an agent can verify quickly without opening the file.

### Date stamps

`[OBS]` always carries a date. `[FACT]` doesn't need one — it's timeless
by definition. `[HYP]` is dated implicitly by its directory (journal/YYYY-MM)
but include a date in the marker if the hypothesis was formed at a
specific time of investigation.

### Confidence shifts

When a `[HYP]` becomes `[OBS]` or `[FACT]`, edit the marker in place. Don't
write a new note. Add a "Promoted YYYY-MM-DD: [reason]" line at the top.

When a `[FACT]` is found wrong: promote to `[SUPERSEDED]`. Write a new
`[FACT]` note. Cross-link both ways.

## File naming

Filenames describe **when the note becomes useful**, not what it's about.

```
✗ grobner-tuning.md           (topic-named; agent has to read to know)
✓ nla-z3-times-out-on-       (trigger-named; one line tells you when
  lyapunov-benchmarks.md       to load it)
```

Files in `durable/` use kebab-case, no dates. Files in `journal/` start
with the date:

```
journal/2026-04/2026-04-26-grobner-q50-stroeder.md
```

The date is redundant with the directory but makes filenames sortable in
plain `ls`.

## Note structure

Short header, then content. No heavy frontmatter.

```markdown
# Title

Optional one-line subtitle.

[FACT] First load-bearing claim. Cite source.

[FACT] Second claim if needed.

## Why this matters

How this knowledge changes future work. One paragraph.

## See also

- ./related-note.md
- ../journal/2026-04/specific-finding.md
```

Sections beyond `# Title` and the markers are optional. A single-paragraph
note is fine.

## Length budget

| location | typical length | hard cap |
|---|---|---|
| CLAUDE.md (notes dir root) | 50-80 lines | 100 lines |
| durable/ note | 50-200 lines | 300 |
| journal/ entry | 30-100 lines | 200 |
| weekly/ digest | 50-100 lines | 150 |
| monthly/ digest | 100-200 lines | 300 |
| loose-ends/parked.md entry | 10-15 lines | 20 |

If a note grows past its cap, split it. Two related notes with cross-links
beat one sprawling note.

## Linking

Filename references, not URLs or relative paths with directory traversal:

```
✓ See also: grobner-defaults.md
✓ See also: ../journal/2026-04/2026-04-26-grobner-q50-stroeder.md
✗ See also: [Gröbner notes](../../durable/grobner-defaults.md)
```

Plain filenames are grep-able. The agent finds them by `grep -l <filename>`
across the notes tree.

## Code excerpts in notes

Use fenced code blocks with the language tag. Include 3-10 lines of
context, not the whole function.

```markdown
[FACT] `force_phase` is the general phase-hint API. Sets `m_phase` directly
       in `bool_var_data`, effective under PS_CACHING (the default mode).
       → src/smt/smt_context.h:1097

       ```cpp
       void force_phase(bool_var v, bool phase) {
           bool_var_data & d = get_bdata(v);
           d.m_phase_available = true;
           d.m_phase = phase;
       }
       ```
```

## Conventions for trace and command examples

When showing a command an agent should run to verify a `[FACT]`, format
it so it's copy-pasteable:

```bash
# Verify Gröbner default quotas
$Z3 -p | grep grobner | head -10
```

When showing trace output, mark it as the source for the conclusion:

```
$ lemur nla M_s0.trace -f plain | grep -A 4 "Lemmas generated"
Lemmas generated: 2470
  hi<val: 901 (36.5%)
  nlsat: 882 (35.7%)
  ...
```

Trace excerpts are evidence — they belong in the journal entry that drew
the conclusion, not in `durable/`.
