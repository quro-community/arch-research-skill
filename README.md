# Architecture Research Through Falsifiable Inquiry

> Stop debating the design. Build the smallest thing that could prove a candidate wrong.

A portable agent skill for **architecture research on questions where the answer is genuinely unknown** — where several designs are each plausible, where committing early creates an expensive trap, and where the real problem is not *"which design is correct"* but *"we don't yet have evidence that would tell us."*

Applicable far beyond software: storage consistency models, payment idempotency contracts, ML pipeline reproducibility, even who owns a cross-team decision.

---

## Table of Contents

- [Why this exists](#why-this-exists)
- [The core reframe](#the-core-reframe)
- [The four layers](#the-four-layers--and-why-collapsing-them-is-the-recurring-mistake)
- [The research loop](#the-research-loop)
- [The ten phases](#the-ten-phases)
- [Verdict taxonomy](#verdict-taxonomy)
- [The boundary-isolation test](#the-boundary-isolation-test)
- [Anti-patterns](#anti-patterns)
- [Repository layout](#repository-layout)
- [Using this skill](#using-this-skill)
- [Acknowledgements](#acknowledgements)

---

## Why this exists

Some architecture questions have a known-good answer waiting to be looked up. **This skill is not for those.**

It is for the other kind: questions where several designs are each individually plausible, where committing early creates a trap that is expensive to back out of, and where the team's actual problem is *missing evidence* rather than *missing opinion*.

This skill exists to make the discipline concrete and repeatable, rather than a vague gesture at *"let's prototype it."*

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

The output of one pass is **never** "the final design." It is: possibilities eliminated, invariants confirmed, and a sharper remaining question. Treat that as success.

## The ten phases

| Phase | Purpose |
|---|---|
| **1. Check whether this is actually an open question** | Detect questions that already assume their answer, e.g. *"Should the version field live on the record?"* (already picked Layer B + D) vs. *"Can the system reconstruct correct semantics without a version field, or is a counterexample constructible?"* |
| **2. Generate genuinely competing hypotheses** | At least two, mutually exclusive where possible, each predicting *different, observable* outcomes for the same test. Steelman the candidate you suspect is wrong. |
| **3. Design the discriminating experiment** | One-sentence objective, explicit scope (included/excluded), transformations under test, negative controls, what counts as data, and what result falsifies which hypothesis. |
| **4. Pre-register before running** | Write the assertions before you see results. A green suite is not evidence unless the assertions could have gone red. Guards against HARKing. |
| **5. Run it, then adversarially audit your instrumentation** | Mutation-testing mindset: deliberately neuter the mechanism and confirm the "it worked" signal turns to "it failed." Treat harness bugs as findings, not embarrassments. |
| **6. Classify results against a living, minimal taxonomy** | Four verdicts only. Maintain a small domain-specific failure-class taxonomy that grows only when a genuinely new failure shape is observed. |
| **7. Write the evidence record** | Claim / grounds / warrant shape, with explicit non-conclusions and non-binary confidence. |
| **8. Apply the boundary-isolation test** | Decide item by item what can be safely deferred and what genuinely blocks. |
| **9. Fold results in as a versioned patch** | `Location | Before | After | Breaking?` — never a silent in-place rewrite. |
| **10. Gate promotion into the real system** | Repeated evidence + a stated invariant + no counterexample + boundary test. Until then, keep experimental code physically separate with a one-way dependency. |

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
    named extension point — not a vague TODO.

NO to any           →   CANNOT ISOLATE — GENUINELY BLOCKING
    Resolve it, at least minimally, before calling the
    surrounding design settled.
```

This is what lets a team keep shipping the parts that *are* settled without (a) freezing something the evidence doesn't support, or (b) treating everything as blocked because one piece is still open. It also produces, for free, a written boundary law that constrains whatever eventually fills the gap.

Full procedure and table format: [`references/boundary-isolation-test.md`](references/boundary-isolation-test.md)

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

## Repository layout

```text
arch-research/
├── SKILL.md                                # The skill itself — frontmatter + the full method
└── references/
    ├── mvp-experiment-template.md          # Fill-in-the-blanks Phase 3 experiment plan
    ├── evidence-record-template.md         # Fill-in-the-blanks Phase 7 write-up
    ├── boundary-isolation-test.md          # Phase 8 decision procedure + table
    └── worked-examples.md                  # Five full vignettes across five domains
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

This repository is a self-contained agent skill in the standard `SKILL.md` format — a YAML frontmatter block (`name`, `description`) followed by the method body, with `references/` holding the fill-in-the-blanks templates.

**Load it** by pointing any SKILL.md-compatible agent harness at this directory, or by including `SKILL.md` in the system prompt / skill search path.

**It triggers on questions like:**

- *"Not sure if this should live on X or Y."*
- *"Which of these designs is right?"*
- *"Let's spike this."*
- *"Prove this wrong before we build it."*
- *"Did this experiment actually show what we think it did?"*

**Expected workflow:** read `SKILL.md` → pick the relevant template from `references/` → fill it in **before** writing implementation code → run → audit instrumentation → write the evidence record → run the boundary-isolation test → patch, don't rewrite.

## Acknowledgements

This skill is a **collaborative synthesis**, distilled from an extended multi-model working session on real architecture research. The method, its layers, and its templates emerged from the dialogue between:

- **Claude / Sonnet-5** — Anthropic
- **OpenAI / ChatGPT-5**
- **DeepSeek / deepseek-v4-flash**

Each contributed complementary strengths: the falsification-first framing and experimental discipline, the four-layer decomposition and the boundary-isolation test, and the worked examples that stress-test the method against genuinely unrelated domains. The result is intended as a shared, portable artifact for any team doing architecture work under real uncertainty.

### Intellectual lineage

This method did not invent falsification, severity, or strong inference — it operationalizes them. It stands on:

- **Karl Popper** — falsifiability as the demarcation of empirical claims.
- **J. Platt (1964)** — *Strong Inference*: multiple competing hypotheses, one discriminating experiment.
- **D. Campbell / J. Stanley** — pre-registration and the separation of observation from assertion.
- **S. Toulmin** — the claim / grounds / warrant structure of the evidence record.
- **S. Goodhart** — the measure-as-target failure mode behind "implementation proves semantics."
- **Mutation testing, blameless postmortems, fail-safe design, anti-corruption layers** — the engineering practices Phase 5, Phase 6, and Phase 10 borrow from.

---

<div align="center">

**A good architecture research process does not prevent wrong ideas.**
**It makes wrong ideas cheap to discover, and it makes the discovery legible to someone who wasn't in the room.**

*The measure of progress is not how much architecture got built. It is how much genuine uncertainty got removed — and whether the next person can tell exactly which uncertainty is gone and which remains.*

</div>
