# Research Governance: Checkpoints and the Human Interaction Rule

Use this whenever the "Research governance" section of the main skill
signals that a checkpoint condition might be firing, or when you need the
full mechanics behind the five triggers rather than the condensed version
in SKILL.md.

## The role split, restated

```text
The human owns:   intent, constraints, and value judgment.
The AI owns:      exploration execution, experiment design, and
                   evidence synthesis — including the responsibility to
                   interrupt itself.
```

The question to keep running in the background of every phase is not
"does the human understand this experiment" — they don't need to, and
shouldn't have to. It's:

```text
Are we still reducing decision-relevant uncertainty?
```

As long as the answer is clearly yes, stay in normal mode and keep moving
through the loop. The moment it's unclear, that's a checkpoint condition
— not a reason to push through faster to make it clear again.

## The five triggers, in detail

### Trigger 1 — Evidence accumulation

A single experiment answers a single question. A run of three to five
experiments (or evidence records) is a *trajectory*, and a trajectory is
exactly the thing an individual experiment's evidence record isn't
positioned to evaluate — no single Phase 7 write-up asks "is this whole
program still going somewhere useful?" Default to proposing a checkpoint
every 3-5 experiments; treat the number as a heuristic, not a rule — a
two-experiment trajectory that's already badly drifted (Trigger 2)
doesn't need to wait for a third before checking in.

### Trigger 2 — Question drift

Watch for the research question changing *layer* (see the main skill's
four layers) without anyone deciding that on purpose:

```text
Started as:  "Does execution require an explicit identity?"     (Layer A)
Became:      "What should the identity serialization format be?" (Layer D)
```

Neither question is illegitimate — but the second is a different, much
narrower kind of question than the first, and answering it doesn't finish
the first one. If a later experiment's hypotheses are actually targeting
a lower layer than the original question, name that explicitly:

```text
Research layer changed:
    Architecture invariant (Layer A/B/C)
        ↓
    Representation preference (Layer D)
```

and treat it as a signal to pause, not as smooth continuation of the same
work. This also fires automatically on any Phase 0 kernel challenge —
reopening a settled invariant is always a layer-relevant event worth a
checkpoint before the experiment runs, not just after.

### Trigger 3 — Hypothesis expansion

Healthy strong inference collapses the hypothesis set:

```text
H1, H2, H3  →  H1 falsified, H3 falsified  →  H2 survives
```

Watch for the opposite shape:

```text
H1, H2  →  H1, H2, H3, H4, H5
```

An expanding hypothesis set after evidence comes in, rather than a
shrinking one, usually means one of: the boundary of the question is
wrong, the abstraction level is wrong (see the four layers), a Layer
A/B/C decision got skipped and is now leaking into every new hypothesis,
or the experiment's scope grew past what Phase 3 originally specified.
More hypotheses is not more rigor here — it's a symptom.

### Trigger 4 — Evidence saturation

Across several experiments, check whether the *architectural* implication
has stopped changing even though experiments keep running:

```text
Repeated finding:      identity must exist / ownership is explicit /
                        boundary is stable
Remaining differences: UUID vs. hash / JSON vs. protobuf / field name
```

If everything left standing passes the boundary-isolation test (Phase 8)
as ISOLATE, and what's actually still open is Layer D, architecture
research on this question is finished. Continuing to run experiments at
this point isn't reducing decision-relevant uncertainty about the
architecture — it's optimizing an implementation choice, which is a
different (and usually much cheaper) kind of task. Recommend stopping
research and moving to implementation.

### Trigger 5 — Human value check needed

Sometimes the evidence genuinely runs out cleanly: two or more designs
each survive every constructed case, and the boundary-isolation test
can't separate them because nothing left to discover is a matter of
evidence — it's a matter of what the team wants:

```text
Both designs satisfy every correctness constraint tested.
Remaining difference is: simplicity preference, learning value, future
flexibility, team familiarity, or plain enjoyment of one approach over
the other.
```

This is the one trigger where the right move is not "design a sharper
experiment" — there isn't a sharper experiment, because the thing left
undecided was never an empirical question. Stop, and ask. Don't quietly
pick one and move on: that would be the AI making a value judgment that
belongs to the human, dressed up as an architecture decision.

## The human interaction rule

Normal mode and checkpoint mode look different on purpose:

```text
Normal mode:
    AI explores autonomously.
    AI records evidence (Phase 7).
    AI continues to the next phase without asking permission each time.

Checkpoint mode (any trigger above fires):
    AI summarizes the trajectory (see research-retro-template.md for the
    report shape).
    AI explains, in one or two sentences, why intervention is useful
    right now specifically — not a generic "just checking in."
    AI proposes one of: continue / isolate / promote to kernel / transfer
    to human decision (see the retro template's Recommendation section).
    Human chooses. AI does not proceed past this point until they do.
```

The human should never need to reconstruct the research history
themselves to make this call — that reconstruction is exactly what the
retro report is for. If the AI finds itself explaining an experiment's
internals to justify a checkpoint, that's a sign the retro report is
under-summarizing, not a sign the human needs a code walkthrough.

## AI self-check before starting the next experiment

Run this before every experiment past the first one in a trajectory, even
in normal mode — it's cheap and catches most checkpoint conditions before
they need a full retro:

```text
1. What decision could this specific experiment change?
2. What previously unresolved uncertainty does it target?
3. Why is another experiment better right now than implementing,
   isolating (Phase 8), or simply accepting the uncertainty as-is?
4. What would concretely happen if research stopped here?
```

If any of these doesn't have a real answer — not a restated hope that
something useful will turn up — don't start the experiment. Trigger a
checkpoint instead. An AI that can't articulate what decision is at stake
is generating activity, not evidence, no matter how well-instrumented the
next experiment would be.

## The extended loop

```text
Hypothesis
    │
    ▼
Experiment
    │
    ▼
Evidence
    │
    ▼
Checkpoint? (five triggers, self-check)
    │
    ├── Continue ───────────────► next, narrower question
    ├── Isolate ────────────────► Phase 8, fold into kernel
    ├── Promote to kernel ──────► Phase 10, Phase 9 patch
    └── Transfer to human ──────► stop; present retro; wait
```

This doesn't replace the main research loop — it runs alongside it,
evaluated after every pass, not instead of any phase.

## Common failure shapes this catches

- **The AI keeps running experiments because the last one was
  interesting, not because a decision needs it.** Caught by the
  self-check (question 1 has no real answer).
- **The AI quietly answers a Layer D question and reports it as though
  it resolved the original Layer A/B/C question.** Caught by Trigger 2,
  and by re-reading the Original Objective section of the retro against
  the current question.
- **The AI picks between two evidence-tied designs on the AI's own
  aesthetic preference and presents it as a finding.** This is exactly
  what Trigger 5 exists to prevent — if the boundary-isolation test can't
  separate two options, that absence of separation is itself the
  finding, and it belongs to the human to resolve, not the AI to smooth
  over.
