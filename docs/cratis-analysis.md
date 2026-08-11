# Cratis — an analysis, 2026-08-11

**Class: trail.** Not normative, and nothing lags it (AGENTS.md rules 13 and 14). No ruling is made
here, no settled row is reopened, and no document is asked to reconcile to it. It is a dated survey
of an outside platform that is building, right now, several of the things this project is working
toward — recorded so the next reader meets the evidence rather than rediscovering it.

Prompted by the owner, 2026-08-11: *"my next thing that I was just reviewing https://www.cratis.io/
and its repos. this looks like something we should investigate."*

⚠️ **`cratis.io` itself was unreachable from this session** — the egress proxy blocks the domain, and
blocks `mermaid.js.org` and `ingebrigtsen.blog` too. Everything below is sourced from the GitHub
repositories instead, read directly rather than through a summarizer (rule 3). Where a claim comes
from a page I could not open, it is marked as such and not relied on. The documentation site may
say more, and may say it differently.

---

## 1. What Cratis is

A .NET platform for event-sourced applications, MIT-licensed throughout, at
[github.com/Cratis](https://github.com/Cratis). It is organized as one repository per product, and
the products are named for parts of telling a story.

| Repo | What it is | Read here as |
|---|---|---|
| [`Chronicle`](https://github.com/Cratis/Chronicle) | the event sourcing engine — kernel, .NET client SDK, MongoDB storage, Orleans runtime, web workbench | infrastructure; not our layer |
| [`Arc`](https://github.com/Cratis/Arc) | CQRS application model over ASP.NET Core, with TypeScript proxy generation | infrastructure; not our layer |
| [`Screenplay`](https://github.com/Cratis/Screenplay) | **a declarative modeling language — one `.play` file describes a whole system as typed slices** | **the significant one** |
| [`Stage`](https://github.com/Cratis/Stage) | **runs an authored event model directly, with no code generation** | **the significant one** |
| [`Prologue`](https://github.com/Cratis/Prologue) | captures what an existing system does and interprets it into an event model | interesting, not ours (we are greenfield) |
| [`Narrator`](https://github.com/Cratis/Narrator) | VS Code browser for a running Chronicle event store | tooling |
| `Studio` | referenced by Stage, Screenplay and Prologue as the visual authoring tool | **not publicly readable as of today** |
| `Fundamentals`, `Specifications`, `AuthProxy`, `Prompter`, `AI`, `Samples`, `CLI`, `Documentation` | supporting libraries and tooling | — |

Two things about the state of it. **It is moving fast**: `Screenplay` carries commits dated
2026-08-07 through 2026-08-10 — the day before this was written — and the titles show the language
itself still changing shape (*"Take the running of an application back out of the language"*,
*"Let a read model stand on its own, and a reducer build it"*). And **the visual authoring tool is
closed**: Studio is named as the thing that authors and storyboards the model, and it is the one
piece not available to read.

## 2. Screenplay — the part that matters

Screenplay is a textual modeling language whose organizing unit is the slice. Its README states the
lineage plainly:

> A `.play` describes an entire system as a set of typed **slices**, aligned with Event Modeling's
> vocabulary.

The structure is `module` → `feature` → `slice`, where a module *"maps to a bounded context"* and a
feature is *"a vertical grouping of related slices"*. The slice is defined as *"the atomic unit of
behavior, aligned with Event Modeling"*, and there are four types:

| Slice type | Their gloss |
|---|---|
| `StateChange` | *"a command → events flow; something that changes the system"* |
| `StateView` | *"a query + projection + screen; something that reads the system"* |
| `Automation` | *"a reactor or reducer; something that reacts to events"* |
| `Translate` | *"a capture; converts external data into events"* |

Inside a slice sit `event`, `command`, `constraint`, `query`, `readmodel`, `projection`, `reducer`,
`screen`, `reactor`, `capture` — and, importantly, `specification`.

### 2.1 Given/When/Then is in the language, per slice

A `specification` block lives inside a slice, next to the command and events it exercises:

> Specifications express Given/When/Then test scenarios directly against a slice's own command and
> events — executable documentation for the behavior a slice implements.

```
specification RejectingAnInvoiceWhoseNumberIsAlreadyTaken
  given InvoiceRegistered
    invoiceNumber = "INV-000123"
  when RegisterInvoice
    invoiceNumber = "INV-000123"
  then error
```

Two details are worth noticing because this project has argued about both. `given readmodel` seeds
read-model state directly *"for scenarios where expressing the state as events would be noise"* —
an explicit escape from events-only `Given`s. And the rejection form is deliberately two-valued:
`then error "<message>"` means rejected for **this** reason; bare `then error` means rejected for a
reason the specification does not name, with the reason living in the specification's name. Their
justification for keeping the bare form rather than an empty string is a form argument of exactly
the kind this repo makes: *"An empty string reads as a reason someone left blank; the bare form says
one was never stated."*

### 2.2 `reads` — the consulted-view problem, solved in syntax

This is the single most useful thing in the survey. A command that must consult state declares it:

```
command StartMonth
  engagementId EngagementId identifier
  year         TimesheetYear
  month        TimesheetMonth
  reads EngagementScope by engagementId

  produces TimesheetStarted
    engagementId = engagementId
    consultantId = EngagementScope.consultantId
```

Their rationale, verbatim from `Documentation/screenplay/commands.md`:

> This is the read-model-to-command arrow of Event Modeling — one of the four the method is built
> on, and the only one a document could not draw. Without it, a command that decides against state
> shows its inputs and its events but not what it consulted in between, and the mapping fed from
> that state has nowhere to come from.

They also make an unread read model a warning: *"Reading something no projection produces is a
warning — the document says it depends on state nothing in it explains."*

Compare this repo's own position, reached independently and recorded in
[`paths/EXPLORE-view-slice.md`](paths/EXPLORE-view-slice.md) and
[`paths/EXPLORE-rewalk-cadence.md`](paths/EXPLORE-rewalk-cadence.md): *"A consulted view has **no**
top row — its readers declare it."* Same shape. Cratis made the reader's declaration a first-class
construct with a lookup key, and gets a completeness check out of it.

### 2.3 Compliance travels with the value

Concepts — wrapped primitive value types — carry `@pii` / `@sensitive` attributes once, and
*"every usage inherits them, so GDPR and sensitivity travel with the data instead of being
re-litigated per field."* Not a question this project has reached, and worth remembering when it
does: an attribute on the value type rather than on the field.

### 2.4 Pluggable sub-languages

Projections are written in a Projection Declaration Language and captures in a Change Data Capture
Language, described as *"embedded sub-grammars parsed inside their constructs"* and as reference
implementations of an extensible registry. The full EBNF grammar is published in the repository.

## 3. Stage — "the model *is* the application"

Stage takes the serialized event model and stands the application up at runtime:

> Hands an authored event model a stage and lets it perform — a live, running Cratis application at
> runtime. No code generation, no compilation: the model *is* the application.

> At startup the host loads the model, stands up its commands, queries, read models, and
> projections, and registers the projections with Chronicle so they run and populate the read-model
> store — a real event-sourced application, materialized from JSON.

And the pair of consumers is stated as a deliberate split: *"Hand the same model to **Studio** and
it storyboards it — visualizing and generating. Hand it to **Stage** and it performs it."*

**This is a real alternative to the premise `FnEmail-Model` is built on.** This project's five-repo
architecture (`DECISIONS.md` → *The repository architecture*) has a future `FnEmail` code repo
holding *"what the model generates"*. Stage says the generation step is optional — that an
interpreter over the model can be the deployable, and that the drift between model and code
disappears because there is only one artifact. Screenplay makes the same claim: *"Because the model
and the app are the same artifact, there's no drift to reconcile."*

Nothing about that is a reason to change course. It is a reason to know the position exists, and to
be able to say why we chose otherwise when the code repo is finally created — which is a rule 16
obligation the moment it becomes a stated position rather than an unexamined default.

## 4. Prologue — the arrow pointing the other way

Prologue stands beside a running system, captures SQL Server CDC, Postgres logical replication,
HTTP commands through a reverse proxy, and OpenTelemetry spans, correlates them by time window and
trace id, and interprets the result into an event model of *"modules, features, and slices with
their commands, events, read models, and projections."* It captures *"metadata, not data"* — which
tables and columns changed, never the values.

Not applicable to FnEmail, which has no legacy system. Recorded because it is evidence about what
the model-as-artifact is being asked to be: if a model can be *derived from* a running system and
*executed as* a running system, the model is being treated as the primary representation in both
directions. That is the same conviction this project holds, arrived at from the opposite end.

## 5. Mermaid has a native Event Modeling diagram type

Found while checking a claim in Cratis's own agent skills, and verified independently against the
Mermaid repository — this is not a Cratis fact, it is an upstream one:

- Mermaid ships an **`eventmodeling` diagram type from v11.15.0**, documented at
  `packages/mermaid/src/docs/syntax/eventmodeling.md`.
- Added by [PR #6440](https://github.com/mermaid-js/mermaid/pull/6440), *"feat: add Event Modeling
  diagram"*, authored by `lgazo` with `yordis`, opened 2025-03-30 and **merged 2026-04-07**.
  Mermaid is at 11.16.1 as of this writing.
- The documentation cites [eventmodeling.org](https://eventmodeling.org/) and the cheat sheet
  directly, and names the four patterns as *State View, State Change, Translation, Automation*.
- Entities are `ui`, `pcr` (processor), `cmd`, `rmo` (read model), `evt`, laid out in swimlanes on a
  timeline of numbered time frames (`tf 01 ui CartUI`), with relations inferred rather than drawn:
  *"Relations among the entities are inferred by default, so the distraction while you design is
  limited."*
- It supports **instance data**, inline (`tf 02 cmd AddItem { description: string }`) and in
  separate `data` blocks holding real values (`description: 'john'`, `price: 20.4`).
- The grammar is *"also maintained in a [separate project](https://github.com/lgazo/event-modeling-dsl)
  with the intention to provide also different types of output, VS Code (and potentially other)
  IDEs, etc."*

Cratis uses it: their `create-event-model` skill instructs agents to draw `EventModel.md` files in
*"Mermaid's native `eventmodeling` diagram type (v11.15+)"*, one per module, alongside the code.

### What this does and does not do to our Mermaid ruling

**It does not overturn it.** `DECISIONS.md` → *Superseded* replaces Mermaid with markdown tables as
of 2026-08-06, and `diagrams/README.md` → *Why tables* gives three reasons. The decisive one —
*"Mermaid needs a **diagram engine**; a table needs only a markdown renderer. The Android app has
the second and not the first, so the diagrams were invisible there"* — applies to the native
diagram type exactly as it applied to `block / columns 3`. AGENTS.md rule 5's **No Mermaid** stands
unchanged, and no ⚠️ correction is owed.

**One premise was narrower than the ruling's wording, and that is worth recording.** The rejection
was of hand-built `block` diagrams, and two of its three reasons are specific to them: that *"the
blocks were already tabular"* and that each carried *"five duplicated `classDef` lines that had to
stay in sync across nine diagrams"*. Neither is true of the native type, which is not a table in
disguise and has no `classDef`s. The native type had existed for four months when the ruling was
made and does not appear in its reasoning. The verdict survives on the renderer argument alone —
but the register row reads *"Mermaid diagrams and their label vocabulary"*, which is broader than
what was actually weighed.

**Where it is live is the tool, not this repo.** A modeling tool emits into renderers that do have a
diagram engine. An upstream, standardized, grammar-published EM notation that already carries
instance data is an output-format candidate that did not exist as a considered option when
`ModelingTool-Model` was created. It also partly fills the landscape gap `FOLLOW-UPS.md` records
under `event-modeling-research` — the unexplored `app.eventmodelers.ai` and the DSL question behind
defect D3.

## 6. Where Cratis corroborates us

Independent convergence is worth more than agreement, because neither party was reading the other.

**Information completeness.** Cratis runs the check at modeling time, in both directions:

> **Backward:** for each read model, walk every property back to the event that carries it. A field
> with no source event is a **missing event or command** — not a nullable column.

> **Forward:** every event you define should feed at least one read model, automation, or
> translation. An event nothing consumes is a smell — either a consumer is missing or the event
> shouldn't exist.

That is two of this project's three checks — *Completeness closes in three checks* (`DECISIONS.md`,
method commits 27627f9 here and 7ac81fb there): backward, payload, forward. **The payload check
appears to be ours.** Cratis has no equivalent, and the two defects this repo found by walking with
real data (AGENTS.md rule 8) are the argument for it. That makes the third check a genuine flow
candidate for the method repo rather than a restatement of common practice.

**No nullable fields on events.** Cratis: *"If you reach for a nullable property on an event, you
need a second event."* This repo reached the same place through the dataset cascade and the
constant-field rulings.

**Events never carry their own event source id.** Cratis makes it a compile error. Comparable in
spirit to the orphan-field discovery that started the completeness apparatus here.

**Reader-declares, for a consulted view.** Section 2.2 above.

## 7. Where Cratis diverges — recorded, not litigated

None of these is evidence that overturns a settled row, and none is offered as one (rules 4 and 9).
They are recorded because rule 16 asks a position to know whose it is, and because silence here
would later read as oversight.

| This project | Cratis | Reading |
|---|---|---|
| **Two slice types, not four patterns** — Automation and Translation are compositions (`DECISIONS.md`) | four typed slices, `StateChange` / `StateView` / `Automation` / `Translate`, as language primitives | Not a contradiction of the corpus reading. Our row is a claim about what is *primitive in the method*; theirs is a choice about what a *language declares*. Mermaid's docs, notably, call the same four **patterns** — our word — while listing exactly Cratis's four. Worth knowing that two independent implementations made the four the thing you type |
| **Workflow, never *chapter*** for a named group of slices (Dymitruk 241× to 10×) | `feature`, *"a vertical grouping of related slices"*, nesting arbitrarily deep | A third word, from neither author. Our row is about choosing between the corpus's two; it is untouched. The nesting-without-fixed-depth part matches our *"workflow nesting has no fixed depth"* row exactly |
| **Aggregates and DCB are out of scope** — a DDD import, not an Event Modeling question | Chronicle organizes events *"per aggregate or entity"*; **Screenplay has no `aggregate` construct** | This supports the row rather than straining it. Where Cratis models, there is no aggregate; the aggregate appears one layer down, in the store. That is precisely the position the row takes |
| **No arrows** — meaning is row position and left-to-right time | Mermaid infers relations by default and draws them; Cratis's diagrams carry explicit `->>` references by frame number for multi-source read models | The no-arrows caveat already concedes the trade: arrows do information-completeness tracing, and we do that check in prose. Cratis does it in the language (`reads`, and the unread-read-model warning) rather than in the picture |
| **`Given` is a labeled block; the minimal-`Given` departure** | `given <EventType>` and `given readmodel <ReadModelType>` | They allow seeding read-model state directly, *"for scenarios where expressing the state as events would be noise"*. Ours is a departure register entry waiting on a register (`FOLLOW-UPS.md`); theirs is an unremarked convenience |

## 8. Stance and attribution (rule 16)

Cratis's public documentation says *"aligned with Event Modeling's vocabulary"* and *"aligned with
Event Modeling"* without naming anyone. The attribution is real but it is one level down, in the
agent instructions at `.ai/skills/event-modeling/SKILL.md`:

> **Lineage.** Cratis's four slice types and the Given/When/Then-per-slice discipline follow
> **Event Modeling** (Adam Dymitruk; Martin Dilger, *Understanding Eventsourcing*); this skill
> applies that method to Cratis.

So: corpus **adopted**, attributed, and adapted to a platform. By this project's own standard the
attribution is in the wrong file — the reader of the language documentation meets the vocabulary
unlabeled, which is the failure rule 16 names in both directions. Recording that is not a criticism
worth sending anywhere; it is a note that we hold ourselves to something they do not, and that our
own tier vocabulary sits under the same hazard (rule 16's closing paragraph).

Their glossary also does something we should notice for the tool: *"The vocabulary of the Screenplay
language, defined once."* One page, one precise line per term, split from the shared platform
glossary. That is the shape of a thing this project does not have.

## 9. License and dependency (rule 7)

Every Cratis repository read here is **MIT**, `Copyright (c) 2026 Cratis`. Three consequences:

- **Quotation and citation are unproblematic.** MIT permits redistribution with the license notice;
  this document quotes briefly with attribution and links, which is well inside it, and matches
  the practice rule 7 sets for corpus material anyway.
- **No license conflict if code ever consumed it.** MIT is one-way compatible into AGPL-3.0, so a
  future `FnEmail` could depend on Cratis packages without a licensing problem.
- **The runtime cost is the real question, not the license.** Adopting Cratis means .NET, MongoDB,
  Orleans and gRPC underneath an SMTP server whose current scope is inbound `HELO` only. That is a
  large substrate for a small surface, and it is a decision for the code repo, which does not exist.

## 10. What I would do with this

Nothing that changes a document. Four things worth holding:

1. **Do not adopt Cratis as a dependency, and do not adopt Screenplay as our notation.** It is weeks
   old, changing daily, and coupled to Chronicle and Arc by design — its `.play` file is not a
   neutral interchange format for event models, it is the input to one platform. The premature-
   commitment risk is obvious and the standing instruction is to get the basics working first.
2. **Treat Screenplay as the benchmark for the modeling tool.** It is the closest public prior art
   to what `ModelingTool-Model` is for, and it is readable in full — grammar, documentation, and the
   commit trail of a language being designed in the open. Reading how they resolved a construct is
   cheaper than resolving it twice.
3. **Bring `reads` to the step 7 ruling as input.** `FOLLOW-UPS.md`'s *"What is step 7 for?"* is
   ruling-gated and open, and asks *"how a consulted view is read on a path"*. Cratis answers the
   same question by making the consult a declaration on the reader with a lookup key. It is not our
   ruling and it does not make one — but it is a worked answer with a stated rationale, and the
   ruling should not be made without having seen it.
4. **Watch Studio.** The one piece that is closed is the visual authoring tool — the direct
   counterpart to the tool this project is working toward. If it opens, it is worth a second look.

## 11. What was checked, and how

Sourcing, so the next reader can retrace rather than re-trust (rule 3):

- `Cratis/Screenplay` cloned at depth 50 and read from the working tree — `README.md`,
  `Documentation/screenplay/` (slices, commands, specifications, glossary, overview,
  why-screenplay), `.ai/skills/event-modeling/SKILL.md`, `.ai/skills/create-event-model/SKILL.md`,
  `AGENTS.md`, `LICENSE`. Every quotation above is from those files.
- `Stage`, `Prologue`, `Arc`, `Chronicle`, `Narrator`, `Fundamentals`, `AI` — raw `README.md` read
  from `raw.githubusercontent.com`, not through a summarizer.
- `Cratis/Studio` — no anonymous git read and no raw `README.md`; concluded not publicly readable,
  not concluded absent.
- Mermaid: `packages/mermaid/src/docs/syntax/eventmodeling.md` and `packages/mermaid/CHANGELOG.md`
  read raw; tags `mermaid@11.15.0` and `mermaid@11.16.1` confirmed by `git ls-remote`; PR #6440's
  authorship and merge date read from the PR page.
- **Not reached:** `cratis.io` and its documentation site, `mermaid.js.org`, and
  `ingebrigtsen.blog` — all blocked by the egress proxy. The Aksio origin of Cratis (the
  `Aksio.Cratis.*` NuGet package lineage) surfaced only in search results and is **not verified
  here**; it is stated nowhere above as fact.
- Two summarizer claims were checked against primary sources and one was wrong: a summary of
  `Cratis/Stage` reported that its documentation *"does not reference Event Modeling, Adam
  Dymitruk, or Martin Dilger"*. The Stage README indeed does not — but the platform's attribution
  exists, in Screenplay's agent skills, and a grep found it. Rule 3, earning its place again.
