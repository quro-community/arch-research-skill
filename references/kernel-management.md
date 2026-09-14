# Kernel Management

Use this whenever Phase 0 asks you to load, consult, or update the
kernel, and whenever an MVP's scope (Phase 3) needs to be checked against
what's already settled. This file exists to answer "build the MVP
package from zero every time" — the single biggest source of research
programs that generate a lot of experiments and very little compounding
knowledge.

## What the kernel is

The kernel is the versioned, authoritative record of what a research
program has actually settled about one system or subsystem — kept
separate from, and prior to, whatever the real system's shipped code
says. It exists so that:

- a new MVP can depend on a settled invariant instead of re-deriving it
- a reader can find out what's known without re-reading every evidence
  record ever written
- "is this still open?" has one place to check, instead of requiring an
  archaeology project through old patches

The kernel is not the code. It's the layer between research and code:
what the code should eventually implement, and what the next MVP is
allowed to assume without re-proving it.

## Kernel structure

A kernel snapshot has four parts. Keep them as one living document
(`kernel.md`, or whatever the project's convention is) that gets
patched, never silently rewritten:

```markdown
# [System/Subsystem] Kernel

> Version: [n]
> Last patch: [date / patch reference]

## Confirmed Invariants

Each entry: the invariant, the Layer(s) it settles (A/B/C/D), the
evidence record that promoted it (Phase 10), and the patch that
introduced it (Phase 9).

| Invariant | Layer | Evidence record | Introduced by |
|---|---|---|---|

## Boundary Laws & Extension Points

Items that passed the boundary-isolation test as ISOLATE (Phase 8): the
content is still open, but a constraint on it is fixed. Each entry names
the extension point and states its boundary law as a real constraint,
not a TODO.

| Extension point | Boundary law | Isolated by |
|---|---|---|

## Patch History

The full `Location | Before | After | Breaking?` table from every
Phase 9 patch, in order. This is the paper trail that lets anyone ask
"did this new evidence falsify something previously frozen, or just
advance an already-open item?" without guessing.

## Kernel Challenge Log

Any attempt to overturn a Confirmed Invariant (see below), whether it
succeeded or not. A challenge that failed is still worth recording — it's
evidence the invariant survived another severe test.
```

## MVP scope control: scope as a delta, not a rebuild

Before writing Phase 3's Scope section for any MVP, do this:

```text
1. List every kernel invariant that touches this question's system.

2. For each, ask: does this MVP need to RE-PROVE it, or can the MVP just
   ASSUME it and build on top?

   → If the MVP can assume it, exclude it from scope explicitly. Name it
     in "Excluded" as "assumes [invariant], per kernel v[n]" — this is
     not the same as ignoring it; it's citing it.

   → If the MVP genuinely needs to re-test it (e.g. a new domain
     property might violate it — see worked-examples.md's recurring
     "naive current-global-state" pattern for how this actually shows
     up), that's a kernel challenge, not ordinary scope. Route it
     through "Systematic re-evaluation" below before writing Phase 3.

3. The MVP's Included scope should be describable in one sentence as:
   "the kernel, plus exactly this one new thing."
```

An MVP whose Included list quietly grows to cover ground the kernel
already settled is the clearest early signal of the "hypothesis
expansion" and "unbounded MVP scope creep" failure modes (see the main
skill's Research governance section and Anti-patterns) — catch it here,
before a single test case gets written, rather than at the retro.

## Systematic re-evaluation: the kernel is a strong prior, not an axiom

A Confirmed Invariant earned its place by surviving Phase 5's adversarial
audit and Phase 10's promotion gate. That's real evidence, and it should
change how much scrutiny a routine question gets. It should not become
immunity from scrutiny forever — that's "premature contract freezing" and
"kernel ossification" (see the main skill's anti-patterns) wearing a
different hat: the belief was frozen once, correctly, and then treated as
though freezing it made it true regardless of what shows up later.

The bar for reopening a Confirmed Invariant is deliberately higher than
for an ordinary new question, not absent:

```text
To challenge a kernel invariant, the challenger needs:

    a constructed case the invariant's original evidence record did NOT
    test (not a restatement of a case it already survived)
        +
    a stated prediction of what result would falsify the invariant under
    this new case, written BEFORE running it (Phase 4 applies in full)
        +
    a Trigger-2 checkpoint (question drift) raised BEFORE the experiment
    runs, because overturning a settled invariant can invalidate whatever
    downstream work already assumed it — the human should know a
    challenge is in flight, not just see the result afterward
```

If the challenge succeeds, it's a Falsified verdict against a *previous*
promotion, not against the new experiment — write it up as a patch to the
kernel (Phase 9) that explicitly supersedes the old entry, with the old
entry kept in the Patch History rather than deleted. Silently deleting a
superseded invariant destroys exactly the paper trail the kernel exists
to provide.

If the challenge fails, that's not wasted effort: log it in the Kernel
Challenge Log as another severe test the invariant survived, which is
itself evidence worth having on record the next time someone wonders
whether the invariant still holds.

## Relationship to the boundary-isolation test

Everything that passes Phase 8 as ISOLATE graduates into the kernel's
"Boundary Laws & Extension Points" table, not just into that pass's
evidence record. That's what makes the isolation useful across MVPs
instead of just within one: the next MVP that touches the same extension
point reads the boundary law off the kernel instead of rediscovering it
from scratch.

## A minimal worked shape

```text
Question:  Does the payment service need its own idempotency key, or can
           it rely on the gateway's?

Kernel check (Phase 0):
    Confirmed Invariants table already has: "idempotency key ownership:
    client-supplied, per Example 1 in worked-examples.md, evidence
    record EK-003."
    → This question is already answered. Cite EK-003. Do not open a new
      experiment for it.

New, genuinely open question surfaces instead:
    "What makes two client-supplied keys the same attempt?" — this was
    logged in EK-003 as a follow-up, not resolved there.

Scope for the new MVP:
    Included: identity/equivalence basis for client-supplied keys under
              retry, merge, and replay.
    Excluded: ownership (assumes EK-003, kernel v2) — cite, don't re-test.
```
