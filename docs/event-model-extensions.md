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

## 3. (open)

Space for what emerges once the orthodox model is on paper.

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
