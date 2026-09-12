# MVP Experiment Template

Use this to design the experiment from Phase 3 of the main skill. Fill in
every section before writing any implementation code — a section you can't
fill in is a sign the experiment isn't well-formed yet, not a section to
skip.

ALWAYS use this exact structure when writing up an experiment plan for the
user to review before running it:

```markdown
# [Question] — Experiment Plan

## 1. Research Question

One sentence, phrased so a specific observation could answer it.

Bad:  "How should identity work here?"
Good: "Does [transformation] preserve [property], or can a case be
       constructed where it silently doesn't?"

## 2. Competing Hypotheses

At least two, mutually exclusive where possible, each with a stated
predicted observation:

H1: [claim] — would be supported by observing [X], falsified by [Y]
H2: [claim] — would be supported by observing [X'], falsified by [Y']
H3: [claim] — (optional, but a third hypothesis is often the one that
    reveals the actual discriminating experiment)

## 3. Experiment Goal

One sentence: "This experiment determines whether ___, by ___."
Not: "This experiment builds ___."

## 4. Scope

### Included
- [the specific transformations / conditions / components under test]

### Excluded
- final API design / production integration / optimization
- [anything else genuinely out of scope — say so explicitly, even if
  obvious, so a reader doesn't assume it was overlooked]

## 5. Minimal Model

Define only the objects the experiment actually needs. Do not add a field
or object "because it might be useful" — that's schema-first design, and
it's an anti-pattern (see main skill). If you find yourself adding a field
with no test that exercises it, stop and ask whether the experiment scope
grew without you deciding that on purpose.

```
Object A: [fields actually used, and why each is needed]
Object B: ...
```

## 6. Test / Case Matrix

| ID | Question this case answers | Setup | Expected (per hypothesis) |
|---|---|---|---|
| R1 | | | H1 predicts ___, H2 predicts ___ |
| R2 | | | |

Include at least one case per hypothesis where the hypotheses' predictions
genuinely diverge — a case where every hypothesis predicts the same
outcome tells you nothing about which is true.

## 7. Negative Controls

At least one case that is deliberately broken, to prove the harness can
detect brokenness:

```
Control: [what is disabled/removed/left unbound]
Expected: this MUST fail, and fail in [specific, named way]
If it doesn't fail: the harness is not measuring what section 2 assumes
it measures — fix the harness before trusting any other result.
```

Also include, where relevant, a control aimed specifically at the
"silent wrong answer" failure shape (see main skill, Phase 6) — a case
designed to succeed-but-be-wrong if your favored hypothesis is false, not
just a case designed to crash if it's false.

## 8. Required Data Per Case

Never record only pass/fail. For every case capture:

```
input state
transformation / operation applied
output state
identity preserved? (state the equivalence basis used — not object
    identity unless that's genuinely what you mean)
dependency preserved / lost?
failure mode (if any) — exact and specific, not "it broke"
comparison basis (what makes two outputs "the same" here, explicitly)
```

## 9. Pre-registered Falsification Criteria

Write this BEFORE running anything:

```
If [specific observation], H1 is falsified.
If [specific observation], H2 is falsified.
If [specific observation], neither is falsified and the question needs
    a sharper follow-up experiment.
```

## 10. Success Criteria

Not "all tests pass." Instead:

```
This experiment succeeds if it distinguishes H1 from H2 — regardless of
which one wins, or if it produces a clean falsification of both,
revealing that the real answer is H3 (or an H4 nobody had proposed yet).
```

An experiment where every plausible outcome would have been reported as
"success" was not actually testing anything.

## 11. Kernel / Production Impact

Answer explicitly:

```
Does this require:
    no change to the existing system
    a new, isolated extension point
    a change to an existing contract
```

State why, in one or two sentences, referencing the specific finding —
not "seems fine" or "seems necessary."
```
