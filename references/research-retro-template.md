# Research Retro Report Template

Use this whenever "Research governance" (main skill) or
`references/research-governance.md` calls for a checkpoint. The AI
prepares this without asking the human to reconstruct anything — pull it
together from the trajectory's experiment plans, evidence records, and
kernel patches. This is the artifact that lets the human make a
continue/stop/redirect call in a couple of minutes instead of re-reading
every experiment.

ALWAYS use this exact structure:

```markdown
# [Research Program] — Retro Report

> Trajectory: [N] experiments / evidence records since the last checkpoint
> Trigger: [which of the five triggers fired, or "human requested"]

## 1. Original Objective

[What were we trying to discover, stated as it was originally posed —
not reworded to match where we ended up. If the current question has
drifted from this, that's Section 4's job to surface, not this section's
job to paper over.]

## 2. Research Timeline

| Iteration | Question | Experiment | Result |
|---|---|---|---|
| R1 | | | |
| R2 | | | |

## 3. Uncertainty Reduction

```
Before this trajectory, unknown:
    -

After this trajectory:
    Confirmed:   -
    Falsified:   -
    Weakened:    -
    Still open:  -
```

## 4. Research Drift Check

```
Are we still answering the original question?  YES / NO
```

If NO, state plainly:

```
Original question:
Current question:
Why the drift happened:
Was the drift a deliberate, good decision, or did it just happen?
```

A drift that was deliberate and well-reasoned isn't a problem — an
undiscussed one is exactly what this section exists to catch.

## 5. Architecture Extraction

Sort everything this trajectory produced into three buckets — this is
where the trajectory's findings get routed into (or explicitly kept out
of) the kernel:

```
Stable invariants (→ candidates for kernel promotion, Phase 10):
    -

Temporary findings (useful for the current MVP only, not kernel-worthy):
    -

Deferred / isolated (→ kernel extension points, Phase 8):
    -
```

## 6. Recommendation

Pick exactly one, and state the reason in terms of this specific
trajectory, not generically:

```
CONTINUE RESEARCH
Reason: [what decision-relevant uncertainty remains]
Next experiment: [one sentence]

ISOLATE AND PROCEED
Reason: [what boundary law closes this, per Phase 8]

PROMOTE TO KERNEL
Reason: [what's Confirmed and ready for Phase 10's gate]

TRANSFER TO HUMAN DECISION
Reason: [what technical uncertainty is resolved; what remains is
preference — name the actual trade-off, don't just say "preference"]
```

## What This Trajectory Does NOT Settle

Same discipline as an evidence record's "does not conclude" section
(`references/evidence-record-template.md`), applied at the trajectory
level: name the adjacent claims a reader might round this trajectory up
to, and say plainly they're not supported yet.
```

## Notes on filling this in well

- **Section 1 is a quotation, not a summary.** If you can't state the
  original objective without also explaining how it changed, that's
  itself evidence for Section 4 — don't pre-resolve the drift check by
  wording Section 1 to match the current question.
- **Section 2's timeline should be skimmable in under a minute.** One row
  per experiment, not a paragraph. If a row needs a paragraph to explain,
  point to the evidence record instead of inlining it here.
- **Section 6 must pick exactly one recommendation.** A retro that hedges
  across two recommendations hasn't actually decided anything, which
  defeats the point of pausing for a checkpoint in the first place. If
  the honest answer is genuinely split, that split — stated as a
  concrete disagreement between two named options — belongs in Section
  6's Reason line, not as a rewrite of the recommendation into two soft
  half-recommendations.
- **This is not a substitute for the evidence records themselves.** The
  retro is a navigation aid over evidence that already exists elsewhere;
  it should link to or name the underlying evidence records rather than
  restate their contents in full.
