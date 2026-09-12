# Evidence Record Template

Use this to write up results from Phase 7 of the main skill, after the
experiment has run and its own instrumentation has been adversarially
audited (Phase 5). This is the artifact a stranger should be able to read
and, within a minute, answer: what's settled, what's open, what would
change your mind.

ALWAYS use this exact structure:

```markdown
# [Experiment Name] — Evidence Record

> Status: Experiment complete — [architecture conclusions frozen /
> partially frozen / still unresolved — pick one honestly]
> Follows: [prior experiment or open question this addresses, if any]

## Executive Summary

### Question
[the one-sentence research question]

### Result
[the headline finding, in plain language, before any tables]

### Confidence
[high / medium / low, and why — what property of the test domain limits
confidence, if any]

## Hypothesis Verdicts

| Hypothesis | Verdict | Decisive evidence |
|---|---|---|
| H1 | Confirmed / Weakened / Falsified / Unresolved | [case ID(s)] |
| H2 | | |

## Confirmed Findings

Facts repeatedly supported, with no counterexample found despite looking.
State what looking was actually done — "confirmed" without a description
of the attempt to break it is just an assertion.

## Falsified Assumptions

For each, use the counterexample shape:

```
Initial assumption:      what was believed true, stated precisely
Experiment:               what was actually done
Observed result:          what happened, plainly
Why it (didn't) fail:     the mechanism, not just the outcome
Implication:              what this changes, stated as narrowly as the
                           evidence supports — resist rounding up
```

## Weakened Claims

Where the strong/naive form of a claim died but a narrower form survives.
State the narrower form explicitly — "weakened" is not the same as
"still basically true," and leaving it vague is how a weakened claim
quietly gets treated as confirmed six months later.

## Decision / Closure Matrix

| Question | Observation | Conclusion |
|---|---|---|
| | | |

## What This Does NOT Conclude

Explicit, every time. List the adjacent claims a reader might round this
evidence up to, and say plainly that they are not supported yet:

```
This experiment does NOT establish:
    - [plausible over-reach #1]
    - [plausible over-reach #2]
```

## Remaining Open Questions

Only genuinely unresolved semantic questions — not implementation
TODOs. For each, note what evidence would resolve it, so the next
experiment has a clear target.

## Kernel / Production Impact

```
Can this enter the real system as-is?     Yes / No
Reason:
```

If no: what boundary law (see boundary-isolation-test.md) can be closed
now, and what stays explicitly isolated as an open extension point?

## Confidence Caveats

State plainly if the result depends on a property of the test domain that
a real deployment might not share (e.g., "meanings were branch-independent
in this toy domain; a domain where they aren't could falsify this at
lower confidence"). A too-narrow domain producing a clean result is a
limitation of the experiment, not evidence the architecture is simple.
```
