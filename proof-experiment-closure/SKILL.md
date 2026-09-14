---
name: proof-experiment-closure
description: Use between architecture-research Phase 2 and 3 to test whether competing hypotheses can be resolved by formal reasoning before building an experiment. Trigger only for genuinely formal claims involving invariants, transformations, types, state machines, or equivalence relations. Do not use for ownership, representation, preference, organizational behavior, or inherently empirical runtime questions. Time-box formalization; if no faithful model or derivation emerges, fall through to Phase 3. Any proof must leave model adequacy and implementation conformance as empirical residuals.
---

# Proof–Experiment Boundary Closure

## What this is for

Before turning a Phase-2 hypothesis set into a Phase-3 experiment, check
whether formal reasoning already settles some of the hypotheses, so the
experiment that actually gets built tests only what remains genuinely
empirical. This narrows Phase 3's scope; it never replaces the part of
Phase 3 that has to run.

## What this is not for

- **Not a replacement for Phase 3.** The output of this skill is always
  either "nothing left to test" (rare) or a narrower residual handed
  to Phase 3 — never a substitute for running the residual experiment.
- **Not for Layer B or D questions.** Ownership and representation are
  conventions, not theorems. "Proving" who should own a field, or which
  encoding is correct, is a category error — there is nothing there to
  derive. Send these straight to Phase 3, or to Trigger 5
  (`research-governance.md`) if nothing about it is actually a matter
  of evidence at all.
- **Not for domains that resist formalization.** Organizational,
  social, or "what will actually happen when different people build
  this" questions rarely have a faithful formal model. If you can't
  honestly state one within a bounded attempt, stop trying and go to
  Phase 3. A forced formal model on an unformalizable domain produces
  false confidence, which is worse than no proof at all.

## Where this plugs into the main skill

```text
Phase 0 — kernel
Phase 1 — is this actually open
Phase 2 — competing hypotheses
        │
        ▼
  [this skill, optional] ← enters here, only with ≥2 hypotheses in hand
        │
        ▼
Phase 3 — discriminating experiment (scoped to the residual only)
Phase 4 onward — unchanged
```

Only load this skill after Phase 2 has produced at least two competing,
mutually exclusive hypotheses. Formalizing before hypotheses exist just
formalizes vague intuition — the "architecture by intuition" anti-pattern
wearing notation instead of prose.

## The gate: attempt formalization, but time-box it

For the hypothesis set from Phase 2:

```text
1. Can each hypothesis be restated as a claim about a formal object —
   a function, an invariant, an equivalence relation, a type, a
   composition law — built only from things already committed to (an
   existing contract, an existing kernel invariant, a stated
   definition)?

2. If yes: attempt the derivation, time-boxed. If a faithful model
   can't be honestly stated and a derivation attempted within that
   box, STOP. This is not a provable question today — fall through to
   Phase 3 unmodified. Don't extend the box "just a little longer";
   that's how a bounded gate becomes an open-ended detour.

3. If no — the hypotheses are actually about who owns something, what
   it's called, or what a person or team prefers — this is Layer B,
   Layer D, or a preference question. Do not attempt a proof. Go to
   Phase 3, or straight to Trigger 5 if nothing left is a matter of
   evidence.
```

This mirrors the boundary-isolation test's discipline: a small, fixed
set of yes/no questions run once per item, not a pipeline every question
is obligated to pass through (see `boundary-isolation-test.md` in the
main skill).

## Formal compression

If the gate says "attempt it," build the smallest formal model that lets
you state the hypotheses as claims about it — same discipline as Phase
3's Minimal Model. No object or law included "because it might matter."

```text
Model:          [the objects and the operation(s) under test, named]
Assumptions:    [every fact taken as given — cite its source: a kernel
                invariant, an existing contract, a stated definition.
                An assumption you can't cite is a Layer A/B/C decision
                smuggled in as a premise, not a real assumption.]
H1 as a claim:  [restate H1 about the model]
H2 as a claim:  [restate H2 about the model]
```

Then derive. A derivation is evidence only if it could, in principle,
have come out the other way — the same severity requirement Phase 3/4
impose on experiments. If the "proof" only restates the conclusion in
symbols, it isn't one — see "Proof theater," below.

## What a proof can and cannot close

```text
Proof can close:
    Layer A (necessity) — does the concept follow from what's already
        committed to, or is a formal counterexample constructible
        inside the stated model?
    Parts of Layer C (identity) — is the proposed "same as" relation
        actually an equivalence relation (reflexive, symmetric,
        transitive) under the model? Does it distinguish two things
        the hypotheses need distinguished?

Proof essentially never closes:
    Layer B (ownership) — a convention, not a theorem.
    Layer D (representation) — an encoding choice, not a theorem.
    Model adequacy itself — a proof is only as strong as the model's
        fidelity to the real system, and that fidelity is an empirical
        claim, not a formal one. It stays open even after a clean
        proof — see "What remains empirical," below.
```

If a derivation appears to close a Layer B or D question, that's a
signal some assumption secretly encoded a representation choice as a
premise. Go find which one did that; don't accept the conclusion as-is.

## What remains empirical, always

Even a correct, honest proof leaves at least this residual, every time:

| Residual question | Why the proof can't close it |
|---|---|
| Does the concrete implementation conform to the model? | A model is a description; conformance is a fact about code, checkable only by running it. |
| Is the model's own assumption set adequate to the real domain? | Assumptions were taken as given, not derived — a proof cannot certify its own premises. |
| Does the real system exhibit the predicted behavior under real execution conditions (timing, concurrency, partial failure)? | The model abstracted these away; whether that abstraction was safe is itself empirical. |
| Can a plausible-but-wrong implementation evade the invariant while appearing to satisfy it? | The proof shows the *model* can't do this; it says nothing about code that merely claims to implement the model. |

This table is the actual deliverable of this skill — not "the proof,"
but a short, explicit residual handed to Phase 3 as its scope. Phase 3's
experiment is then designed against this residual, not the original,
broader hypothesis set, which is what makes the resulting MVP smaller
without becoming weaker.

## The discrimination invariant

Narrowing to the residual must never narrow *what the experiment can
rule out*. State this explicitly before handing off to Phase 3:

```text
Pre-proof hypothesis set distinguished: [H1 vs H2 vs ...]
The proof closed:                       [which hypotheses/parts, and by
                                          which derivation step]
Phase 3 must still be able to discriminate:
                                         [what's left — must be
                                          non-empty, or there's no
                                          experiment left to design]
```

If the proof closes every hypothesis and nothing is left to discriminate,
there's no Phase-3 experiment: write a proof-only record (below), route
any remaining Layer-D question through Phase 8, and fold what the proof
settled into the kernel via Phase 9.

## Proof record

Use `references/proof-record-template.md` — deliberately mirrors the
main skill's `evidence-record-template.md` counterexample shape, so a
proof-based finding and an experiment-based finding read the same way
to someone auditing the kernel later.

## Feeding the kernel: proofs are not experiments

A kernel invariant earned by proof and one earned by repeated failed
falsification attempts are both real evidence, but not the same *kind*
of evidence, and collapsing them loses information a later reader needs.
When patching the kernel (main skill Phase 9), record the basis
explicitly:

```text
| Invariant | Layer | Basis | Model adequacy still open? | Evidence record | Introduced by |
```

`Basis` is `Proof` or `Experiment`. `Model adequacy still open?` is
`Yes` by default for anything closed by proof (see the residual table
above), and only becomes `No` once a separate conformance experiment has
confirmed the model actually matches the real system — cite that
experiment's own evidence record when it flips.

A `Proof`-basis invariant with adequacy still open is not yet the same
strength as an `Experiment`-basis invariant that survived Phase 5's
adversarial audit. The systematic re-evaluation bar in
`kernel-management.md` applies to both, but an unaudited proof is the
more likely of the two to be silently wrong. A flat "Confirmed" list
that doesn't preserve this distinction loses exactly the thing this
skill exists to protect.

## Governance hooks

- If the gate determines a supposedly-empirical residual is actually
  Layer B/D or preference, don't route it into a Phase 3 experiment "to
  be thorough" — that's precisely the failure Trigger 5
  (`research-governance.md`) exists to catch: resolving a preference
  question and presenting it as an architecture finding. Raise Trigger
  5 instead.
- If applying this gate causes the research question to change layer
  (a Layer A question turns out, once formalized, to reduce entirely to
  a Layer D encoding choice), that's Trigger 2 (question drift), and it
  fires here exactly as it would inside an ordinary experiment.

## Anti-patterns specific to this gate

**Proof theater.** Writing symbols that restate the hypothesis rather
than derive a consequence of it. If stripping the notation and reading
the "proof" in plain English shows it just repeats H1, it isn't a proof.

**Category error.** Attempting to "prove" an ownership or representation
choice. These aren't true or false; they're chosen. A convention has no
derivation.

**Silent premise smuggling.** An assumption listed as "given" that is
actually the Layer B/D decision under dispute, dressed up as a starting
condition so the "proof" reaches the desired conclusion.

**Treating model adequacy as free.** A clean derivation inside a model
says nothing about whether the model describes the real system. The
kernel table above exists specifically so this can't be silently
forgotten a few patches later.

**Open-ended formalization.** Spending real time building a model for a
domain that visibly resists it — see worked-examples.md's Example 4
(who owns a cross-team decision), which has no faithful formal model and
was correctly settled by case study, not proof. The time-box in the gate
exists to prevent this; respect it.

**Downgrading the residual experiment's rigor because "most of it is
proven now."** The discrimination-invariant check exists precisely so a
smaller residual doesn't quietly also become a less severe experiment.
Phase 3's negative-control and silent-wrong-answer requirements apply in
full to whatever residual remains, however small.

## Calibration against the existing worked examples

Not a retroactive audit obligation — just illustrative, for calibrating
when to reach for this gate going forward:

- **Idempotency key ownership** (worked-examples.md, Example 1) — H1/H2/H3
  are Layer B (ownership) questions dressed as reliability behavior. The
  gate stops at step 3: ownership isn't provable, it's chosen and then
  tested for consequences. Straight to Phase 3, exactly as it already
  ran.
- **Event schema-version identity** (Example 2) — "does an event need a
  version identity at all" (Layer A) is closer to formalizable: given a
  fixed meaning-changing transformation and a projector defined only on
  shape, a real derivation can show the projector can't distinguish two
  meanings without an outside oracle. "Which producer owns setting it"
  (H1 vs H2) is Layer B again — not provable. The gate would have split
  this into two questions and formalized only the first half.
- **Checkpoint interpretation identity** (Example 5) — the two-registries
  counterexample that falsified H1 is, in hindsight, closer to a
  constructive proof than a runtime experiment: it required only
  constructing two valid interpretations inside the stated model and
  showing they diverge, not executing real code under real timing or
  concurrency. This is exactly the shape of question this gate targets;
  running it as a full implementation experiment, rather than a
  model-level construction checked once for conformance, was probably
  more expensive than it needed to be.
