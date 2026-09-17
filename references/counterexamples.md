# Counterexamples — Where This Method's Own Discipline Was Not Enough

Every rule in the main skill was earned by a failure. This file records a
failure mode the rules do **not** yet catch, taken from a real programme:
four milestones (`M1`–`M4`), twelve experiment trees, 902 experiment tests,
four clean closures, four retros, and a drift from engineering value that
none of that rigour detected.

Read this when a programme looks healthy. It is written for that moment,
because every entry here happened *while* the method was working exactly as
specified — controls fired, hypotheses were falsified, pre-registrations
held, retros ran. Nothing here is an argument that the method is wrong.
It is an argument that the method has a blind spot, and it is measurable.

---

## What did NOT fail

State this first, or the rest reads as a rewrite.

```text
Falsification worked      hypotheses died when they should (two general claims
                          falsified across the programme, each with a named
                          narrow form surviving)
Controls worked           every injected defect fired its named class; two
                          controls were *found* to measure nothing and were
                          rebuilt rather than kept
Pre-registration held     the criteria written before the runs were not edited;
                          where operationalizations were wrong they were
                          corrected and BOTH versions kept in the artifact
Instrumentation audit     six harness defects were self-caught, including a
worked                    check that passed a case declaring nothing at all
Kernel separation held    nothing in the production tree ever depended on a
                          disposable experiment; the read-the-source check
                          caught a violation the day it was introduced
```

The blind spot is not rigour. It is that **every one of those checks is
local to a window**, and drift happens *between* windows.

---

## CE-1 — The exhausted baseline

```text
Symptom   After the roadmap's last item closed, each new stage was generated
          by asking "what did the previous stage leave behind?" — residuals,
          carried debts, its own lemma generalized. Four retros ran and none
          asked whether the ROADMAP still had items.
Evidence  The programme's own roadmap named three experiments. All three ran,
          plus a fourth that came from an earlier roadmap. It had no successor
          item, and nothing recorded that fact. Every subsequent candidate came
          from the previous milestone's leftover list.
Mechanism R0/R5 recalibrate against a NAMED BASELINE. When the baseline is
          exhausted, recalibrating against it produces a valid delta against a
          plan that has no work left — and "here is what's left over" keeps
          producing plausible, narrow, valid questions forever.
Fix       Research governance gains a sixth trigger, and R5 of the retro gains
          an exhaustion check: does the baseline still have unstarted items?
          If not, choosing the next question is a ROADMAP decision, not a
          research decision, and it needs the wider input (a design corpus, a
          phase plan, a consumer) that a residual list cannot supply.
```

## CE-2 — The unapplied patch

```text
Symptom   The canonical documents stopped describing the system, and no
          per-window audit could see it, because each window audits only its
          own record.
Evidence  Five independent staleness defects, found only when three corpora
          were reconciled by hand: a design doc documenting a two-argument
          interface that had three; a status table still listing three
          mechanisms as "deferred" after all three closed; two laws minted by
          closures and absent from the printed register; a README frozen three
          phases back; a README referencing a file renamed away.
Mechanism Phase 9 writes a patch. Nothing applies it to the documents a reader
          actually consults. The closure document — which is written by the
          window — is current; the shared document — which nobody owns — rots.
          A register that under-reports is worse than no register, because it
          is consulted.
Fix       Phase 9's patch must name the canonical documents it makes stale and
          update them in the same pass. Phase 10 adds: a finding that is
          recorded only in its own evidence record has not been folded into the
          kernel — it has been filed next to it.
```

## CE-3 — The orphaned isolation

```text
Symptom   The boundary-isolation test returned ISOLATE for a growing pile of
          items. Years of "safely deferred" turned into a queue of decisions
          that landed all at once, with no research behind any of them.
Evidence  When the programme finally asked "what would it take to build this",
          seven decisions came due simultaneously — ownership of a manifest, an
          interpretation binding, three record schemas, a default route, and two
          questions that are not the framework's to answer at all. Every one had
          a recorded ISOLATE verdict. None had an owner.
Mechanism Question 2 asks "is it safe to leave the CONTENT open?" and answers it
          against the proofs: nothing already frozen depends on it. That is the
          right test for freezing and the wrong test for building. An item can
          be perfectly safe against every proof and still be a forced decision
          the moment anyone lands the feature — and landing is not a research
          event, so no retro will ever schedule it.
Fix       The isolation test gains a fourth question (§ the reference), and a
          fourth verdict: ISOLATE-ORPHANED — safe, unnamed, and therefore due
          at the next build. Orphaned items go into a decision queue with a
          trigger, not into a "safely deferred" list.
```

## CE-4 — The mechanism-scoped milestone

```text
Symptom   Full-cost experiments whose headline outcome a stated rule had
          already predicted.
Evidence  A design document's own membership rule predicted, before any
          experiment ran, that four operations would each resolve the same way.
          All four did. Two of them spent a full milestone producing one
          genuinely new finding each — real findings, at a price that the
          uncertainty did not justify, because the milestone was scoped as a
          MECHANISM ("does folding need a primitive?") rather than as an
          UNCERTAINTY ("is there any operation a declared record cannot
          express?").
Mechanism The self-check asks "what decision could this change?" — necessary
          and not sufficient. It does not ask whether the outcome is already
          predicted by something stated, in which case the experiment is a
          confirmation purchased at discovery cost.
Fix       Phase 3 gains a prior-prediction check: if a stated rule, invariant
          or contract already predicts this experiment's outcome, say so in the
          design and justify the cost — usually by re-scoping to the uncertainty
          that the rule does NOT settle.
```

## CE-5 — The uncollected founding answer

```text
Symptom   The question the whole programme was opened to answer was answered,
          and no document said so.
Evidence  The founding divergence was written down at the start as a single
          question. Four milestones answered it. It appears in no design doc, no
          closure, no freeze set and no retro — because each window reports its
          own findings, and "the programme's founding question, answered"
          belongs to no window.
Mechanism Phase 7 writes a record per experiment. Phase 9 patches the kernel.
          Nothing in the loop is scoped to the PROGRAMME rather than to a
          window, so a cross-window answer has no home.
Fix       A retro over a completed range must state the programme's founding
          question and its answer in one place, in one paragraph, or record
          explicitly that it is not yet answered. This is cheap and it is the
          artifact a consumer actually needs.
```

## CE-6 — Purpose-free rigour

```text
Symptom   Four milestones of impeccable internal discipline, and the record
          never once referenced the reason the project existed.
Evidence  A purpose statement existed outside every corpus. No design document,
          specification or milestone referenced it. The programme optimised what
          it could measure — falsifiability — and never checked back against
          what it was for. The costs of that gap were visible only from outside:
          twelve experiment trees, zero production lines.
Mechanism "The human owns intent" is stated in the skill, and the loop never
          reads intent. Vigour is not the same as direction; a research
          programme can be exactly right about everything except what it is
          for, and no amount of falsification detects this.
Fix       Phase 0 loads the kernel. It must also load the PURPOSE, and if none
          is written down, say so and ask for one before generating hypotheses.
          A governance trigger fires when a window's output cannot be traced to
          the stated purpose.
```

## CE-7 — Rigour that becomes the product

```text
Symptom   The evidence record became the deliverable, and it is not what the
          consumer needed.
Evidence  The load-bearing content of the whole programme fits in about one
          page: three inputs to the reconstruction call, a declared-payload
          discipline, a reachability requirement, an append-only law. Around it:
          four numbering namespaces that disagreed with each other, a
          per-case argument template, and cross-cutting patterns correctly
          forbidden promotion. The abstraction was not hollow — it was correct,
          small, and buried.
Mechanism The method's output format is per-experiment evidence. When a
          programme runs long enough, the format's volume starts to read as the
          programme's value, and the one-page answer inside it becomes harder to
          find with each passing window.
Fix       Final principle gains a size check: if the settled content can be
          stated compactly, stating it compactly IS the deliverable. Volume of
          evidence is a cost paid by every future reader, not a proxy for
          having established something.
```

---

## How to use this file

```text
Run the method unchanged — every rule in it is earned.
Then, at each retro, run these seven checks against the PROGRAMME rather than
the window:

  1  Is the baseline that generated these questions still unexhausted?
  2  Do the canonical documents still describe the system?
  3  Does every isolated item have a named owner and a trigger?
  4  Did any experiment confirm what a stated rule already predicted?
  5  Is the founding question answered, in one place?
  6  Can this window's output be traced to the stated purpose?
  7  Can the settled content be stated compactly — and was it?

A "no" on any of 1–7 is not a defect in the research. It is the class of
defect that four clean closures, 902 tests and four retros did not catch,
and it is cheaper to check than to discover.
```
