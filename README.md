# Architecture Research Through Falsifiable Inquiry

> Stop debating the design. Build the smallest thing that could prove a candidate wrong — and know when to stop building.

A portable agent skill for **AI-driven architecture research on questions where the answer is genuinely unknown** — where several designs are each plausible, where committing early creates an expensive trap, and where the real problem is not *"which design is correct"* but *"we don't yet have evidence that would tell us."*

This version assumes the **AI**, not a human, runs the research loop end to end: designing hypotheses, writing and executing experiments, auditing its own instrumentation, and maintaining a versioned **kernel** of settled findings — while actively watching its own trajectory for signs it should pause and hand a decision back to a human.

Applicable far beyond software: storage consistency models, payment idempotency contracts, ML pipeline reproducibility, even who owns a cross-team decision.

---

## Table of Contents

- [Why this exists](#why-this-exists)
- [Who drives this](#who-drives-this)
- [The core reframe](#the-core-reframe)
- [The four layers](#the-four-layers--and-why-collapsing-them-is-the-recurring-mistake)
- [The research loop](#the-research-loop)
- [The eleven phases](#the-eleven-phases)
- [Verdict taxonomy](#verdict-taxonomy)
- [The boundary-isolation test](#the-boundary-isolation-test)
- [The kernel](#the-kernel)
- [Research governance: checkpoints and retros](#research-governance-checkpoints-and-retros)
- [Anti-patterns](#anti-patterns)
- [Repository layout](#repository-layout)
- [Using this skill](#using-this-skill)
- [Acknowledgements](#acknowledgements)

---

## Why this exists

Some architecture questions have a known-good answer waiting to be looked up. **This skill is not for those.**

It is for the other kind: questions where several designs are each individually plausible, where committing early creates a trap that is expensive to back out of, and where the team's actual problem is *missing evidence* rather than *missing opinion*.

This skill exists to make the discipline concrete and repeatable, rather than a vague gesture at *"let's prototype it."*

## Who drives this

This skill assumes an AI agent runs Phases 0 through 10 — not a human relaying instructions to one. That changes what the human is for:

```text
The human owns:   intent, constraints, and value judgment — which
                   question actually matters, what trade-offs are
                   acceptable, and which of two equally-valid designs to
                   prefer once evidence alone can't decide.

The AI owns:      exploration execution, experiment design,
                   instrumentation, and evidence synthesis — and,
                   just as importantly, deciding when to stop and hand a
                   question back rather than keep running experiments.
```

A human collaborator shouldn't need to read every experiment's code to trust the process. That only holds up if the AI actively watches its own trajectory and interrupts itself when it should — see [Research governance](#research-governance-checkpoints-and-retros).

## The core reframe

Don't ask:

> "What is the correct architecture?"

Ask:

> "What is the smallest experiment that could prove a specific candidate architecture wrong?"

This is Karl Popper's basic move — a theory earns its keep by being **falsifiable**, not by being confirmed — combined with what philosophers of science call **severity**: an experiment is only evidence for a claim if the claim had a real chance of failing it. *"I ran it and it worked"* is weak evidence if failure was never really on the table.

The practical form is **strong inference** (J. Platt, 1964): hold at least two genuinely competing, mutually exclusive hypotheses at once, and design one experiment whose outcome discriminates between them. A single hypothesis invites confirmation-seeking, which is cheap and always available. Two hypotheses force you to design something that can actually kill one.

## The four layers — and why collapsing them is the recurring mistake

Almost every failure mode traces back to skipping straight from *"this needs to exist"* to *"here's the field for it,"* without passing through the two questions in between.

```text
Layer A — Necessity
    Must this concept exist for the system to be correct at all?

Layer B — Ownership
    Which component is authoritative for it?
    (Not: which component is convenient to put it in.)

Layer C — Identity
    What makes two instances of this thing "the same" one?
    An equivalence relation — very easy to smuggle in silently.

Layer D — Representation
    How is it actually encoded? Field, argument, reference,
    content hash, capability, out-of-band registry entry?
```

- A semantic requirement (A) does **not** imply an ownership decision (B).
- An ownership decision (B) does **not** imply you know what equality means for the thing (C).
- None of the above tells you the encoding (D).

Move through them **in order**. Don't let a data structure answer a question that hasn't been asked yet.

> **Layer C is the one people skip, and it's the one that bites hardest.**
> You can correctly settle *who owns X* and still get burned, because you never separately asked *what makes two X's equal*. A name is not automatically a stable identity — two different things can answer to the same name. Binding identity to actual content rather than a label is a distinct decision from ownership.

These distinctions appear under other names elsewhere: DDD's strategic vs. tactical design mirrors Necessity vs. Representation; a C4 diagram's "container" vs. "component" levels are a Layer B vs. Layer D split. The names don't matter — **the discipline of not collapsing them does.**

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

The output of one pass is **never** "the final design." It is: possibilities eliminated, invariants confirmed, and a sharper remaining question — or, increasingly often as a program matures, a clean signal that the question is answered and it's time to stop.

## The eleven phases

| Phase | Purpose |
|---|---|
| **0. Load the kernel** | Check whether the question is already answered by a Confirmed invariant, or contradicts one (a kernel challenge). Scope any genuinely new question as a delta against the kernel, not a rebuild from zero. |
| **1. Check whether this is actually an open question** | Detect questions that already assume their answer, e.g. *"Should the version field live on the record?"* (already picked Layer B + D) vs. *"Can the system reconstruct correct semantics without a version field, or is a counterexample constructible?"* |
| **2. Generate genuinely competing hypotheses** | At least two, mutually exclusive where possible, each predicting *different, observable* outcomes for the same test. Steelman the candidate you suspect is wrong. |
| **3. Design the discriminating experiment** | One-sentence objective, explicit scope (included/excluded, scoped against the kernel), transformations under test, negative controls, what counts as data, and what result falsifies which hypothesis. |
| **4. Pre-register before running** | Write the assertions before you see results. A green suite is not evidence unless the assertions could have gone red. Guards against HARKing. |
| **5. Run it, then adversarially audit your instrumentation** | Mutation-testing mindset: deliberately neuter the mechanism and confirm the "it worked" signal turns to "it failed." Treat harness bugs as findings, not embarrassments. |
| **6. Classify results against a living, minimal taxonomy** | Four verdicts only. Maintain a small domain-specific failure-class taxonomy that grows only when a genuinely new failure shape is observed. |
| **7. Write the evidence record** | Claim / grounds / warrant shape, with explicit non-conclusions and non-binary confidence. |
| **8. Apply the boundary-isolation test** | Decide item by item what can be safely deferred and what genuinely blocks. Isolated items graduate into the kernel's extension-point table. |
| **9. Fold results into the kernel as a versioned patch** | `Location | Before | After | Breaking?` — never a silent in-place rewrite. |
| **10. Gate promotion into the kernel** | Repeated evidence + a stated invariant + no counterexample + boundary test. Until then, keep experimental code physically separate with a one-way dependency. |

Running alongside all eleven phases: **Research governance**, which watches the whole trajectory for five checkpoint conditions and decides when to pause for a human retro instead of starting the next experiment. See below.

### Why pre-registration matters

Separate **observation** from **assertion**:

```text
observe:  code that records what happened, and asserts nothing.
assert:   code that names an invariant and fails loudly if violated,
          written against the hypotheses from Phase 2 — not against
          whatever the observations turned out to show.
```

This is the reform experimental psychology and medical trials adopted after learning, expensively, that hypotheses quietly rewritten to match results produce a record that *looks* rigorous and isn't.

## Verdict taxonomy

| Verdict | Meaning |
|---|---|
| **Confirmed** | Repeatedly supported, no counterexample found despite looking. |
| **Weakened** | Still plausible, but the naive/strong form is dead — a narrower form survives. |
| **Falsified** | A counterexample was constructed. Say what specifically broke it. |
| **Unresolved** | Not enough evidence either way. Say what evidence would resolve it. |

Resist inventing a fifth verdict for a special case — that's usually Layer A creep.

> **Give silent-wrong-answer failures more weight than loud-refusal failures**, deliberately, even though they're the same number of "bugs." A loud failure announces itself and is cheap to fix. A transformation that *succeeds* while producing a plausible-looking but semantically wrong result is the dangerous case: nothing points at it, and it surfaces later, further from its cause. Make sure at least one negative control specifically hunts this shape.

## The boundary-isolation test

The single most reusable tool in the method. Run it at the end of every pass, on every item still open.

```text
1. Can a boundary law be stated now — a constraint that must hold
   regardless of how the open content eventually gets filled in?
2. Is it safe to leave the CONTENT open — can no existing proof or
   law be silently violated while it stays unresolved?
3. Will filling in the content later be a superset refinement
   (strictly additive), not a breaking change to what's fixed now?
```

```text
YES to all three   →   ISOLATE
    Fix the boundary law now. Leave the content as an explicit,
    named extension point — not a vague TODO. It graduates into the
    kernel's extension-point table (see below), not just this
    pass's evidence record.

NO to any           →   CANNOT ISOLATE — GENUINELY BLOCKING
    Resolve it, at least minimally, before calling the
    surrounding design settled.
```

Full procedure and table format: [`references/boundary-isolation-test.md`](references/boundary-isolation-test.md)

## The kernel

The kernel is the versioned, authoritative record of what a research program has actually settled — separate from, and prior to, whatever the real system's shipped code says. It exists so a new MVP can depend on a settled invariant instead of re-deriving it, and so "is this still open?" has one place to check instead of an archaeology project through old patches.

A kernel snapshot has four parts: **Confirmed Invariants**, **Boundary Laws & Extension Points** (from the boundary-isolation test), a **Patch History** (the Phase 9 table, in order), and a **Kernel Challenge Log** (attempts to overturn a Confirmed invariant, successful or not).

Two disciplines keep it from failing in either direction:

- **MVP scope control.** Every new MVP's Included scope should be describable as *"the kernel, plus exactly this one new thing."* An MVP that quietly re-derives settled invariants has lost scope control before its first line of code.
- **Systematic re-evaluation.** A Confirmed invariant is a strong prior earned by surviving adversarial audit and the promotion gate — not an axiom immune to further evidence. Reopening one requires a genuinely new constructed case, a pre-registered falsification prediction, and a checkpoint raised *before* the challenge experiment runs.

Full structure, scope-control procedure, and challenge bar: [`references/kernel-management.md`](references/kernel-management.md)

## Research governance: checkpoints and retros

Running the phases well on a single question isn't sufficient across a whole research program — the AI also has to notice when the *trajectory* needs a human check-in, rather than mechanically starting the next experiment because the last one finished.

Five conditions warrant proposing a checkpoint:

| Trigger | What it looks like | What it means |
|---|---|---|
| **1. Evidence accumulation** | ~3-5 experiments/evidence records since the last checkpoint | A trajectory exists and deserves review as a whole, not just experiment-by-experiment |
| **2. Question drift** | The question moved from Layer A/B/C to Layer D without anyone deciding that | Also fires on any kernel challenge (Phase 0) |
| **3. Hypothesis expansion** | H1, H2 → H1, H2, H3, H4... instead of collapsing | Usually a wrong abstraction level or scope, not a need for more hypotheses |
| **4. Evidence saturation** | Everything open passes boundary-isolation as ISOLATE, and it's all Layer D | Architecture research is done; what's left is an implementation choice |
| **5. Human value check needed** | Two+ designs both survive every test; nothing left is a matter of evidence | A preference question, not a research question — no experiment will resolve it |

Before starting any experiment past the first in a trajectory, run a short self-check: *what decision could this change, what uncertainty does it target, why is this better than implementing/isolating/accepting, and what happens if research stops here?* No real answer to one of these → checkpoint instead of experiment.

At a checkpoint, the AI prepares a retro report (Objective, Timeline, Uncertainty Reduction, Drift Check, Architecture Extraction, and one of `CONTINUE RESEARCH` / `ISOLATE AND PROCEED` / `PROMOTE TO KERNEL` / `TRANSFER TO HUMAN DECISION`) so the human never has to reconstruct the trajectory themselves to make the call.

Full trigger mechanics and the human interaction rule: [`references/research-governance.md`](references/research-governance.md)
Retro report format: [`references/research-retro-template.md`](references/research-retro-template.md)

## Anti-patterns

| Anti-pattern | Why it fails |
|---|---|
| **Architecture by intuition** | "This feels right" is not evidence. If you can't say what observation would change your mind, you weren't doing research. |
| **Schema-first design** | Adding fields until a test goes green has no stopping point. Richness of representation is not evidence of semantic completeness. |
| **Implementation-proves-semantics** | "The code works, therefore the architecture is correct" mistakes a necessary condition for a sufficient one. A textbook Goodhart's Law case. |
| **Silent success** | "It ran without error" and "it produced the right semantics" are two claims that both need checking. |
| **Confirmation-seeking** | Five slightly different successful cases are an absence of falsification attempts, not a presence of severe testing. |
| **HARKing** | Adjusting the hypothesis after seeing results. If the "hypothesis" section was edited after the "results" section, the write-up is not evidence. |
| **Hiding unknowns in a generic bag object** | A `Context { everything unknown }` or `metadata: dict` doesn't resolve Layer A/B/C — it defers them somewhere less visible. |
| **Premature contract freezing** | "We need the final API before we can run experiments" gets the order backwards. Decide representation (Layer D) at the last responsible moment. |
| **Unbounded MVP scope creep** | An MVP that re-derives kernel-settled invariants or absorbs "final API design" back into scope is a half-built feature wearing an experiment's name. |
| **Kernel ossification** | Treating every kernel entry as permanently beyond question is premature freezing's mirror image — it also lets the evidence stop mattering. |
| **Experimenting past decision-relevance** | Running another experiment because the loop is comfortable, not because a decision needs it, is confirmation-seeking at the program level. |

## Repository layout

```text
arch-research/
├── SKILL.md                                # The skill itself — frontmatter + the full method
├── README.md                               # This file
├── LICENSE
└── references/
    ├── mvp-experiment-template.md          # Fill-in-the-blanks Phase 3 experiment plan
    ├── evidence-record-template.md         # Fill-in-the-blanks Phase 7 write-up
    ├── boundary-isolation-test.md          # Phase 8 decision procedure + table
    ├── worked-examples.md                  # Five full vignettes across five domains
    ├── kernel-management.md                # Kernel structure, MVP scope control, re-evaluation bar
    ├── research-governance.md              # Full checkpoint-trigger mechanics + human interaction rule
    └── research-retro-template.md          # Fill-in-the-blanks retro report for checkpoints
```

### Worked examples

Five short vignettes show the *shape* of a well-formed pass through the loop — including ones where the answer came back *"unresolved, but narrower."*

1. **Distributed systems** — who owns an idempotency key?
2. **Data platform** — does an event need a schema-version identity, and who owns it?
3. **ML systems** — does a prediction need a feature-computation identity?
4. **Organizational design** — who owns a cross-team decision? *(The method doesn't require code.)*
5. **Workflow engine** — is a record boundary sufficient for correct checkpoint resumption?

A recurring finding across examples 1–4: the naive *"use the current global state"* hypothesis keeps producing **silent-wrong-answer** failures under an ordinary maintenance operation (restart, replay across a meaning change, backfill, personnel turnover). An explicit, request-carried identity is what converts that silent failure into a visible one. That recurrence across unrelated domains is itself evidence the pattern is general.

See [`references/worked-examples.md`](references/worked-examples.md).

## Using this skill

This repository is a self-contained agent skill in the standard `SKILL.md` format — a YAML frontmatter block (`name`, `description`) followed by the method body, with `references/` holding the fill-in-the-blanks templates and the governance/kernel mechanics.

**Load it** by pointing any SKILL.md-compatible agent harness at this directory, or by including `SKILL.md` in the system prompt / skill search path.

**It triggers on questions like:**

- *"Not sure if this should live on X or Y."*
- *"Which of these designs is right?"*
- *"Let's spike this."*
- *"Prove this wrong before we build it."*
- *"Did this experiment actually show what we think it did?"*
- *"We've been running experiments on this for a while — are we still getting anywhere?"*

**Expected workflow:** consult `SKILL.md`'s Phase 0 to load the kernel → pick the relevant template from `references/` → fill it in **before** writing implementation code → run → audit instrumentation → write the evidence record → run the boundary-isolation test → fold into the kernel → run the self-check and, if a trigger fires, prepare a retro instead of starting the next experiment.

## Acknowledgements

This skill is a **collaborative synthesis**, distilled from an extended multi-model working session on real architecture research, and subsequently extended with AI-driven research-governance and kernel-management practices contributed in a follow-up round. The method, its layers, its templates, and its governance extension emerged from the dialogue between:

- **Claude / Sonnet-5** — Anthropic
- **OpenAI / ChatGPT-5**
- **DeepSeek / deepseek-v4-flash**

Each contributed complementary strengths: the falsification-first framing and experimental discipline, the four-layer decomposition and the boundary-isolation test, the worked examples that stress-test the method against genuinely unrelated domains, and the kernel/checkpoint governance layer that keeps an AI-driven research program from running an MVP out of control. The result is intended as a shared, portable artifact for any team — human-led or AI-led — doing architecture work under real uncertainty.

### Intellectual lineage

This method did not invent falsification, severity, or strong inference — it operationalizes them. It stands on:

- **Karl Popper** — falsifiability as the demarcation of empirical claims.
- **J. Platt (1964)** — *Strong Inference*: multiple competing hypotheses, one discriminating experiment.
- **D. Campbell / J. Stanley** — pre-registration and the separation of observation from assertion.
- **S. Toulmin** — the claim / grounds / warrant structure of the evidence record.
- **S. Goodhart** — the measure-as-target failure mode behind "implementation proves semantics."
- **Mutation testing, blameless postmortems, fail-safe design, anti-corruption layers** — the engineering practices Phase 5, Phase 6, and Phase 10 borrow from.
- **Strategic management's "stage-gate" and R&D portfolio review practice** — the model behind treating a research trajectory, not just a single experiment, as something that periodically needs an explicit go/no-go review.

---

<div align="center">

**A good architecture research process does not prevent wrong ideas.**
**It makes wrong ideas cheap to discover, and it makes the discovery legible to someone who wasn't in the room.**

*The measure of progress is not how much architecture got built. It is how much genuine uncertainty got removed — and whether the next person can tell exactly which uncertainty is gone and which remains. Increasingly, it's also whether the process itself knew when to stop.*

</div>
