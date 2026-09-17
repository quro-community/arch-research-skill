# The Boundary-Isolation Test

The most reusable tool in this whole method. Use it at the end of every
research pass, on every item still marked open, before deciding what
happens next.

## The problem it solves

After a research pass, you'll have a pile of findings sorted into
Confirmed / Weakened / Falsified / Unresolved (main skill, Phase 6). The
Unresolved pile creates a trap either direction:

- Treat everything in it as blocking, and the team never ships anything,
  because there's always one more open question.
- Treat everything in it as fine to ignore, and a genuinely load-bearing
  unknown gets silently frozen into a contract because nobody stopped to
  check whether it was safe to defer.

The boundary-isolation test is how you tell these apart, item by item,
instead of making one blanket call for the whole pile.

## The test

For each open item, ask four questions in order:

```text
1. Can a boundary law be stated now?

   Not "what is the full content of this thing," but: is there a
   constraint that MUST hold regardless of how the open content is
   eventually filled in? If you can write that constraint down as a
   sentence today, the answer is yes.

2. Is it safe to leave the CONTENT open?

   Could leaving this unresolved silently violate something already
   proven or already frozen elsewhere? If no existing guarantee depends
   on this item's content being pinned down, the answer is yes.

3. Will filling in the content later be a superset refinement?

   When someone eventually does resolve this, will that resolution be
   strictly additive to what's fixed now (a new case, a new field, a new
   registered option) — or would it require breaking/changing something
   that's already been committed to? If additive, yes.

4. Is filling it in SOMEONE'S JOB, with a trigger?

   Not "could it be filled in later" — who, and triggered by what? An
   item that will be forced by the next build but has no owner and no
   trigger is ORPHANED, and orphaned items do not stay isolated. They
   come due all at once, at implementation time, with no evidence
   behind them.
```

**Why question 4 exists.** Questions 1–3 are answered against the
*proofs*: "could leaving this unresolved silently violate something
already proven?" That is the correct test for **freezing**, and it is the
wrong test for **building**. An item can be perfectly safe against every
proof and still be a forced decision the moment anyone lands the feature
it constrains.

And landing is not a research event. No retro, checkpoint, or evidence
record is triggered by "someone started building", so nothing in the loop
will ever schedule an orphaned item. Four passes can each correctly
return ISOLATE, and the fifth produces a queue of decisions nobody
researched.

## The verdict

```text
YES to all four    →   ISOLATE
    Fix the boundary law now. Leave the content as an explicit,
    named extension point — not a vague TODO, a real named slot
    with the boundary law attached to it.

NO to 1, 2 or 3    →   CANNOT ISOLATE — GENUINELY BLOCKING
    This has to be resolved, at least minimally, before the
    surrounding design can honestly be called settled. Don't
    isolate around it; that just hides a real gap behind a
    reassuring label.

YES to 1-3,
NO to 4            →   ISOLATE — ORPHANED
    Safe to leave open, and due at the next build, with nobody
    scheduled to fill it. Fix the boundary law as above, then move
    the item to a DECISION QUEUE: a named owner, a named trigger,
    and an honest note on whether any evidence can inform the
    choice or whether it is a preference the domain must settle.
    An orphaned item does not need more research. It needs someone
    to decide.
```

## Worked shape (fill in your own item)

```text
Item:  [the open question]

1. Boundary law statable now?
   [yes/no + the one-sentence law, if yes]

2. Open content safe to leave open?
   [yes/no + what existing guarantee it might threaten, if no]

3. Later fill = superset refinement?
   [yes/no + what would have to break, if no]

Verdict: ISOLATE / CANNOT ISOLATE

If isolate — the boundary law:
    [stated as a constraint, e.g. "any implementation of X must never
    silently substitute a default when the identity is unresolved;
    it must fail explicitly"]

If cannot isolate — what's needed to close it minimally:
    [the smallest experiment or decision that would move this to
    ISOLATE or fully Confirmed]
```

## Why this beats "just mark it as tech debt"

A generic "tech debt" or "TODO" label doesn't distinguish a genuinely
inert unknown from a live landmine. Running the three questions forces you
to actually check whether something downstream already silently assumes
an answer to the open question — which is exactly the failure mode this
whole method exists to catch early. It also produces, for free, a written
boundary law that constrains whatever eventually fills the gap in, so the
later resolution can be checked against something concrete instead of
against vibes.

## Applying it across a whole pass

Run it as a table, not one item at a time in isolation — seeing every open
item's verdict next to each other makes it obvious when several
"isolated" items actually share one underlying unresolved question (a
sign to merge them into a single follow-up experiment rather than three
separate vague ones).

| Item | (1) Law statable? | (2) Content safe open? | (3) Superset later? | (4) Owner + trigger? | Verdict |
|---|---|---|---|---|---|
| | | | | | |

The column-4 entries are the ones worth reading twice. A table of clean
`ISOLATE` verdicts with an empty owner column is not a settled design —
it is a decision queue that has not been acknowledged as one. If several
orphaned items share a trigger (all due the first time anyone builds a
particular module, say), that is a scheduling fact worth acting on now,
while it is one queue instead of seven separate surprises.
