# Integration Patch — wiring `proof-experiment-closure` into `arch-research-skill`

Three small, additive touch points. None change an existing phase's
requirements — they add a signpost at the two places the residual
matters, plus one governance cross-reference. Apply directly to your
copies of these files.

---

## 1. `SKILL.md` — insert between Phase 2 and Phase 3

Insert after the closing line of "## Phase 2 — Generate genuinely
competing hypotheses" and before "## Phase 3 — Design the discriminating
experiment":

```markdown
## Phase 2.5 — Check whether formal reasoning can close part of this (optional)

Before designing the experiment, check whether any of Phase 2's
hypotheses are actually claims that follow from something already
committed to — an existing contract, a kernel invariant, a stated
definition — rather than facts only observable by running code. If so,
load the `proof-experiment-closure` sibling skill and hand it the
Phase 2 hypothesis set. It returns a residual: the narrower set of
claims that remain genuinely empirical, which becomes Phase 3's actual
scope.

This step is a gate, not a requirement. Most questions this skill is
used for — ownership, representation, anything involving real runtime
behavior or human/organizational judgment — will fail the gate
immediately and fall straight through to Phase 3 unchanged, exactly as
before. Don't force a formal model onto a question that resists one;
see the sibling skill's anti-patterns section.
```

---

## 2. `kernel-management.md` — Confirmed Invariants table

Replace:

```
| Invariant | Layer | Evidence record | Introduced by |
```

with:

```
| Invariant | Layer | Basis | Model adequacy open? | Evidence record | Introduced by |
```

`Basis` is `Proof` or `Experiment` — see `proof-experiment-closure`'s
kernel section for why these aren't interchangeable. When back-filling
existing rows, default to `Basis: Experiment`, `Model adequacy open?:
N/A`.

---

## 3. `research-governance.md` — Trigger 5, one added sentence

At the end of the existing Trigger 5 section, add:

```markdown
This also fires when `proof-experiment-closure`'s gate determines a
supposedly-empirical residual is actually Layer B/D or preference —
route it here instead of into a Phase 3 experiment.
```

---

## Why nothing else changes

`evidence-record-template.md`, `boundary-isolation-test.md`,
`mvp-experiment-template.md`, and `worked-examples.md` don't need
edits. A proof gets its own record via
`proof-experiment-closure/references/proof-record-template.md`; once
Phase 3 runs on the residual, everything downstream — evidence
classification, boundary-isolation, MVP scoping, the retro report — is
unchanged. That's deliberate: the smallest correct patch touches the
fewest existing files, which is the same discipline the main skill
already asks of every MVP's scope.
