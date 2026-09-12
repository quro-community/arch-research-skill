# Worked Examples

Five short vignettes showing the method applied to genuinely different
domains. Each follows the same shape: question, competing hypotheses, the
discriminating experiment, what it found, and the implication — compressed
here to the essentials; a real write-up would use the full templates.
None of these are prescriptive answers for your system — they're here to
show the *shape* of a well-formed pass through the loop, including the
ones where the answer came back "unresolved, but narrower."

---

## 1. Distributed systems — who owns an idempotency key?

**Question.** A payment API needs to reject duplicate charges caused by
client retries. Does the idempotency key belong to the client, the
gateway, or the payment service itself?

**Competing hypotheses.**
- H1 — the payment service owns it: it derives a key from the charge's
  own content (amount, account, timestamp bucket).
- H2 — the client owns it: it generates and supplies a key per logical
  attempt, and the server just stores whatever it's given.
- H3 — the gateway owns it: it's assigned at the edge and threaded
  through, invisible to both the client and the payment service's own
  logic.

**Discriminating experiment.** Construct the case each hypothesis handles
differently: a genuine retry after a timeout, where the client doesn't
know if the first attempt succeeded. Under H1, two calls with identical
content but issued 400ms apart (crossing the service's timestamp bucket)
collide as two different keys — a false negative. Under H2, the same
literal retry (client reuses its own key on retry, by design) is
correctly deduplicated regardless of timing. Under H3, a gateway restart
between the two calls loses the assignment, and the retry gets a new key
— a false negative under exactly the condition idempotency exists to
survive.

**Result.** H1 falsified directly (timestamp-bucket boundary case). H3
falsified under the "component restart between retries" negative control
— the case specifically designed to hunt the silent-failure shape rather
than a crash. H2 survives every constructed case where the client-side
retry logic actually reuses its key, but a *new* dependency surfaces:
correctness now rests on client discipline the server can't verify,
which is a genuinely different risk than a server-derived key would
carry.

**Implication.** Ownership: client (H2), confirmed for the tested cases.
But this uncovered a Layer C (identity) question that wasn't part of the
original hypotheses: what makes two client-supplied keys "the same
attempt" from the server's point of view (exact string match? a hash of
declared-intent + key?) — logged as a follow-up, not resolved here.

---

## 2. Data platform — does an event need a schema-version identity, and who owns it?

**Question.** An event-sourced system replays historical events through
current projection logic. Does an event need to carry its own schema
version, and does the event store, the producer, or the projector own
that identity?

**Competing hypotheses.**
- H1 — the event store owns versioning globally (one store-wide schema
  version at any point in time).
- H2 — each event carries its own version, set by the producer at write
  time.
- H3 — no version is needed; projectors are written to tolerate any
  historical shape directly (structural typing, defaults for missing
  fields).

**Discriminating experiment.** Replay a stream that spans a schema change
where a field's *meaning* changed, not just its shape (a `status` field
whose enum gained a value that changes what an existing value means, not
just adds a new one — this is the sharper test; a merely-additive field
change tests nothing, because H3 handles it trivially).

**Result.** H3 falsified immediately: the projector can't tell, from
shape alone, which meaning of `status` applied to a given historical
event, and produces a plausible-looking but silently wrong projection —
no error, no refusal. This is a direct match for the silent-wrong-answer
failure shape the main skill flags as the one to weight most heavily.
H1 weakened, not falsified: it works as long as no two producers ever
write concurrently under different versions, which held in testing but
is a property of the deployment, not a property H1 guarantees on its own.
H2 confirmed as the only hypothesis that survives the meaning-change case
without an extra unstated assumption.

**Implication.** Per-event version identity is necessary (Layer A closes:
yes, it's needed). Ownership goes to the producer (Layer B closes: H2).
Left open on purpose: what the version identity actually needs to
*contain* to be a stable identity (a bare integer was sufficient in this
toy replay but is exactly the kind of Layer C shortcut that later broke a
comparable system elsewhere when two independently-versioned producers
turned out to both call their schema "v2") — flagged for a follow-up
experiment rather than assumed.

---

## 3. ML systems — does a prediction need a feature-computation identity?

**Question.** A model-serving system wants to guarantee that offline
evaluation and online serving compute features identically. Does an
inference request need an explicit identity for "which feature
computation produced these inputs," and if so, does the feature store or
the serving request own it?

**Competing hypotheses.**
- H1 — the feature store's current version is sufficient; serving always
  reads "latest," and that's the intended contract.
- H2 — the serving request must pin an explicit feature-computation
  identity, supplied by the caller, independent of the feature store's
  current state.
- H3 — no explicit identity is needed; bitwise-identical feature code
  between training and serving makes the question moot.

**Discriminating experiment.** Run an offline evaluation, then simulate a
feature backfill (a common, ordinary operation — correcting a historical
computation bug) between evaluation and the next online serving window,
then compare.

**Result.** H3 falsified on inspection — "identical code" says nothing
about which *data* a given historical prediction should be compared
against, and the backfill changes that data out from under H1. H1
falsified directly: the backfill silently changes what "latest" means
retroactively, so a prediction logged before the backfill and one logged
after are compared against inconsistent feature semantics with nothing
in the log signaling that. H2 survives: pinning an explicit computation
identity at request time makes the backfill's effect visible (predictions
before and after carry visibly different pinned identities) rather than
silent.

**Implication.** This is structurally the *same* shape of finding as the
distributed-systems and data-platform examples above, despite being a
completely different domain: the naive "use the current global state"
hypothesis (H1, in all three cases) is the one that keeps producing
silent-wrong-answer failures under an ordinary maintenance operation
(restart, replay-across-a-meaning-change, backfill), and an explicit,
request-carried identity is what converts that into something visible.
That recurrence across three unrelated domains is itself useful evidence
that this is a general pattern, not a coincidence of any one system.

---

## 4. Organizational design — who owns a cross-team decision?

**Question.** A decision spans two teams (e.g., a shared API's breaking
change policy). Does the *decision* live with a team, with the document
that records it, or with the review process that approved it?

This example is included on purpose: the method doesn't require code or
even a computer system. It requires a falsifiable question and a real
case to test it against.

**Competing hypotheses.**
- H1 — the owning team (whoever's system triggered the decision) owns it
  going forward.
- H2 — the document (an ADR, a design doc) is the owner, and "who wrote
  it" is a historical fact rather than an ongoing authority.
- H3 — the review process (whoever's sign-off was required to ship it)
  owns it, and authority moves with the reviewer roster over time.

**Discriminating experiment.** Not a code experiment — a case study,
using a real past decision. Find a decision that needed to be revisited
eight months later, after the original team's composition had
completely turned over, and check which of the three candidates actually
produced a correct, current answer when someone went looking for
"who can authorize changing this."

**Result.** H1 falsified in the observed case: "the owning team" had
turned over enough that nobody on it remembered the decision existed,
let alone its rationale — a silent loss of authority that looked, from
outside, like there being no problem at all (nobody complained; it just
quietly became unclear). H3 similarly weakened — reviewers rotate off
review committees and the mapping from "who reviewed it" to "who can
revisit it" degraded the same way. H2 survived: the document, kept
independent of team membership, was the only artifact that still
answered the question correctly eight months later, provided it recorded
*why*, not just *what* — a document that was only a decision log without
rationale would have failed the same test H1 failed.

**Implication.** This maps directly onto Layer B (ownership) and a
subtler point about Layer C: the "identity" of an ongoing decision-owner
is not stable if it's defined by *current team membership* — it needs to
be defined by something that survives personnel turnover, the same way
Example 1 found that identity tied to a timestamp bucket wasn't stable.
The mechanism differs (people vs. software) but the shape of the failure
— a binding to something that silently changes underneath the thing that's
supposed to be stable — is the same finding as Examples 1–3.

---

## 5. A generalized version of the case this skill was extracted from

**Question.** A workflow engine lets execution resume from a saved
checkpoint, long after (and possibly on a different machine than) the
run that created it. A checkpoint records which records/events are
needed to reconstruct state. Is that boundary — "which records" —
sufficient by itself for correct resumption, or is something else
required?

**Competing hypotheses.**
- H1 — the record boundary alone is sufficient; two checkpoints with the
  same records resume identically.
- H2 — resumption additionally requires an explicit, named
  **interpretation identity** — a declaration of *how* the records are
  meant to be read — that is not itself derivable from the records.

**Discriminating experiment.** Register two different, individually valid
readings of the same record stream inside the engine, then resume the
*same* checkpoint under each. If H1 is right, this shouldn't even be
constructible as a meaningful test — there'd be only one way to read a
given record set.

**Result.** H1 falsified directly: both readings resumed successfully,
and produced different, semantically incompatible states. Worse, a third
case — merging two branches that had each been read under a *different*
valid interpretation — resumed successfully under either single
interpretation while *silently discarding* the other branch's meaning.
No error. No refusal. A plausible-looking, wrong result — the same
silent-failure shape as Examples 2 and 3, now found inside the very
project this method was written up from.

**Implication.** H2 confirmed: interpretation identity is a required,
independent axis (Layer A closes: yes). It could not be treated as a
bare label, either — two registries answered to the identical label with
different meanings, so the identity needed to bind to actual content, not
a name (a Layer C finding, discovered only because the team kept asking
"what makes two of these the same" as a separate question from "does one
need to exist"). Left explicitly open: the merge/combination case above
is not closed by any mechanism tested so far, and was recorded as a
named, bounded open problem (via the boundary-isolation test) rather than
either ignored or used to block shipping the parts that *were* settled.

This is the pattern the whole method is built to produce: a clear
Confirmed/Falsified split on most of the question, one sharply-scoped
Unresolved item left over instead of a vague cloud of doubt, and a
written record of exactly what evidence would resolve it next.
