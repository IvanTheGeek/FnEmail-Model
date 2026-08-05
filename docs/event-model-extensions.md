# Extensions to Event Modeling — parked

Ideas for extending Event Modeling beyond what Dymitruk and Dilger describe.

**Deliberately not applied to `event-model.md`.** The orthodox model is being built first, by
the book, so that any extension can be measured against it rather than blended into it. Nothing
here should leak into v0.2.

Status: **captured, not developed.**

---

## 1. Multiple models for different aspects

One event model per *aspect* of the system rather than one model for the whole thing.

The trigger for this was a concrete question in the SMTP model: is `DataPhaseEntered` a domain
fact or protocol bookkeeping? Under one model, that question has no good answer — protocol state
and domain state compete for the same timeline and the same completeness check. Split into two
models and each becomes coherent on its own terms.

Candidate aspects for FnEmail:

- **Protocol model** — session sequencing, verbs, reply codes, RFC conformance
- **Domain model** — mail as a business fact: accepted, delivered, bounced
- **Operational model** — queues, retries, capacity

Open questions this raises:

- What is the interface between models? Presumably events — one model's event is another's
  external (yellow) event, i.e. the Translation pattern doing real work between *our own* models.
- Does the completeness check run per model, or across the set? If per model, what stops a field
  being complete locally and orphaned globally?
- Does this multiply the ownership swimlanes, or replace them?

Prior art to check: whether Dilger's *Dynamic Consistency Boundary* chapter (p626) is addressing
a related problem from a different direction.

## 2. GUI as its own model

A corollary of (1): the user interface gets its own event model rather than occupying the
storyboard row of the system model.

Motivation: the storyboard row conflates two things — *what triggers a state change* and *what a
human sees*. For a protocol server these are barely related. Dymitruk's actor swimlanes already
gesture at the separation by letting one lane be an admin and another a user; this takes it
further and gives the UI a timeline of its own.

Open question: if the GUI is its own model, is a screen still an element of the system model at
all, or does it become purely a consumer of the system model's read models?

## 3. Steps, Paths and Treks — example data as the test suite

The largest of the three, and the one with the clearest payoff.

### The idea

- Every column/slice traversal is a **step**, and a step carries **specific sample data**.
- An ordered run of steps is a **path**.
- Multiple paths exist over the same column inventory.
- Paths compose into larger paths, and ultimately into a **trek** through the whole system.
- Meaningful steps/paths/treks are **promoted** to become the actual tests that verify the
  system — at whatever test level each corresponds to.

The model stops being a document that tests are written *from* and becomes the artifact the
tests *are*.

### What it fixes

Event Modeling deliberately shows **one** non-branching way through. Dymitruk is explicit that
this is the point — branching timelines are what make flowcharts unreadable. The cost is that
alternate flows exist only as per-slice scenarios: they never compose into a narrative, so
nothing in the model describes *a whole session going wrong*.

This keeps the single timeline and moves multiplicity into a **second dimension** — paths over
the same columns — instead of branching the board. The property Dymitruk is protecting survives;
what it costs is recovered.

### Canonical anchors

Not as far from orthodox as it looks. Three existing pieces, never joined up:

- **"Step" is Dymitruk's own term.** *"Each workflow step is tied to either a command or a
  view/read-model"*, and *"a workflow step is considered to be repeated on the event model if it
  uses the same command or view."* He already separates the column from the traversal of it.
- **Workflow Step Contracts already exist** in `eventmodeling-checking-completeness` — explicit
  pre/postconditions per step, defined so teams can work in parallel, and otherwise unused. They
  are exactly the path-joining rule: A+B composes iff A's postconditions satisfy B's
  preconditions.
- **Example data is mandated**: *"Also put all relevant information in it. The more realistic the
  data the better."*

### Why RFC 5321 is an unusually good testbed

**The RFC ships the paths.** Appendix D contains worked example sessions — D.1 typical
transaction, D.2 aborted transaction, D.3 relayed mail, D.4 verifying and sending — each a
complete wire trace with concrete addresses, domains and reply codes.

Under this extension those are not documentation. They are **promoted acceptance tests authored
by the standards body**, and RFC conformance becomes "these paths pass." Most domains require
inventing example data; here the normative spec supplies it.

*(Verify the appendix contents against the RFC text before relying on this.)*

### The tension worth keeping

Dymitruk's flat-cost-curve argument rests on slice independence: *"implementing any other
workflow step does not cause the need to revisit this already complete workflow step."* Paths
explicitly exercise cross-slice composition — the thing that claim says needs no checking.

That is the valuable part, not a conflict. A path that fails while every constituent step passes
has found coupling the method says should not exist. No current artifact surfaces that.

### Open design questions

1. **Does example data belong to the step or the path?** Preferred reading: the column is a
   *slot*, the path supplies the data — so one `RcptTo` column appears in many paths with
   different addresses. Putting data on the step instead collapses back into scenarios and gains
   nothing.
2. **What makes composition legal?** Step contracts are the obvious answer, but they must
   actually exist first. Nothing in `event-model.md` v0.2 has them yet.
3. **What bounds the trek space?** Free composition explodes combinatorially. "Meaningful" needs
   a promotion criterion — coverage-driven, risk-driven, or explicit curation.
4. **Test-level mapping.** Step → slice test, path → integration, trek → acceptance is the
   intuitive reading. Stating it determines whether promotion is mechanical or a judgement call.
5. **On the board or beside it?** Drawing paths on the timeline risks re-creating the branching
   mess the single timeline exists to prevent.
6. **Relationship to Dilger's "chapters"** (blue arrow grouping slices) — coarse grouping that
   already exists. Is a chapter a degenerate path, or an orthogonal concept?

### Prerequisite

Step contracts must be added to the orthodox model before any of this can be built. That is
already listed under *Deferred* in `event-model.md` — this extension is the reason to prioritise
it.

## 4. (open)

Space for what emerges next.

---

## Method notes that are *not* extensions

Recorded here because they were initially mistaken for extensions and turned out to be orthodox —
they belong in `research/METHOD-REFERENCE.md`, not on this list:

- **Admin/operator actor swimlane** carrying verbose or technical output. Canonical: Dymitruk's
  step 3.1 permits swimlanes for *"different people (or sometimes systems)."*
- **A view need not sit next to the events it projects.** Canonical: read models are placed in the
  column of the screen or processor they serve; sources may be arbitrarily far back.
- **Protocol verbs as triggers without wireframes.** Canonical: eventmodeling.org's cheat sheet
  admits *"the route of an http endpoint"* as a trigger.
