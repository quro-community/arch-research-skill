---
name: architecture-research
description: Guides falsification-driven architecture research for hard design questions where the right answer is genuinely unknown and multiple designs compete — schema/data ownership, API contract shape, protocol semantics, system boundaries, consistency models, ML pipeline contracts, or any "should X own Y, and how should it be represented" question. Use whenever the user investigates an architectural unknown, wants a small experiment/spike to settle a design disagreement, is choosing between competing models (field vs. argument, this layer vs. that, this owner vs. that), wants to interpret partial experiment results into what's actually settled, or is about to freeze a contract and should check the evidence supports that yet. Trigger even without the words "architecture" or "research" — "not sure if this should live on X or Y," "which of these designs is right," "let's spike this," "prove this wrong before we build it," "did this experiment actually show what we think it did" are strong signals. Also use it to decide whether an experiment loop already in progress should pause for a retrospective checkpoint rather than run unchecked, to scope a new MVP against the accumulated kernel of already-settled findings instead of rebuilding from zero, and to recognize when a remaining question is a human preference call rather than something another experiment could resolve.
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

## Who drives this

This skill assumes an AI agent — not a human — runs Phases 1 through 10:
designing hypotheses, writing and executing experiment code, auditing its
own instrumentation, and drafting evidence records. That changes what the
human is for, and the split is worth keeping explicit through the whole
process rather than assumed once and forgotten:

```text
The human owns:   intent, constraints, and value judgment — which
                   question actually matters, what trade-offs are
                   acceptable, and which of two equally-valid designs to
                   prefer once evidence alone can't decide between them.

The AI owns:      exploration execution, experiment design,
                   instrumentation, and evidence synthesis — and, just as
                   importantly, deciding when to stop and hand a question
                   back rather than keep running experiments.
```

A human collaborator shouldn't need to read every experiment's code to
trust the process. That only holds up if the AI actively watches its own
trajectory and interrupts itself when it should — see "Research
governance" later in this skill for when and how. An AI that keeps
producing experiments just because it can is optimizing for activity, not
for what the human actually needs: a shrinking set of open questions, and
a clear signal for when only a preference remains.

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
Consult the kernel — already settled? already contradicted?
        │
        ▼
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
Fold into the kernel as a versioned patch — never an in-place rewrite
        │
        ▼
Checkpoint gate — continue, isolate, promote, or transfer to human?
        │
        ▼
Next, narrower question (or stop)
```

The output of one pass through this loop is never "the final design." It
is: possibilities eliminated, invariants confirmed, and a sharper
remaining question. Treat that as success. A research program that ends
with "and now we know the whole architecture" after one pass either asked
a trivial question or is lying to itself.

---

## Phase 0 — Load the kernel before opening a new question

Before generating hypotheses for a question that touches a system this
process has studied before, load the current **kernel**: the versioned
record of what earlier passes through this loop actually settled (see
`references/kernel-management.md`). Check two things before doing
anything else:

```text
1. Does an existing kernel invariant already answer this question?
   → If yes, this isn't research — cite the kernel and stop. Re-running
     an experiment to confirm something already Confirmed is
     confirmation-seeking against your own prior work.

2. Does this question contradict a kernel invariant that's already
   frozen?
   → If yes, this is a kernel challenge, not an ordinary experiment. It
     needs the stronger bar in `references/kernel-management.md`
     ("Systematic re-evaluation") and should usually surface as a
     Trigger 2 checkpoint (question drift — see "Research governance")
     before any code gets written, because overturning something settled
     can invalidate whatever was built on top of it.
```

If neither applies, this is a genuinely new question and Phase 1
proceeds normally — but scope it as a **delta against the kernel**, not
as a build from zero. An MVP that quietly re-derives things the kernel
already settled has lost scope control before its first line of code;
see the scope-control note in Phase 3.

**Load the purpose too, and make it load-bearing.** The kernel says what
is settled; it does not say what the work is *for*. Before generating
hypotheses, locate the stated purpose — the consumer, the deliverable,
the thing that gets better if this succeeds. If none is written down,
say so and ask for one; a programme can be exactly right about
everything except its purpose, and no amount of falsification detects
that (counterexample CE-6 in `references/counterexamples.md`).

Three checks, cheapest first:

```text
1. Is a purpose stated anywhere, and does it name a consumer?
   → If not: stop and ask. Do not generate hypotheses against an
     unstated purpose; every later stage inherits the ambiguity.

2. Is this question traceable to that purpose?
   → If not, this may be a legitimate question that belongs to a
     different programme. Say which one, or say that it is curiosity.

3. Does answering it change what the consumer can do?
   → If not, it is a confirmation run. See the prior-prediction check
     in Phase 3.
```

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
  quietly turns into a half-finished feature. Also exclude anything the
  kernel (Phase 0) already settled — re-implementing a Confirmed
  invariant inside a new MVP instead of just depending on it is the most
  common way an MVP's scope quietly expands past what the current
  question needs. See the scope-control note in
  `references/kernel-management.md`.
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
- **The prior-prediction check.** Before building: does a rule you have
  already stated — a contract, an invariant, a membership test, a prior
  finding — already predict this experiment's outcome? If yes, say so in
  the design, and justify the cost in terms the prediction does not
  cover. A confirmation purchased at discovery cost is the most
  expensive kind of true result, and it is invisible in the final report
  because everything passed.

```text
An experiment whose headline outcome was already predicted by a stated
rule has two legitimate shapes:

  (a) re-scope to the uncertainty the rule does NOT settle, and let the
      predicted outcome be a precondition rather than a finding;
  (b) run it as an explicit confirmation with a stated reason the
      prediction might be wrong — a boundary the rule does not reach, a
      case the rule's derivation silently assumed away.

Without (a) or (b), the mechanism has been scoped instead of the
uncertainty. See counterexample CE-4 in `references/counterexamples.md`.
```

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
question can be safely deferred. Use this four-question test on each
remaining item:

```text
1. Can a boundary law be stated now — a constraint that must hold
   regardless of how the open content eventually gets filled in?
2. Is it safe to leave the CONTENT open — can no existing proof or
   law be silently violated while it stays unresolved?
3. Will filling in the content later be a superset refinement
   (strictly additive), not a breaking change to what's fixed now?
4. Is filling it in SOMEONE'S JOB, with a trigger — or is it orphaned?
```

**Yes to all four → isolate.** Fix the boundary law and a minimal typed
core; leave the rest as an explicit, named extension point.
**No to 1–3 → cannot isolate.** It has to be resolved, at least
minimally, before you can honestly call the surrounding design settled.
**Yes to 1–3 and no to 4 → isolate, but ORPHANED.** Safe to leave open,
and due at the next build, with nobody scheduled to fill it.

Question 4 exists because questions 1–3 are answered against the
*proofs* — "can no existing proof be silently violated?" — and that is
the right test for freezing and the wrong test for building. An item can
be perfectly safe against every proof and still be a forced decision the
moment anyone lands the feature. **Landing is not a research event, so
no retro will ever schedule it.** Orphaned items accumulate silently and
then come due all at once, with no research behind any of them — see
counterexample CE-3 in `references/counterexamples.md`.

An orphaned item does not go on a "safely deferred" list. It goes into a
**decision queue** with a named trigger, because what it needs is not
more evidence — it is a choice someone has to make.

This is the single most reusable tool in this whole method: it's what
lets a team keep shipping the parts that are actually settled without
either (a) freezing something the evidence doesn't support yet, or (b)
treating everything as blocked because one piece is still open. Run it
explicitly, in a table, for every open item at the end of a research pass
— see `references/boundary-isolation-test.md`.

## Phase 9 — Fold results into the kernel as a versioned patch, never a silent rewrite

When a research pass changes something the kernel already records, don't
edit the old kernel entry in place. Write a patch that states, as a
table:

```text
| Location | Before | After | Breaking? |
```

for every change, plus an explicit scope statement of what this patch
does *not* close. This preserves the ability to ask "does this new
evidence falsify a previously frozen conclusion, or does it just advance
an item that was already marked open?" — which is the right question to
ask before reopening something settled, and a much cheaper one to answer
against a paper trail of patches than against a history of in-place
edits. See `references/kernel-management.md` for the kernel's structure
and where a patch's entries land inside it.

**A patch that is written but not applied is not a patch.** The closure
document — written by the window that did the work — will be current. The
*shared* documents a reader actually consults (the design doc, the law
register, the README, the specification) are owned by nobody, and they
rot silently. A register that under-reports the frozen set is worse than
no register, because it is consulted and believed.

So every patch names, explicitly, the canonical documents it makes
stale, and updates them in the same pass:

```text
- which shared documents does this finding contradict or supersede?
- which printed/derived artifact (a law list, a status table, a count)
  does it change, and is that artifact regenerated?
- which document referenced a name, file or signature this patch moved?
```

Five independent staleness defects in one programme were found only by
reconciling three corpora by hand, and none of them was visible to a
per-window audit — see counterexample CE-2 in
`references/counterexamples.md`.

## Phase 10 — Gate promotion into the kernel

An experimental finding earns its way into the kernel — and from there,
into production contracts — only after:

```text
repeated evidence
    +
a stated invariant that held across every attempt to break it
    +
no known counterexample
    +
the boundary-isolation test applied to what remains open
```

Passing this gate promotes a finding into the kernel, not directly into
shipped code. The real system's code catching up to what the kernel now
records is a separate, ordinary implementation task, not part of this
research loop.

**But the handoff is part of this loop, and it has two required outputs.**
"It is an ordinary implementation task" is true and is not a reason to
hand over nothing. When a range closes, the programme owes whoever builds
next:

```text
(1) The founding question, answered, in one place.

    A programme is opened by a question. Four windows later that question
    is usually answered, and the answer is distributed across four
    evidence records because each window reports its own findings. Stating
    it in one paragraph — or stating explicitly that it is NOT yet
    answered — is the artifact a consumer actually needs, and it belongs
    to no window. See counterexample CE-5.

(2) The decision queue (§ Phase 8, question 4).

    What landing will force, itemised, with the evidence that can inform
    each choice and an explicit statement of which items have none. An
    isolated item arriving at implementation as a surprise is a planning
    failure of this loop, not of the implementer.
```

A finding recorded only in its own evidence record has not been folded
into the kernel — it has been *filed next to it*.

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

## Research governance — checkpoints, retros, and knowing when to stop

Running Phases 0–10 well on a single question is necessary but not
sufficient. Across a whole research program, the AI is also responsible
for noticing when the *trajectory* itself needs a human check-in, rather
than mechanically starting the next experiment just because the last one
finished. Six conditions warrant proposing a retro checkpoint — see
`references/research-governance.md` for the full mechanics and
`references/research-retro-template.md` for the report to bring to it:

```text
1. Evidence accumulation  — roughly every 3-5 experiments or evidence
                             records, a trajectory (not just a single
                             question) exists and deserves review.

2. Question drift          — the question has moved from one layer to
                             another (e.g. Layer A necessity → Layer D
                             representation) without anyone deciding
                             that on purpose. Also fires on any kernel
                             challenge from Phase 0.

3. Hypothesis expansion     — the hypothesis set is growing (H1, H2 →
                             H1, H2, H3, H4...) instead of collapsing.
                             That's the opposite of a healthy
                             strong-inference pass, and usually means the
                             abstraction level or the scope is wrong, not
                             that more hypotheses are needed.

4. Evidence saturation      — every remaining open item passes the
                             boundary-isolation test (Phase 8) as
                             ISOLATE, and what's left is genuinely
                             Layer D (representation) rather than
                             Layer A/B/C. Architecture research on this
                             question is done; what remains is an
                             implementation choice.

5. Human value check needed — two or more candidates are each
                             individually not falsified, and the
                             boundary-isolation test can't resolve
                             between them because nothing left is a
                             matter of evidence. That's a preference
                             question, not a research question, and no
                             further experiment will change that.

6. Baseline exhaustion     — the roadmap, phase plan or design document
                             that GENERATED this trajectory has no
                             unstarted items left. Everything since has
                             been generated from the previous pass's
                             residuals. This is the most dangerous
                             condition, because residuals are an
                             inexhaustible source of valid, narrow,
                             plausible questions, and a retro that
                             recalibrates against an exhausted baseline
                             will keep producing them indefinitely.
```

**Condition 6 is the one that hides.** The other five are visible from
inside a trajectory — the evidence pile grows, the question drifts, the
hypotheses multiply. Exhaustion is invisible from inside, because the
work still looks like work and every individual question is still sound.
It is detected only by going back to the document that planned the
trajectory and counting what is left undone.

When condition 6 fires, the next question is a **roadmap decision, not a
research decision**. A residual list cannot supply the input it needs —
recalibrate against the wider corpus instead: the phase plan, the design
documents, the stated purpose (Phase 0), the consumer. See
counterexample CE-1 in `references/counterexamples.md`.

The self-check before starting any experiment beyond the first is short
enough to run every time without it feeling like ceremony:

```text
1. What decision could this specific experiment change?
2. What previously unresolved uncertainty does it target?
3. Why is another experiment better right now than implementing,
   isolating (Phase 8), or accepting the uncertainty as-is?
4. What would concretely happen if research stopped here?
5. Does a rule I have already stated predict this experiment's outcome?
   (If yes → Phase 3's prior-prediction check: re-scope to what the rule
   does not settle, or state why the prediction might be wrong.)
6. Can I trace this experiment to the stated purpose — the consumer, the
   deliverable — in one sentence? (If not → Phase 0's purpose check.)
```

If those four don't have real answers, the right move isn't to run the
experiment anyway — it's to stop and bring a retro to the human, using
the format in `references/research-retro-template.md`. A research
partner that maximizes the number of experiments run is optimizing the
wrong thing; the actual goal is decision-quality improvement per unit of
exploration cost, and recognizing that a question is already answered
(or has become a preference question) is as much a skill here as
designing a clever experiment.

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

**Unbounded MVP scope creep.** An MVP that starts re-deriving invariants
the kernel already settled, or that quietly absorbs "final API design"
and "production integration" back into its included scope, isn't a
tightly falsifiable experiment anymore — it's a half-built feature
wearing an experiment's name. Scope every MVP as the delta beyond the
kernel (Phase 0, Phase 3), not as a build from zero.

**Kernel ossification.** The kernel earns trust by being hard to enter
(Phase 10's gate) — it shouldn't also become impossible to leave.
Treating every kernel entry as permanently beyond question is the mirror
image of premature contract freezing above: it swaps "we decided too
early" for "we refuse to ever reconsider," and both end the same way, by
letting the evidence stop mattering. Kernel entries are strong priors
earned by repeated falsification attempts, not axioms — see the
systematic re-evaluation procedure in `references/kernel-management.md`.

**Experimenting past the point of decision-relevance.** Running another
experiment because the loop is comfortable, rather than because a
specific open decision needs it, is confirmation-seeking at the level of
a whole research program instead of a single test. If the self-check in
"Research governance" can't name the decision at stake, that's the
signal to checkpoint, not to keep going.

**Mechanism-scoped milestones.** Scoping a pass as a *thing to
investigate* ("does folding need a primitive?") instead of an
*uncertainty to remove* ("is there any operation a declared record cannot
express?"). A mechanism-scoped pass has a natural end — the mechanism is
covered — and it will reach that end whether or not the uncertainty was
the load-bearing one, at full discovery cost. Worse, when a rule you have
already stated predicts the outcome, the pass becomes a confirmation
purchased at discovery cost, and the report shows nothing wrong because
everything passed. Phase 3's prior-prediction check is the guard.

**The unapplied patch.** Recording a finding in its own evidence record
and treating that as folding it into the kernel. It has been filed *next
to* the kernel. The shared documents a reader actually consults — the
design doc, the register, the status table, the README — are owned by
nobody and rot silently, and a register that under-reports is consulted
and believed. Name the documents your patch makes stale, in the patch
(Phase 9).

**Orphaned isolation.** Leaving an item open because no *proof* depends
on it, and never asking who fills it in. Questions 1–3 of the
boundary-isolation test are answered against the proofs; they say nothing
about landing. Orphaned items come due all at once, at build time, with
no research behind them — and because landing is not a research event, no
retro will schedule them. Phase 8's question 4 is the guard: an isolated
item needs a named owner and a trigger, not just a clean verdict.

**Rigour without direction.** A programme can be exactly right about
everything except what it is for. Falsification tests hypotheses; it does
not test whether the hypotheses were the ones worth having. If no purpose
is written down, or the record never references it, that is not a gap in
the research — it is a gap in the programme, and it is invisible to every
check in this skill. Phase 0's purpose load is the guard, and it is the
cheapest thing in the whole method.

**Volume read as value.** When the output format is per-experiment
evidence, a long programme's accumulated record starts to look like the
deliverable. It is not: it is a cost paid by every future reader. If the
settled content can be stated compactly, stating it compactly *is* the
deliverable — and the founding question answered in one paragraph is
worth more to whoever builds next than four closures that each answer a
quarter of it.

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

Two checks on that measure, because it is self-flattering:

```text
1. Can the settled content be stated compactly?

   If four passes produced one page of load-bearing content and ninety
   pages of evidence for it, the one page is the deliverable and the
   ninety are its support. Say which is which, or a reader will treat the
   volume as the finding.

2. Is the founding question answered, in one place?

   It is usually answered. It is almost never collected.

A stranger needs "what is settled" to include the answer to the question
the programme was opened to ask — not a map of where the quarters of the
answer are filed.
```

And one check on the process, because rigour is not direction:

```text
A method that cannot tell "we established something true" from "we
established something worth establishing" will produce both at the same
cost, and report them identically. That is the failure this skill is
least able to catch from inside, and the reason Phases 0, 8 (question 4)
and 9 (naming stale documents) exist.
```

That includes knowing when to stop. A research pass that keeps running
experiments after the decision-relevant uncertainty is gone isn't rigor —
it's the same failure this method exists to prevent, aimed at itself.
