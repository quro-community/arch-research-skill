---
name: architecture-research
description: Guides falsification-driven architecture research for hard design questions where the right answer is genuinely unknown and multiple designs compete — schema/data ownership, API contract shape, protocol semantics, system boundaries, consistency models, ML pipeline contracts, or any "should X own Y, and how should it be represented" question. Use whenever the user investigates an architectural unknown, wants a small experiment/spike to settle a design disagreement, is choosing between competing models (field vs. argument, this layer vs. that, this owner vs. that), wants to interpret partial experiment results into what's actually settled, or is about to freeze a contract and should check the evidence supports that yet. Trigger even without the words "architecture" or "research" — "not sure if this should live on X or Y," "which of these designs is right," "let's spike this," "prove this wrong before we build it," "did this experiment actually show what we think it did" are strong signals.
---

# Architecture Research Through Falsifiable Inquiry

## What this skill is for

Some architecture questions have a known-good answer waiting to be looked
up. This skill is not for those.

It is for the other kind: questions where several designs are each
individually plausible, where committing early creates a trap that's
expensive to back out of, and where the team's actual problem is not
"which design is correct" but "we don't yet have evidence that would tell
us." That situation shows up constantly outside of software too — a
storage engine's consistency model, a payment API's idempotency contract,
an ML pipeline's reproducibility guarantees, even a non-technical question
like who owns a cross-team decision — and it responds to the same
discipline every time: stop debating the design, and go build the
smallest thing that could prove one of the candidates wrong.

This skill exists to make that discipline concrete and repeatable rather
than a vague gesture at "let's prototype it."

## The core reframe

Don't ask:

> "What is the correct architecture?"

Ask:

> "What is the smallest experiment that could prove a specific candidate
> architecture wrong?"

This is Karl Popper's basic move (a theory earns its keep by being
falsifiable, not by being confirmed) combined with what philosophers of
science call *severity*: an experiment is only evidence for a claim if the
claim had a real chance of failing it. "I ran it and it worked" is weak
evidence if failure was never really on the table — if a bug, a
misconfigured control, or an untested code path meant the experiment
couldn't have shown the alternative even if the alternative were true.

The practical form of this — used across experimental physics, molecular
biology, and (as it happens) the research process this skill was
generalized from — is **strong inference** (J. Platt, 1964): hold at least
two genuinely competing, mutually exclusive hypotheses at once, and design
one experiment whose outcome would discriminate between them. A single
hypothesis invites you to go looking for confirmation, which is cheap and
always available. Two or more hypotheses forces you to design something
that can actually kill one of them.

## The four layers — and why collapsing them is the recurring mistake

Almost every version of this failure mode traces back to skipping straight
from "this needs to exist" to "here's the field for it," without passing
through the two questions in between. Keep these separate on purpose:

```text
Layer A — Necessity
    Must this concept exist for the system to be correct at all?

Layer B — Ownership
    Which component is authoritative for it?
    (Not: which component is convenient to put it in.)

Layer C — Identity
    What makes two instances of this thing "the same" one?
    This is an equivalence relation, and it is very easy to
    smuggle it in silently — see the callout below.

Layer D — Representation
    How is it actually encoded? Field, argument, reference,
    content hash, capability, out-of-band registry entry?
```

A semantic requirement (Layer A) does not imply an ownership decision
(Layer B). An ownership decision does not imply you know what equality
means for the thing (Layer C). And none of the above tells you the
encoding (Layer D). Move through them in order. Don't let a data structure
answer a question that hasn't been asked yet.

**Layer C is the one people skip, and it's the one that bites hardest.**
It is entirely possible to correctly settle *who owns X* and still get
burned, because you never separately asked *what makes two X's equal*. A
name is not automatically a stable identity — two different things can
answer to the same name, and the fix (binding identity to actual content
rather than to a label) is a distinct decision from ownership. If a
"same declared inputs → same result" claim starts looking shaky, the usual
cause is that Layer C was never asked as its own question.

This maps onto distinctions that show up under other names in other
fields: DDD's strategic vs. tactical design roughly mirrors Necessity vs.
Representation; a C4-style architecture diagram's "container" vs.
"component" levels are a Layer B vs. Layer D split. The names don't
matter. The discipline of not collapsing them does.

## The research loop

```text
Competing Hypotheses (≥2, mutually exclusive if possible)
        │
        ▼
Discriminating Experiment (designed to kill one, not confirm all)
        │
        ▼
Pre-registered observation (write down what falsifies what, BEFORE running)
        │
        ▼
Run it — collect state transitions, not just pass/fail
        │
        ▼
Adversarially audit your own instrumentation
        │
        ▼
Classify: Confirmed / Weakened / Falsified / Unresolved
        │
        ▼
Evidence record (claim, grounds, explicit non-conclusions)
        │
        ▼
Apply the boundary-isolation test to what's left open
        │
        ▼
Fold in as a versioned patch — never an in-place rewrite
        │
        ▼
Next, narrower question
```

The output of one pass through this loop is never "the final design." It
is: possibilities eliminated, invariants confirmed, and a sharper
remaining question. Treat that as success. A research program that ends
with "and now we know the whole architecture" after one pass either asked
a trivial question or is lying to itself.

---

## Phase 1 — Check whether this is actually an open question

Before designing anything, ask whether the question as posed already
assumes its answer.

```text
Bad:   "Should the version field live on the record?"
Good:  "Can the system reconstruct correct semantics without a
        version field, or is a counterexample constructible?"
```

The first framing has already picked Layer D (a field) and Layer B (the
record) before asking Layer A. The second is falsifiable: someone could
show you a working system that doesn't need it, which would settle the
question in the other direction.

If the user's request already contains a proposed schema, don't just
implement it — ask which layer it's actually answering, and whether the
layers underneath it have been settled yet.

## Phase 2 — Generate genuinely competing hypotheses

Aim for at least two, and make them mutually exclusive where you can —
not "maybe X" vs. "definitely not sure," but two candidates that predict
*different, observable* outcomes for the same test.

```text
H1: [Component] owns this — evidence would be that it survives every
    operation [Component] is defined to support, without help.

H2: [Different component] owns this — evidence would be that H1's
    component loses or ignores it under some operation H1 predicts
    it should survive.

H3: nobody owns it; it's a caller/environment obligation each time —
    evidence would be that the value works when supplied and breaks,
    predictably, when omitted (not silently defaults to something).
```

If you can only construct one hypothesis, that's a signal you're not
ready to design the experiment yet — go find the second candidate a
domain expert, a prior design doc, or a sibling system would propose.
Steelman it even if you suspect it's wrong; a hypothesis you don't
actually believe is often the one that reveals the crucial test.

## Phase 3 — Design the discriminating experiment

A real experiment design specifies, explicitly:

- **One-sentence objective.** What does this determine? Not what does it
  build.
- **Scope: included / excluded.** State what the experiment will and will
  not touch. Excluding "final API design," "production integration," and
  "optimization" is usually correct — those are Phase-9-and-later
  concerns, and pulling them in early is how a falsification experiment
  quietly turns into a half-finished feature.
- **The transformations or conditions under test**, not just the
  end states. Prefer testing what happens to a thing across the
  operations your system actually performs on it (an update, a merge, a
  restart, a retry, a boundary crossing) over testing it once in
  isolation. Static existence is weak evidence; behavior under
  transformation is strong evidence.
- **Negative controls.** Build at least one variant that *should* fail,
  deliberately — a version with the safeguard removed, the identity
  unbound, the reading missing. If your negative control doesn't fail,
  your harness isn't measuring what you think it's measuring, and every
  positive result upstream of that bug is suspect. (This is the same
  logic as a blank sample in a wet-lab assay, or a placebo arm in a
  trial: if you don't know what "clearly wrong" looks like on your own
  instrument, you can't trust what "right" looks like either.)
- **What counts as data.** Never collect only pass/fail. For each case,
  capture: input state, the transformation applied, output state,
  whether identity was preserved or changed, whether a dependency
  survived or was lost, the exact failure mode if any, and the basis on
  which two states were compared (object identity is almost always the
  wrong basis — compare the thing you actually care about).
- **What result would falsify which hypothesis.** Write this down before
  running anything. If you can't state in advance what a failing result
  would look like, the experiment isn't well-formed yet.

See `references/mvp-experiment-template.md` for a fill-in-the-blanks
version of this, and `references/worked-examples.md` for four full
examples across different domains (a distributed-systems one, a
data-platform one, an ML one, and a non-software org-design one) showing
what a completed version of this phase actually looks like.

## Phase 4 — Pre-register before running

Separate **observation** from **assertion**, and write the assertions
before you look at results:

```text
observe:  code that records what happened, and asserts nothing.
assert:   code that names an invariant and fails loudly if violated,
          written against the hypotheses from Phase 2 — not against
          whatever the observations turned out to show.
```

This is the same reform the experimental-psychology and medical-trials
communities adopted after learning, expensively, that hypotheses quietly
rewritten to match results ("HARKing" — Hypothesizing After Results are
Known) produce a research record that looks rigorous and isn't. A green
suite is not evidence unless the assertions could have gone red, and were
written down before you knew whether they would.

A useful habit: a passing test suite proves the code does what the tests
check. It does not by itself prove the tests check the right thing, or
that passing constitutes progress on the actual question. Keep "does it
pass" and "does passing mean anything" as two separate questions you ask
in sequence, not one you collapse.

## Phase 5 — Run it, then adversarially audit your own instrumentation

After the experiment runs and produces results you like, do one more
pass: try to break your own measurement, not the hypothesis.

This is the same move as **mutation testing** — deliberately introduce a
defect into the thing being measured and confirm your instrumentation
actually catches it. Concretely:

- Deliberately neuter the mechanism under test and confirm the "it
  worked" signal turns to "it failed." If it doesn't, the signal is
  measuring something else (a very common bug: comparing a *claim* the
  code made about what it did, rather than reading back the actual
  resulting state).
- Check whether a comparison is using identity (`is`, object equality,
  pointer equality) where it should be using the semantic property you
  actually care about.
- Check for vocabulary bleed across domains or test fixtures — a shared
  mutable default, a registry keyed by something less specific than you
  assumed, a fixture reused across two cases that were supposed to be
  independent.
- Check that every allow-listed exception in a leakage/isolation check is
  actually exercised by a test, not merely declared and forgotten.

Treat what this pass finds as valuable evidence about the experiment, not
as an embarrassment to bury. The instructive finding is usually the bug
in the harness, not the bug in the hypothesis — record it in the write-up
rather than quietly fixing it and moving on. This mirrors blameless
postmortem culture in safety-critical engineering: the point of surfacing
your own mistake in the open is that it's reusable information for
whoever reads the report next.

## Phase 6 — Classify results against a living, minimal taxonomy

Every observation gets one of exactly four verdicts:

| Verdict | Meaning |
|---|---|
| **Confirmed** | Repeatedly supported, no counterexample found despite looking. |
| **Weakened** | Still plausible, but the naive/strong form is dead — a narrower form survives. |
| **Falsified** | A counterexample was constructed. Say what specifically broke it. |
| **Unresolved** | Not enough evidence either way. Say what evidence would resolve it. |

Resist inventing a fifth verdict for a special case — that's usually
Layer A creep, where a Representation-level detail is being treated as if
it changed the semantic verdict.

Separately, maintain a small, domain-specific **failure-class taxonomy**
(e.g., "identity ambiguity," "dependency loss across a transformation,"
"silent conflict resolution") that starts empty and grows only when a
genuinely new failure shape is observed — not preemptively. Classifying
every new finding against this list, and asking whether it's really new
or a repeat of a known class, keeps the vocabulary from sprawling and
makes cross-experiment comparison possible.

**Give silent-wrong-answer failures more weight than loud-refusal
failures**, deliberately, even though they're the same number of "bugs."
A loud, explicit failure is cheap: it announces itself, it's caught in
testing, and it's boring to fix. A transformation that *succeeds* and
produces a plausible-looking but semantically wrong result is the
dangerous case — nothing points at it, and it will surface later, further
from its cause. This is the same reasoning behind "fail-safe" design in
safety engineering and why silent data corruption is treated as a more
serious defect class than a crash in storage-systems engineering. When
designing Phase 3's negative controls, make sure at least one of them is
specifically hunting for this shape of failure, not just for crashes.

## Phase 7 — Write the evidence record

For each counterexample or confirmed finding, use this shape (a
lightweight version of Toulmin's model of argument — claim, grounds,
warrant):

```text
Initial assumption:        what was believed true
Experiment:                what was actually done
Observed result:           what happened, plainly stated
Why it did/didn't fail:    the mechanism, not just the outcome
Implication:                what this changes about the architecture,
                            stated as narrowly as the evidence supports
```

Two disciplines matter more than the format:

1. **State what this does NOT conclude**, explicitly, every time. It is
   very easy for a reader (including a future you) to round "we found
   evidence for X" up to "X is settled." A short, explicit list of
   adjacent claims the evidence does *not* support is cheap insurance
   against that. Passing an experiment is not the same as promoting a
   conclusion into the real architecture — say so in those terms if it
   helps make the distinction visible.
2. **Confidence is not binary.** If a result depends on a property of
   your toy domain that a real domain might not share (branch-independent
   meanings, no constraints axis, only one implementation tested), say so
   and flag the confidence as low rather than reporting the verdict at
   full strength.

See `references/evidence-record-template.md` for the full template.

## Phase 8 — Apply the boundary-isolation test to what's left open

Not every open question needs to block progress, and not every open
question can be safely deferred. Use this three-question test on each
remaining item:

```text
1. Can a boundary law be stated now — a constraint that must hold
   regardless of how the open content eventually gets filled in?
2. Is it safe to leave the CONTENT open — can no existing proof or
   law be silently violated while it stays unresolved?
3. Will filling in the content later be a superset refinement
   (strictly additive), not a breaking change to what's fixed now?
```

**Yes to all three → isolate.** Fix the boundary law and a minimal typed
core; leave the rest as an explicit, named extension point.
**No to any → cannot isolate.** It has to be resolved, at least
minimally, before you can honestly call the surrounding design settled.

This is the single most reusable tool in this whole method: it's what
lets a team keep shipping the parts that are actually settled without
either (a) freezing something the evidence doesn't support yet, or (b)
treating everything as blocked because one piece is still open. Run it
explicitly, in a table, for every open item at the end of a research pass
— see `references/boundary-isolation-test.md`.

## Phase 9 — Fold results in as a versioned patch, never a silent rewrite

When a research pass changes something that was previously documented or
implemented, don't edit the old artifact in place. Write a patch that
states, as a table:

```text
| Location | Before | After | Breaking? |
```

for every change, plus an explicit scope statement of what this patch
does *not* close. This preserves the ability to ask "does this new
evidence falsify a previously frozen conclusion, or does it just advance
an item that was already marked open?" — which is the right question to
ask before reopening something settled, and a much cheaper one to answer
against a paper trail of patches than against a history of in-place edits.

## Phase 10 — Gate promotion into the real system

An experimental finding earns its way into production contracts only
after:

```text
repeated evidence
    +
a stated invariant that held across every attempt to break it
    +
no known counterexample
    +
the boundary-isolation test applied to what remains open
```

Until then, keep experimental code physically and structurally separate
from the system it's studying — in its own directory, with a one-way
dependency (the experiment may depend on the real system to test it
against; the real system must never depend on the experiment) and an
explicit isolation check that fails loudly if that direction is ever
reversed. This is the same discipline as an anti-corruption layer in
domain-driven design or a feature-flagged spike in agile practice: the
exploratory code is disposable by construction, so a wrong turn costs a
deleted directory, not a migration.

---

## Anti-patterns

**Architecture by intuition.** "This feels right" is not evidence. If you
can't say what observation would have changed your mind, you weren't
doing research.

**Schema-first design.** Adding fields until a test goes green has no
natural stopping point, and richness of representation is not evidence of
semantic completeness — a structure can grow indefinitely without ever
proving it captures the right thing. Derive the minimal closure from the
falsifiable question; don't grow a struct toward it.

**Implementation-proves-semantics.** "The code works, therefore the
architecture is correct" mistakes a necessary condition for a sufficient
one. Passing tests is a target that's easy to hit without hitting the
thing the tests were meant to stand in for — a textbook case of Goodhart's
Law (a measure, once made the target, stops measuring what you wanted).
Keep the actual falsifiable claim in view, separately from whether the
suite is green.

**Silent success.** Treat "it ran without error" and "it produced the
right semantics" as two different claims that both need checking — see
Phase 6. A mountable-but-wrong result is worse than a refusal, because
nothing points at it.

**Confirmation-seeking.** Running the same successful case five times, or
five slightly different successful cases, is weak evidence — it's an
absence of falsification attempts, not a presence of severe testing. Go
looking for the case that would break your favored hypothesis before
declaring it confirmed.

**HARKing.** Adjusting the stated hypothesis after seeing results so that
the result looks like confirmation. If the write-up's "hypothesis"
section was edited after the "results" section, the write-up is not
evidence of anything.

**Hiding unknowns inside a generic bag object.** A `Context { everything
unknown }` or `metadata: dict` field doesn't resolve Layer A/B/C
questions — it just defers them somewhere less visible, usually past the
point where anyone remembers to come back and ask them.

**Premature contract freezing.** "We need the final API before we can
run experiments" gets the order backwards — it's exactly the "big design
up front" failure mode agile practice pushed back on, for the same
reason: a contract fixed before the evidence exists tends to fossilize
the first plausible-sounding answer rather than the best-supported one.
Decide representation (Layer D) at the last responsible moment, after
Layers A–C are actually settled.

---

## Final principle

A good architecture research process does not prevent wrong ideas. It
makes wrong ideas cheap to discover, and it makes the discovery legible
to someone who wasn't in the room.

The measure of progress on a pass through this loop is not "how much
architecture got built." It's "how much genuine uncertainty got removed,
and can the next person tell exactly which uncertainty is gone and which
remains." If a research write-up doesn't let a stranger answer "what's
now settled, what's still open, and what would change my mind" in under a
minute, it isn't finished yet — regardless of how much code it contains.