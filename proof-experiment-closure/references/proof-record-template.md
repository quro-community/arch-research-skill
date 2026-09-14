# Proof Record Template

Use this when the proof-experiment-closure gate produces a derivation
that closes — fully or partially — one or more Phase-2 hypotheses,
before handing the residual to Phase 3. Deliberately reads like
`evidence-record-template.md` in the main skill, so a reader auditing
the kernel later doesn't have to learn a second format for proof-based
findings.

ALWAYS use this exact structure:

```markdown
# [Question] — Proof Record

> Status: [Fully closes the question / Partially closes — residual
> handed to Phase 3 / Attempted, gate failed — no proof, see fallback]
> Hypotheses addressed: [H1, H2, ... from Phase 2]

## Model

```
Objects:      [named, minimal — no object not used in the derivation below]
Operation(s): [the transformation(s) under test]
```

## Assumptions

Every fact taken as given, each cited to its source — a kernel
invariant, an existing contract, a stated definition. An assumption
with no citation is a Layer A/B/C decision smuggled in as a premise;
if it can't be cited, it isn't an assumption, it's the thing under
dispute.

```
A1: [assumption] — source: [kernel v_, contract §_, definition in ___]
A2: ...
```

## Hypotheses as Formal Claims

```
H1: [restated as a claim about the model]
H2: [restated as a claim about the model]
```

## Derivation

The actual reasoning — not a restatement of the conclusion. Name the
step at which H1 and H2 could, in principle, have come out differently;
if there isn't one, this is an assertion, not a derivation (see "Proof
theater" in the main gate).

## Verdict

| Hypothesis | Verdict | Which step decides it |
|---|---|---|
| H1 | Confirmed / Falsified / Not addressed by this proof | |
| H2 | | |

## Residual Handed to Phase 3

Use the residual table from the main gate (`SKILL.md`, "What remains
empirical, always"). This must be non-empty unless Status above is
"Fully closes" — and even then, model adequacy (below) stays open.

```
Residual: [conformance / model-adequacy / runtime-behavior /
    adversarial-evasion — name which, per the gate's residual table]
```

## Model Adequacy — Explicitly Left Open

```
This proof assumed the model matches the real system in respects:
    [X, Y, ...]
That assumption is NOT itself proven here.
It would be checked by:
    [the conformance experiment this proof's residual hands to Phase 3,
    or "not yet scoped" if none exists yet]
```

## What This Proof Does NOT Conclude

Same discipline as the main evidence-record template — name the
adjacent claims a reader might round this proof up to, and say plainly
they are not supported yet:

```
This proof does NOT establish:
    - that any concrete implementation conforms to this model
    - that the model's assumptions hold outside the cases they were
      drawn from
    - [domain-specific over-reach, if any]
```
```
