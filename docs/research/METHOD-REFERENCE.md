# Event Modeling — Method Reference

The method as its authors define it, reconciled across sources that do not entirely agree.

`METHOD-REFERENCE-DETAIL.md` is the long-form companion. This document is the part you need
to build a model correctly, plus an honest account of where the sources conflict.

---

## Provenance

Built by reading primary sources directly, not from web search or recall:

| Source | Form | Where |
|---|---|---|
| Adam Dymitruk, *"Event Modeling: What is it?"* (2019, updated to 2025) | ~25 KB Markdown | `archive/eventmodeling-org/posts/what-is-event-modeling/` |
| eventmodeling.org cheat sheet (sbortz, 2020, updated 2022) | Markdown | `archive/eventmodeling-org/posts/event-modeling-cheatsheet/` |
| Martin Dilger, *Understanding Eventsourcing* | 650 pp PDF, read in full | not archived — commercial |
| Martin Dilger, Event Modeling Cheat Sheet (2026-07) | 2 pp PDF, read as images | not archived — supplied privately |
| `@eventmodelers/agent-modeling-kit` v0.1.74 | MIT, ~650 KB of skills | `archive/eventmodelers-kits/` |
| EMSL specification | MIT | `archive/eventmodeling-toolkit/` |

eventmodeling.org is blocked by this environment's egress policy, but its content is
maintained as Markdown in a **public GitHub repository**, which is reachable. See
`ACCESS-NOTES.md`.

---

## Source precedence

Event Modeling has evolved. Where sources conflict, resolve in this order:

1. **Recent material by Adam Dymitruk or Martin Dilger** — the method's author and its most
   active practitioner. Later beats earlier.
2. **Earlier material by them** — still authoritative for anything unrevised.
3. **Third-party material on canonical sites** — useful, sometimes the clearest statement of an
   idea, but it does not override (1) or (2). This includes the eventmodeling.org cheat sheet,
   which is authored by `sbortz`, not by Dymitruk.
4. **Everything else.**

Dates for ranking:

| Source | Date | Author |
|---|---|---|
| Dilger, Event Modeling Cheat Sheet | **2026-07** | Dilger |
| `@eventmodelers/agent-modeling-kit` | **2026** (v0.1.74, actively published) | Dilger |
| Dilger, *Understanding Eventsourcing* | 2025-06 print; Leanpub continuously updated | Dilger |
| Dymitruk, *"Event Modeling: What is it?"* | 2019, **content updated through 2025-10** | Dymitruk |
| eventmodeling.org cheat sheet | 2020, updated 2022 | ⚠️ `sbortz` |

Note that Dymitruk's foundational article is still maintained — it is not a 2019 artefact — so it
ranks high on both authorship and recency.

Caveat: recency does not make a source infallible. The 2026 kit contains a confirmed color error
(see [`UPSTREAM-DEFECTS.md`](UPSTREAM-DEFECTS.md)) that older material gets right. Prefer recent,
but check against the rest.

---

## Reconciliation: the sources disagree

Three framings exist. None is wrong; they are different vintages and audiences. Read any
diagram against the vocabulary of *its own* source.

### Building blocks

| | Dymitruk 2019 | eventmodeling.org cheat sheet 2022 | Dilger 2026 |
|---|---|---|---|
| Count | **3** + wireframes | **4** | **5** |
| Blocks | Command, Event, View | Trigger, Command, Event, View | Screen, Command, Event, Read Model, Processor |

Dymitruk: *"Event Modeling only uses 3 moving pieces and 4 patterns based on 2 ideas."*
Dilger promotes the trigger and the processor to first-class elements. Same method,
finer granularity.

### Pattern names

| Dymitruk 2019 / cheat sheet 2022 | Dilger 2026 |
|---|---|
| Command pattern | **State Change** |
| View pattern | **State View** |
| Translation | Translation |
| Automation | Automation |

Both are canonical. "Command pattern" is not an error.

### Colors — settled

| Element | Color |
|---|---|
| **Command** | **blue** — invariant across every source |
| **View / Read Model** | **green** — invariant across every source except one defective file |
| **Event** | **orange** (current) or **yellow** (older material). Both legitimate. |
| Screen / Trigger | white |
| Processor | gear |
| External event | yellow |
| Error / hotspot | red-pink |

**This project uses orange for Event.**

Event's color is the only one that genuinely moved: yellow in the 2022 material, orange in
Dilger's 2026 cheat sheet and book figures. Command and View never moved.

> ⚠️ **One source is wrong.** The kit's `facilitating-event-modeling-workshops.md` transposes
> Event and View (green/orange). It contradicts its own author's cheat sheet and book figures,
> and it is the only place in the entire corpus making that claim. Documented with exact line
> numbers in [`UPSTREAM-DEFECTS.md`](UPSTREAM-DEFECTS.md) — **do not follow it.**

---

## The two ideas

Underneath everything:

> *"Empowering the user and informing the user."* — Dymitruk

That is the whole method. Every column either **writes** (a command producing events) or
**reads** (events producing a projection). See *Column types* below.

---

## The five elements

| Element | Gloss (Dilger 2026) | Naming |
|---|---|---|
| **Event** | "Something that happened in the system" | Past tense — `OrderPlaced`, never `PlaceOrder` |
| **Command** | "Something that we want to happen in the system" | Present-tense verb — `PlaceOrder` |
| **Read Model** | "Making sense of what happened in the system" | Named for the question it answers |
| **Screen** | "How users make things happen in the system" | Human roles only |
| **Processor** | "Side effect of what happened in the System" | System actors; drawn as gears |

### Systems with no UI

The governing rule is Dilger's, 2026, and it is stated in element terms:

> *"Human roles get SCREEN nodes. System actors and processors get AUTOMATION nodes. Place both
> types during storyboarding — do not defer automations to a later step."*
> — `eventmodeling-storyboarding-events/SKILL.md`

> *"When a processor or system actor reacts to events automatically (no human interaction), place
> an **AUTOMATION** node in the actor row instead of a SCREEN."* — *ibid.*

⚠️ **Do not over-apply this.** "System actor" here means an *internal or integrated processor* —
`eventmodeling-identifying-outputs` glosses it as *"scheduler, background job, external trigger."*
It does **not** mean "any non-human client."

The distinguishing test is Dymitruk's, and it is about todo lists, not about humans:

> *"the idea of a 'todo list' for some processor **in our system**"*

…and automations *"may even actually be manual todo lists that our employees use"* — so a human can
be an automation, and a machine can be a role. What decides it is whether the thing works a todo
list we own, or initiates against us from outside.

An external client — even a fully automated one — is an ordinary **actor role** and gets screens.
A protocol client is the clear case: a text protocol like SMTP can be driven by a person typing
into `telnet`, and the server cannot tell the difference, so the element type cannot depend on it.

The **Trigger** framing corroborates it, and is the clearest prose statement of the idea:

> *"It can be a user via a UI or it can be some external piece of software calling our public API.
> Or it can even be a robot aka an automated process. Describe it via a simple wireframe **or the
> route of an http endpoint**."*

⚠️ But note the attribution: that passage is the **eventmodeling.org cheat sheet, authored by
`sbortz`** — hosted on the canonical site, not written by Dymitruk. Under *Source precedence* it
is tier 3. Cite it as illustration, not as authority; the load is carried by the kit.

Screens are in any case optional — `analyze-existing-model` warns against flagging *"a slice named
'Internal' with no SCREEN"* as a gap.

### Field metadata

Every element carries `meta.fields`. `COMMAND`, `SCREEN` and `READMODEL` fields each require a
**`mapping`** recording provenance. **Events are the exception** — no mapping, and a bare event
with no fields is legitimate for a simple state transition.

| Element | Legal `mapping` values |
|---|---|
| Command | `user-input`, `session:<f>`, `<Event>.<f>`, `derived:<expr>`, `webhook:<f>` |
| Command screen | `<Command>.<f>`, `session:<f>`, `<Event>.<f>`, `derived:<expr>` |
| View screen | `<ReadModel>.<f>`, `<Event>.<f>`, `derived:<expr>` |
| Read model | `<Event>.<f>`, `latest:<Event>.<f>`, `aggregate:<Event>.<f>`, `derived:<expr>` |

> *"An event without fields is an opaque label — it cannot be used to validate completeness,
> write scenarios, or generate code."* … *"Do not pad events with fields just to reach a count."*

The strongest procedural rule in the kit:

> *"When you encounter a mapping that cannot resolve to a connected element, write the mapping
> as-is but flag it as a gap in the completeness notes. **Do not invent connections that don't
> exist.**"*

---

## Column types

> **Naming note.** This section uses the corpus's own vocabulary. FnEmail's documents call these
> **Command Slice** and **View Slice** — see `../event-model.md`. The mapping is one-to-one:
> write column = Command Slice, read column = View Slice. Corpus terms are kept here so quotations
> and cross-references to the sources stay checkable.

The four patterns reduce to **two** column shapes:

```
write column:  Trigger/Processor  →  COMMAND     →  EVENT(s)
read  column:  EVENT(s)           →  READ MODEL  →  Trigger/Processor
```

| Pattern | Composition | Distinguished by |
|---|---|---|
| State Change | write | trigger is a Screen |
| State View | read | consumer is a Screen |
| Automation | read + write | trigger is a Processor |
| Translation | read + write | source events are **external** (yellow) |

This is not an interpretation. Dymitruk specifies automation as **two** specs:

> *"The specification for this is always done in 2 parts: The Given-When-Then for creating the
> to-do list and the Given-When-Then for executing the command."*

And the kit forbids mixing:

> *"A slice never mixes a COMMAND and a READMODEL… that's **two slices**, not one."*
> — `eventmodeling-slicing-event-models/SKILL.md`

Translation is not even a slice type — it is an automation whose input events belong to someone
else's system.

Keep both altitudes: **two column types** is the structure, **four patterns** is the workshop
vocabulary. Rule 18 is *"optimize for conversation."*

### The connection rule

> *"It's imperative to always follow the only way that these elements connect: command to event,
> event to state view, state view to UI or processor, UI/processor to command. **Connecting an
> event directly to a command is not allowed.**"* — Dymitruk

This constrains legal adjacency, independent of whether arrows are drawn.

### View placement follows the consumer

> *"A read model must be placed in a column that already contains a SCREEN or AUTOMATION it
> serves. Do not place read models in columns with no screen or automation — doing so creates
> orphaned read models that will never have a consumer."*
> — `eventmodeling-identifying-outputs/SKILL.md`

A read model is placed where it is **used**, not next to the events it projects. Its sources may
lie arbitrarily far back on the timeline — Dymitruk's step 5 speaks of information *"accumulated"*
by storing events. A view exists to serve the column it sits in.

---

## The seven steps

Dymitruk's headings, verbatim:

| # | Step | Goal |
|---|---|---|
| 1 | Brain Storming | Collect all events. *"only state-changing events are to be specified"* |
| 2 | The Plot | Arrange into a plausible story on one line |
| 3 | The Story Board | Wireframes/triggers on top. §3.1 **UX Concurrency** |
| 4 | Identify Inputs | The blue commands |
| 5 | Identify Outputs | The views/read-models |
| 6 | Apply Conway's Law | Organize **events** into ownership swimlanes |
| 7 | Elaborate Scenarios | Given/When/Then |
| — | Completeness Check | Unnumbered in the article; **step 8** in the kit |

Dilger's 2026 cheat sheet compresses to **six**, merging steps 4 and 5 into `04 - Input / Output`.
Do not cite a step count without naming the source.

---

## Swimlanes — the word means two things

Both senses are Dymitruk's own, and they are different lanes on different rows.

**Actor swimlanes (step 3.1)** — top of the board, one per actor:

> *"We show wireframes or web page mockups across the top. These can be organized in
> **swim-lanes to show different people (or sometimes systems)** interacting with our system.
> We also show any automation here with a symbol like gears."*

An operator/admin lane carrying technical or verbose output is squarely within this. Constraint:

> *"There are **no screens that appear above one another** as we need to capture each change in
> the system state as a separate vertical slice."*

**Ownership swimlanes (step 6)** — the event rows, per team or system:

> *"organizing the events themselves into swimlanes… to allow the system to exist as a set of
> autonomous parts that separate teams can own."*

Dilger's cheat sheet uses the word only in the second sense: *"Swimlanes can be Teams or Systems.
Swimlanes indicate ownership of Events (Data)."*

---

## Slices

A slice is one column: **exactly one command, or exactly one read model**. Never both.

> *"Each workflow step is tied to either a command or a view/read-model."* — Dymitruk, step 7

- Named after that command or read model
- Independently deployable; communicates with other slices **only** via events
- *"Keep slices small. One business capability per slice."* (Rule 14/15)
- Every column holding a command or read model **must** have a slice defined, or it can never be
  built as a feature

---

## Scenarios

| Kind | GIVEN | WHEN | THEN |
|---|---|---|---|
| State Change | events | command | event(s) |
| State View | events | **query** | read model |
| Error | events | command | **error** (red) |

Dymitruk: *"each specification is tied to exactly one command or view."* Also
`Given-When-Then == "Arrange, Act, Assert" == (in UX) "Situation, Motivation, Value"`.

---

## The completeness check

> *"every field accounted for. All information has to have an **origin and a destination**.
> Events must facilitate this transition and hold the necessary fields to do so."* — Dymitruk

**Bidirectional.** Backward-only tracing is half the check; the forward half is what exposes
events nothing consumes.

The kit adds hard rules:

- **No calculated events.** *"Totals, averages, counts are read models, NOT events."*
- *"If a value changes multiple times, it's a read model."*
- Processor outputs split: facts → events; calculations → read models; notifications → neither
- Every column with a command or read model has a slice
- Workflow step contracts: explicit pre/postconditions per step, so teams can build in parallel

---

## The four anti-patterns

Dilger 2026. *"Those Patterns are not red flags, but something to keep an eye on. I first look
for them on every model."*

| Name | Shape |
|---|---|
| **left chair** | 1 Command → * Events |
| **right chair** | 1 Read Model ← * Events |
| **bed** | 1 Screen → * Commands |
| **shelf** | 1 Slice with * Scenarios, when all other slices have none |

---

## Notation extensions (Dilger 2026)

- **Hotspots** — red sticky for open questions, blockers, disputes. *"Unknowns become hotspots."*
- **Actor lanes** — group screens by who is viewing
- **Chapters** — blue arrow above a group of slices
- **UI-only interaction** — screens in sequence, no commands or events
- **Slice status** — Informational / Planned / In Progress / Done / Blocked
- *"Drawing a screen must not take longer than 2 minutes."*

Arrows are notation, not semantics. Meaning is carried by lane position, left-to-right time, and
which pattern a column matches. This project does not depend on arrow notation.

---

## The 20 rules

1. Start with events. 2. Model facts, not ideas. 3. Time flows left → right. 4. If the order is
unclear, create a hotspot. 5. Name by intent. 6. Use only four patterns. 7. State Change · State
View · Automation · Translation. 8. Every command has a reason. 9. Never add commands "just
because." 10. Every Read Model answers a question. 11. No question, no Read Model. 12. Don't
guess. 13. Unknowns become hotspots. 14. Keep slices small. 15. One business capability per
slice. 16. Keep it simple. 17. Complexity usually hides a problem. 18. Optimize for conversation.
19. Shared understanding beats pretty diagrams. 20. Model behavior, not structure.

Most important question: **"What happens next?"**

---

## When it does not fit

The corpus is thin here. What it does say:

- Screens are optional — `analyze-existing-model` warns against flagging *"a slice named
  'Internal' with no SCREEN"* as a gap.
- Views are passive: *"cannot reject an event after it's been stored."*
- Dymitruk's adoption argument cuts both ways — effectiveness *"is inversely proportional to the
  amount of learning individuals must do."* A model requiring extensive explanation is suspect.

No source in this corpus addresses protocol-level or infrastructure modeling directly. Applying
it to RFC 5321 is therefore an extension of the method's demonstrated range, and should be
labelled as such rather than presented as settled practice.
