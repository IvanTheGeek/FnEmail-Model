# Extensions to Event Modeling — parked

Ideas for extending Event Modeling beyond what Dymitruk and Dilger describe.

**Deliberately not applied to `event-model.md`.** The orthodox model is being built first, by
the book, so that any extension can be measured against it rather than blended into it. Nothing
here should leak into the orthodox model (currently v0.3).

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

⚠️ **Correction.** An earlier note held that FnEmail's H3 (`RecipientDirectory` unsourced) was the
strongest motivation for this extension. It is not. H3 resolved as a **separate context** —
different actors, timescales, normative source and change cadence — and separate contexts joined by
Translation are orthodox Event Modeling, needing no extension at all. See `event-model.md`,
*H3 resolved*.

This extension is narrower than it looked: it is about splitting **one** context into layers
(protocol / domain / operational), where every layer shares a charter and an owner. That case is
still unsupported by the corpus, and the kit still has no mechanism for a second model. But it must
not be justified with examples that are really separate contexts.

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
  **A contract is a GWT with the example data removed** — the schema of which a step is one
  inhabitant. `GIVEN` is the precondition, `THEN` the postcondition, `WHEN` the step itself.
  Contracts are therefore not extra work on top of scenarios; they are the same artifact typed.
- **Example data is mandated**: *"Also put all relevant information in it. The more realistic the
  data the better."*

### Why RFC 5321 is an unusually good testbed

**The RFC ships the paths.** ✅ Verified 2026-08-05 against the archived text
(`research/archive/rfc/rfc5321.txt`). Appendix D contains four worked scenarios — D.1 typical
transaction, D.2 aborted transaction, D.3 relayed mail (two steps), D.4 verifying and sending —
each a complete wire trace with concrete addresses, domains and reply codes.

Under this extension those are not documentation. They are **promoted acceptance tests authored
by the standards body**, and RFC conformance becomes "these paths pass." Most domains require
inventing example data; here the normative spec supplies it.

⚠️ **Two qualifications found on verification:**

1. **Every example uses `EHLO`** — five occurrences, zero of `HELO`. Under a HELO-only scope the
   paths are RFC-*derived*, not RFC-*quoted*. A promotion scheme needs to distinguish the two,
   because only one of them is evidence of conformance.
2. **The scenario labelled "typical" is not the clean one.** D.1 contains a `550` rejection
   mid-flow, as does D.2. The clean single-recipient paths are D.3 and D.4. Whatever picks the
   "golden path" cannot just take the one the source calls typical.

### The tension worth keeping

Dymitruk's flat-cost-curve argument rests on slice independence: *"implementing any other
workflow step does not cause the need to revisit this already complete workflow step."* Paths
explicitly exercise cross-slice composition — the thing that claim says needs no checking.

That is the valuable part, not a conflict. A path that fails while every constituent step passes
has found coupling the method says should not exist. No current artifact surfaces that.

### Open design questions

1. **Does example data belong to the step or the path?** **Settled: the step.** A step is a
   *(column, data)* pair — an identified atom that participates in many paths. An earlier draft
   argued the opposite (column as slot, path supplies data) on the grounds that data-in-step
   "collapses back into scenarios." That was wrong: a scenario is per-slice GWT, whereas a step
   is a reusable node. Data-in-step is also the only version that supports §8, since a real event
   log needs identified atoms to match against.
   Open sub-question: **when are two steps the same step?** Without an answer, every distinct data
   value spawns a node and the catalog stops being navigable.
2. **What makes composition legal?** Step contracts are the obvious answer, but they must
   actually exist first. Nothing in `event-model.md` v0.2 has them yet.
3. **What bounds the trek space?** Free composition explodes combinatorially. "Meaningful" needs
   a promotion criterion — coverage-driven, risk-driven, or explicit curation.
   **Terminology, settled:** *happy* is descriptive (the path reaches its intended successful
   outcome; many paths qualify); *golden* is a **designation** (the coverage baseline, the viewer
   default, what documentation leads with). Being clean does not make a path golden. So the
   question is not "find the golden path" but "**decide which path to designate golden, and by what
   criterion**" — a product decision, not something discoverable in the model.
   **Open:** outcome appears to be relative to an actor. In the first worked path the session is a
   complete success for the server, partial for the sender, and a failure for one recipient. Should
   a path carry an outcome per actor lane rather than a single verdict?
4. **Test-level mapping.** Step → slice test, path → integration, trek → acceptance is the
   intuitive reading. Stating it determines whether promotion is mechanical or a judgement call.
5. **On the board or beside it?** Drawing paths on the timeline risks re-creating the branching
   mess the single timeline exists to prevent.
6. **Relationship to Dilger's "chapters"** (blue arrow grouping slices) — coarse grouping that
   already exists. Is a chapter a degenerate path, or an orthogonal concept?
7. **Naming — avoid "workflow" for the business-level grouping.** Dymitruk already uses *workflow
   step* for one column, i.e. the technical unit. Three distinct things are in play and two are
   already named:
   | Thing | Name | Altitude |
   |---|---|---|
   | One column | **workflow step** (Dymitruk) | technical unit |
   | A named business grouping | **chapter** (Dilger) | business narrative |
   | A concrete traversal with data | **path** (this extension) | execution instance |
   Caveat: a chapter groups *adjacent* slices. If business groupings must span non-contiguous
   slices, chapters need extending from "contiguous group" to "named semantic group" — at which
   point it may be a fourth construct rather than a reuse.

### Prerequisite

Step contracts must be added to the orthodox model before any of this can be built. That is
already listed under *Deferred* in `event-model.md` — this extension is the reason to prioritise
it.

## 4. Two classes of rule: normative vs. operator

Some rules are **hard-lined** — an external authority defines them and the system has no
discretion. RFC 5321 for a mail server; IRS regulations for tax software. Others are **operator
or business configuration** — chosen locally, within what the hard rules permit.

The clean example: RFC 5321 defines what a valid email address *can* be. A given server operator
may accept only a subset. Both are rules; only one is negotiable.

Implications to work out:

- Are these two kinds of scenario, two kinds of slice, or two models?
- Under §3, RFC-defined paths are **hard-lined** — failing one is a conformance defect. Operator
  paths are per-deployment and their expected outcomes vary by config. That means a path needs a
  *class*, and the promoted test suite is really two suites.
- Does an operator rule ever *loosen* a normative one? If never, that is a checkable invariant:
  the operator's accepted set must be a subset of the RFC's.

**The normative set is a versioned graph, not a document.** RFC 5321 is updated by RFC 7504, which
adds two reply codes. So "RFC-conformant" is a claim about a *set* of documents at a point in time,
and the set can grow without our model changing. A conformance claim has to name its set and its
date, or it silently rots.

**Normative rules can contain operator choices.** RFC 7504 §3 is a clean specimen: it hard-lines
*when* `521` must be used, then says that afterwards the server **MAY** continue replying `521` or
**MAY** close the connection. So the two classes are not cleanly separable per-rule — a single
normative rule can delegate part of itself. Any scheme that tags rules as hard or soft needs to
handle rules that are both.

## 5. Business errors vs. technical errors as separate models

Dymitruk is firm that the model shows **business/domain** errors, not technical ones.

The extension: model both, in **separate models**, and let a viewer merge them — with visibility
selectable by the reader. The domain model stays clean, and the technical detail exists without
polluting it.

This is where the SMTP work keeps landing: `DataPhaseEntered` and a `503` sequencing error are
protocol facts, not domain facts, and forcing them onto one timeline is what makes H1 and H2
awkward. Two models dissolves both questions.

Related: Dilger appears to advocate multiple domain models directly, stacking a short HELO line
above or below a fuller EHLO model — which is the same move applied to protocol variants rather
than to error classes.

## 6. The live model viewer

Paths, models and layers are only useful if they can be selected. The intended artifact is a
**live viewer**, not a static board:

- Choose a path or trek, or step through interactively
- Choose which models to include (domain / protocol / GUI)
- Choose depth and layer

With **persona defaults**: a CEO view that is very high level; a technical-ops view; an accounting
view; a contributor view that follows exactly what happens on the technical side. One artifact,
many audiences — the same model serving as user documentation, onboarding, and technical
reference depending on which knobs are set.

This is also the answer to §3's question 5 (board or beside it): **neither** — paths are selected
in a viewer, so the static board never branches.

## 7. Graphs

If steps are nodes and paths are walks, the model is a graph and graph tooling applies directly:

- **Forensics** — an operator's event log is a walk. Which known walks reach this error node, and
  where does the log diverge? (see §8)
- **Coverage** — promotion becomes spanning-tree-plus-unique-edges rather than a judgement call
- **Reachability** — which slices can never be reached? which errors have no path to them?

## 8. Paths as a support and diagnosis artifact

A use for §3 that has nothing to do with testing.

An operator hits an error and hands over their event log. Because every event leading to the
error is on the path, the failure can be traced back through the exact sequence that produced it
— not reconstructed from guesswork or partial logs.

This is a strong argument for **data living in the step** (§3, question 1). If data lives on the
path, an incoming log has no atoms to match against. If steps are identified nodes, the log maps
onto a known sequence and the point of divergence is detectable mechanically.

**The mechanism is already canonical.** Dilger, *Handling Metadata* (p547):

> *"If a problem arises with a command, correlation and causation IDs enable us to see exactly
> what the user did before the problem and, if necessary, replay all actions to restore the system
> to the exact state at the time the command was issued."*

So correlation/causation already give you the trail and the replay. What this extension adds is
the **known path to diff against** — replay tells you what happened, the path catalog tells you
what was *supposed* to happen and where the two diverge. That is the part that does not exist yet.

Practical consequence: metadata design is a prerequisite alongside step contracts. A path catalog
is only matchable against real logs if those logs carry correlation and causation IDs from day one.

## 9. (open)

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
