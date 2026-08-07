# Exploring — how to write Given/When/Then in the markdown model

**Open. Nothing here is adopted.** Thirteen ways to attach a specification to a Command Slice, all
carrying the *same* content so the comparison is about form and never about the example.

Everything is pure markdown. Raw HTML tables are settled dead — they strip in the Claude Android app
and print as literal tags in the Claude Code desktop app. See
[`../diagrams/EXPERIMENT-vertical-align.md`](../diagrams/EXPERIMENT-vertical-align.md).

---

## The running example — `RcptTo`

The slice as it stands today in
[`helo-direct-single-recipient.md`](helo-direct-single-recipient.md):

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

> **Pre** — `MailTransactionStarted` ✓ · `RecipientDirectory.is_local` ✓ &nbsp;·&nbsp; **Post** — `RecipientAccepted` emitted ✓

**That blockquote is what every variant below replaces.**

`RcptTo` was chosen over `MailFrom` because both of its failure cases are RFC-grounded rather than
invented, and one of them is a MUST:

| Scenario | Outcome | Source |
|:--|:--|:--|
| recipient is local | **RecipientAccepted** | D.1 |
| recipient is not local | `550` No such user here | §3.3 line 1072; the string is D.1's own |
| no `MAIL FROM` yet | `503` Bad sequence of commands | §4.1.1.3 — *"the server MUST return a 503"* |

**The `Given` holds events only.** Whether a recipient is local is a *read model*, derived from
events rather than being one, so it never appears in a `Given`. The local and non-local scenarios
therefore share an identical history and are told apart by **the address in the `When`** —
\<Jones@foo.com> is local, \<Green@foo.com> is not. That is D.1's own pairing, and it puts the
distinguishing value in the command's argument where it belongs.

Ordering still matters: scenarios are written shortest-history-first, so each earns its rule from the
event that makes it necessary — *increasing complexity* in
`research/archive/open-spaces-comocamp/_WITNESS.md`.

⚠️ **An unresolved conflict, flagged before it gets embedded.** 🟥 already means **hotspot** in this
project. Dymitruk's models use red for an **error** outcome. Every variant below that shows a failure
uses 🟥 for it, which silently overloads the chip. Options are a different chip for errors, retiring
🟥 for hotspots, or accepting the overload because context disambiguates. **Not decided.**

---

# Family 1 — inside the slice table

## V1 · Three rows appended, word labels

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| **Given** | 🟧 **MailTransactionStarted** |
| **When** | 🟦 **RcptTo** \<Jones@foo.com> |
| **Then** | 🟧 **RecipientAccepted** |

*`When` restates row 2 and `Then` restates row 3. Only `Given` is new.*

## V2 · Three rows appended, chips instead of words

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| ⬅ **G** | 🟧 **MailTransactionStarted** |
| ⚡ **W** | 🟦 **RcptTo** \<Jones@foo.com> |
| ➡ **T** | 🟧 **RecipientAccepted** |

## V3 · One row, stacked inside the cell

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Spec | **Given** 🟧 **MailTransactionStarted**&#10;<br>**When** 🟦 **RcptTo** \<Jones@foo.com>&#10;<br>**Then** 🟧 **RecipientAccepted** |

## V4 · One row, inline

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Spec | 🟧 **MailTransactionStarted** → 🟦 **RcptTo** → 🟧 **RecipientAccepted** |

## V5 · Relabel — the spec *becomes* the rows

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| **Given** | 🟧 **MailTransactionStarted** |
| **When** | 🟦 **RcptTo** \<Jones@foo.com> |
| **Then** | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

*No duplication at all. Costs the blank-middle-row convention, since the command row becomes `When`.*

---

# Family 2 — a separate table below the slice

## V6 · One table per scenario, named

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

| Recipient is local | ✓ |
|:--|:--|
| **Given** | 🟧 **MailTransactionStarted** |
| **When** | 🟦 **RcptTo** \<Jones@foo.com> |
| **Then** | 🟧 **RecipientAccepted** |

| Recipient is not local | ✗ |
|:--|:--|
| **Given** | 🟧 **MailTransactionStarted** |
| **When** | 🟦 **RcptTo** \<Green@foo.com> |
| **Then** | 🟥 `550` No such user here |

## V7 · Scenarios as columns — Dilger's board shape

Matches Fig. 13.6, where `Given`/`When`/`Then` are row labels and each scenario is a column.

| | Local | Not local | No transaction |
|:--|:--|:--|:--|
| **Given** | 🟧 **MailTransactionStarted** | 🟧 **MailTransactionStarted** | 🟧 **ClientIdentified** |
| **When** | 🟦 **RcptTo** \<Jones@foo.com> | 🟦 **RcptTo** \<Green@foo.com> | 🟦 **RcptTo** \<Jones@foo.com> |
| **Then** | 🟧 **RecipientAccepted** | 🟥 `550` No such user here | 🟥 `503` Bad sequence of commands |

*Scales sideways, which is the wrong axis for a phone. Three scenarios already need a scroll.*

## V8 · One row per scenario — scales down the page

| Scenario | Given | When | Then |
|:--|:--|:--|:--|
| Local | 🟧 **MailTransactionStarted** | 🟦 **RcptTo** \<Jones@foo.com> | 🟧 **RecipientAccepted** |
| Not local | 🟧 **MailTransactionStarted** | 🟦 **RcptTo** \<Green@foo.com> | 🟥 `550` No such user here |
| No transaction | 🟧 **ClientIdentified** | 🟦 **RcptTo** \<Jones@foo.com> | 🟥 `503` Bad sequence of commands |

*The corpus says 5–20 scenarios per command handler. This is the only form that survives twenty.*

---

# Family 3 — not a table

## V9 · Blockquote per scenario

> **Recipient is local** ✓
> **Given** 🟧 **MailTransactionStarted**
> **When** 🟦 **RcptTo** \<Jones@foo.com>
> **Then** 🟧 **RecipientAccepted**

> **No transaction open** ✗
> **Given** 🟧 **ClientIdentified**
> **When** 🟦 **RcptTo** \<Jones@foo.com>
> **Then** 🟥 `503` Bad sequence of commands

## V10 · Nested list

- **Recipient is local** ✓
  - **Given** 🟧 **MailTransactionStarted**
  - **When** 🟦 **RcptTo** \<Jones@foo.com>
  - **Then** 🟧 **RecipientAccepted**
- **Recipient is not local** ✗
  - **Given** 🟧 **MailTransactionStarted**
  - **When** 🟦 **RcptTo** \<Green@foo.com>
  - **Then** 🟥 `550` No such user here

## V11 · Fenced block, Gherkin-style

```gherkin
Scenario: Recipient is local
  Given MailTransactionStarted(reverse_path: <Smith@bar.com>)
   When RcptTo(<Jones@foo.com>)
   Then RecipientAccepted(forward_path: <Jones@foo.com>)

Scenario: No transaction open
  Given ClientIdentified(claimed_domain: bar.com)
   When RcptTo(<Jones@foo.com>)
   Then Error(503, "Bad sequence of commands")
```

⚠️ **Costs every convention this project settled.** No chips, no monospace/standard distinction, and
values are back inside quotes. It gains machine-readability and copy-paste into a test generator —
which is what the corpus says GWT is *for*. Included because that trade is real, not to endorse it.

---

# Family 4 — what Dymitruk actually does

## V12 · Timeline of checkpoints

His `specs.json` shape: an ordered run of events, each paired with the state expected after it. **The
`Given` is never written — it accumulates.** Every row's given is every row above it.

| Time Line — Happy Path | state after |
|:--|:--|
| 🟧 **ClientIdentified** `claimed_domain`: bar.com | 🟩 `identified`: true |
| 🟧 **MailTransactionStarted** `reverse_path`: \<Smith@bar.com> | 🟩 `open`: true · `recipient_count`: 0 |
| 🟦 **RcptTo** \<Jones@foo.com> | |
| 🟧 **RecipientAccepted** `forward_path`: \<Jones@foo.com> | 🟩 `recipient_count`: 1 |

## V13 · Increasing complexity, showing only the delta

Each scenario is the one above it **plus one event**. Only the new row is written; `↑` means
everything above.

| Time Line | added to history | 🟦 command | outcome |
|:--|:--|:--|:--|
| **Happy path** | 🟧 **MailTransactionStarted** | 🟦 **RcptTo** \<Jones@foo.com> | 🟧 **RecipientAccepted** |
| **Second recipient** | ↑ + 🟧 **RecipientAccepted** | 🟦 **RcptTo** \<Green@foo.com> | 🟧 **RecipientAccepted** |
| **Not local** | ↑ | 🟦 **RcptTo** \<Nobody@foo.com> | 🟥 `550` No such user here |
| **After `RSET`** | ↑ + 🟧 **TransactionReset** | 🟦 **RcptTo** \<Jones@foo.com> | 🟥 `503` Bad sequence of commands |

*This is the form the witness account describes and the only one that shows a rule being **earned** by
the event that makes it necessary.*

## V14 · Numbered legend — the slice points at its spec

Dymitruk on a whiteboard: *"put a big number one here and then number one down below"*.

| 🟦 C · Step 6 · ① | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

Specs collected at the end of the walk, keyed back:

| ① `RcptTo` | Given | When | Then |
|:--|:--|:--|:--|
| Local | 🟧 **MailTransactionStarted** | 🟦 **RcptTo** \<Jones@foo.com> | 🟧 **RecipientAccepted** |
| Not local | 🟧 **MailTransactionStarted** | 🟦 **RcptTo** \<Green@foo.com> | 🟥 `550` No such user here |

*Keeps the walk readable at the cost of making the reader jump. The corpus uses this for a 2D board
where the spec is visible below; in a scrolling document the jump is much longer.*

---

# Comparison

| | Duplication | Scales to 20 | Keeps conventions | Phone | Corpus support |
|:--|:--|:--|:--|:--|:--|
| **V1** three rows | 2 of 3 rows | ✗ | ✓ | ✓ | Dilger's labels |
| **V2** chips | 2 of 3 | ✗ | ✓ | ✓ | none — invented |
| **V3** stacked cell | 2 of 3 | ✗ | ✓ | ✓ | none |
| **V4** inline | 2 of 3 | ✗ | ✓ | ✓ | none |
| **V5** relabel | **none** | ✗ | loses blank row | ✓ | Dilger's labels |
| **V6** table each | some | poorly | ✓ | ✓ | Fig. 13.5 |
| **V7** columns | none | **sideways ✗** | ✓ | ✗ scrolls | **Fig. 13.6** |
| **V8** row each | none | **✓ best** | ✓ | ✓ | none directly |
| **V9** blockquote | none | ✓ | ✓ | ✓ | none |
| **V10** list | none | ✓ | ✓ | ✓ | none |
| **V11** Gherkin | none | ✓ | **✗ loses all** | ✓ | *"testable, e.g. Gherkin"* |
| **V12** timeline | none | ✓ | ✓ | ✓ | **his `specs.json`** |
| **V13** delta | none | **✓** | ✓ | ✓ | **witness account** |
| **V14** legend | none | ✓ | ✓ | reader jumps | *"a big number one"* |

**Three questions the choice actually turns on**, none of which is about looks. Answered
2026-08-07 — two settled, one open.

1. **Does a step get one scenario or many?** ✅ **Only its own walk's scenarios.** A step carries what
   is at hand for *that* traversal and nothing else. **The 5-to-20 figure never lands on a step**, so
   the size objection that ruled out V1–V5 does not apply and the whole field is back open. It lands
   instead on the *slice*, where the per-step sets accumulate — and a slice may well want a different
   form from a step, since V8 and V13 are the only ones that survive twenty.
2. **Does the spec belong to the step or to the slice?** ✅ **Both, in one direction.** The step
   carries its local set; those sets are then combined and deduplicated across every walk, and that
   union *is* the slice. So a specification is authored at the step and **accumulates** at the slice.
   Nothing is duplicated, because the slice's set is derived rather than copied — see
   `../HANDOFF.md` §1, *Paths are the source; slices are derived*.
3. **Is the walk itself already a timeline?** ✅ **Yes.** Which weakens V12: a per-step timeline
   would restate inside one step the ordering the ten steps already express. V13 survives the
   objection because it only ever writes the **delta**, never the accumulated history.

⚠️ **A distinction not to lose here.** A walk being a timeline does *not* make it a workflow. They
are held as different concepts — a walk is one traversal with real values, a workflow is a named
region of the model, and one walk crosses several. Untested, and flagged rather than assumed.

---

## What this is really deciding

**The markdown is a prototype of generated output.** The long-run intent is that these documents are
emitted by a modeling tool rather than written by hand, so the question is not *which form is
pleasant to type* but **which form is right to read** — the typing cost disappears when a generator
does it. A form that is tedious by hand and correct on the page beats a form that is quick to write
and ambiguous. See `../HANDOFF.md` §1.

---

## Why multiple GWTs rather than one timeline

Asked 2026-08-07: *our walk, narrowed to just what it deals with, looks like the expanded version of
one of Adam's GWTs — his GWT stack is just condensing an exhaustive timeline.*

**The structural half of that is exactly right.** A step in a walk *is* a GWT. Its `Given` is every
event above it, its `When` is the step's command, its `Then` is the step's event. So a ten-step walk
is ten chained GWTs where each `Given` is the previous `Then` accumulated. Nothing is being added by
writing it out; the walk already has that shape.

**Where the two forms come apart is what each one varies.**

| | varies | holds fixed |
|:--|:--|:--|
| **Timeline / walk** | the **command** — it moves forward | the history, which is simply whatever accumulated |
| **GWT set** | the **history** — the `Given` is the variable | the command under test |

That is why the reduction only runs one way. **A timeline always decomposes into GWTs**, one per
step. **A GWT set only recomposes into a timeline when the scenarios happen to nest.** Adam's four
blocks nest — each adds one event — which is precisely why they read as slices of one exhaustive
history. That is a property of *that set*, not of GWTs.

It breaks the moment a scenario needs a **shorter or mutually exclusive** history. `RcptTo` with no
`MAIL FROM` is not a later point on the happy timeline; it is a *different, shorter* one. To force it
onto a single line you would have to build a contrived history that visits every state, and then
every scenario's `Given` drags in irrelevant events and — the real cost — **the scenarios become
coupled**. Change one early event and every later scenario shifts. GWTs are independent by
construction, and that independence is what makes them testable in isolation.

### The corpus says where variation is allowed to go

Dymitruk routes it to exactly two places, and **neither is the timeline**. Copenhagen DDD,
2020-03-29 @27347, machine transcript:

> *"that's one of the reasons that that we don't do branching specifically an alternate major
> workflow would be basically added on the end of this thing … if we want to just have really
> inconsequential derivations or maybe the the difference in different types of data how it's stored
> maybe how the calculation is made we can elaborate that on anywhere from 5 to 20 different given
> when thenns for a particular command handler … so that's why branching is is not shown"*

| Kind of variation | Where it goes |
|:--|:--|
| a major alternative | **a new workflow, added on the end** — Dilger's *"Linearize it!"* |
| a derivation, a data difference, a different calculation | **5–20 GWTs on the command handler** |
| anything else | nowhere — *"branching is not shown"* |

**So the timeline is deliberately branch-free, and the GWTs are where the branches were sent.** The
walk shows the one path taken; the GWTs show what the path did *not* take. They are complements, not
compressions of one another.

### ⚠️ Which lands on one real question for this project

Within a single walk, a step's `Given` is **every event above it** — already on the page. So an
explicit per-step GWT restates the walk and adds nothing, *unless* the `Given` is the **minimal
history that makes the rule fire** rather than the accumulated one. Those are different claims:

- *Everything above* — Dymitruk's stated convention, so a runner *"can always just use an accumulator
  to add events"* (episode 8). Mechanical, and **zero new information at a step**.
- *Minimal sufficient* — what a test would assert. Genuinely new information: it says which prior
  events the rule actually depends on.

`RcptTo` in this walk has three events above it and needs one. **That gap is the only thing a
per-step GWT can contribute**, and the corpus points at the accumulator. Unresolved, and it decides
whether the step-level GWT is worth writing at all.
