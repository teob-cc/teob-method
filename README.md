# The Outside-In Method — agent skills

Two skills that carry a system design method your agent can run against **a system
you already have**, not a curated example.

Full method, worked example and eight derivations — the method indexed by what it
does, each proved on the systems where it bites: **<https://teob.cc/method>**

## Install (Claude Code)

```bash
/plugin marketplace add teob-cc/teob-method
/plugin install outside-in@teob-method
```

Then, in any repository:

```
/outside-in:review     audit what exists
/outside-in:derive     derive what doesn't yet
```

## What they do

**`review`** — takes a running system, a design document or a diagram and:

- recovers what is outside the boundary — clients, tolerances, the organisation
  that pays for it
- types every component into surface / channel / processor / store, decomposing
  where one product plays several roles
- runs the **deletion test**: which constraint's removal would delete this?
- then does the division — is the cited requirement's magnitude anywhere near the
  mechanism's? A component can be properly cited and still be four orders of
  magnitude larger than anything asked for
- builds the traceability map and reads it both ways — uncited mechanisms,
  unmapped requirements, and mechanisms carrying more than one promise
- enumerates the promises whose acceptable failure count is **zero**, and names
  the mechanism carrying each, its failure mode and the counter that proves it
- labels every figure **measured**, **contractual** or **assumed** — plus
  **unconsumed** for a number collected and never used — and turns the assumed
  ones into a risk register
- re-derives the arithmetic independently and reports where it disagrees, and
  where it agrees on the wrong quantity

**`derive`** — runs the ten steps forward from a described problem, refusing to
name any technology until the demand arithmetic binds. Produces the artifacts in
order: client classes and tolerances, data classes, the labelled graph, the
degradation ladder, the operations plan, the trace.

## Three rules they will not bend

**No invented numbers.** A missing figure is reported as a finding, never filled
in from a reference architecture. If arithmetic needs an input that does not
exist, the skill names it, marks it assumed, shows the sensitivity and continues.

**The verdict depends on what the artifact is.** In a design not yet built the
deletion test is a razor. In a **running system** it is Chesterton's fence: an
uncited mechanism is reported as *"no recorded reason — confirm before removing"*,
never as an instruction to delete, and the recommendation is an incremental
cutover. In a **worked example** whose purpose is to demonstrate the mechanism the
test inverts — the finding becomes *"the stated requirement cannot discriminate
this mechanism"*, and it lands against the requirement rather than the mechanism.

**No ratio without the division.** "Oversized" is taste until somebody divides.
The skill reports what the requirement demands, what the mechanism supplies, and
the number between them — because taste arguments get settled by seniority and
arithmetic does not.

## Other agents

The skills are plain markdown with YAML frontmatter — the same files work in
Cursor, OpenCode, Windsurf and Copilot. Copy `skills/<name>/SKILL.md` (and its
`references/`) into wherever that agent reads skills from.

## Licence

MIT.
