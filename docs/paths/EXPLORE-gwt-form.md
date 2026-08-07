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
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com>&#10;<br>`250` OK |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

> **Pre** — `MailTransactionStarted` ✓ · `RecipientDirectory.is_local` ✓ &nbsp;·&nbsp; **Post** — `RecipientAccepted` emitted ✓

**That blockquote is what every variant below replaces.**

`RcptTo` was chosen over `MailFrom` because both of its failure cases are RFC-grounded rather than
invented, and one of them is a MUST:

| Scenario | Outcome | Source |
|:--|:--|:--|
| recipient is local | `250` OK → **RecipientAccepted** | D.1 |
| recipient is not local | `550` No such user here | §3.3 line 1072; the string is D.1's own |
| no `MAIL FROM` yet | `503` Bad sequence of commands | §4.1.1.3 — *"the server MUST return a 503"* |

Three scenarios, and they grow: each has one more event in its history than the last. That ordering
is deliberate — see *increasing complexity* in
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
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com>&#10;<br>`250` OK |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| **Given** | 🟧 **MailTransactionStarted** · 🟩 `RecipientDirectory.is_local`: true |
| **When** | 🟦 **RcptTo** \<Jones@foo.com> |
| **Then** | 🟧 **RecipientAccepted** |

*`When` restates row 2 and `Then` restates row 3. Only `Given` is new.*

## V2 · Three rows appended, chips instead of words

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com>&#10;<br>`250` OK |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| ⬅ **G** | 🟧 **MailTransactionStarted** · 🟩 `is_local`: true |
| ⚡ **W** | 🟦 **RcptTo** \<Jones@foo.com> |
| ➡ **T** | 🟧 **RecipientAccepted** |

## V3 · One row, stacked inside the cell

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com>&#10;<br>`250` OK |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Spec | **Given** 🟧 **MailTransactionStarted** · 🟩 `is_local`: true&#10;<br>**When** 🟦 **RcptTo** \<Jones@foo.com>&#10;<br>**Then** 🟧 **RecipientAccepted** |

## V4 · One row, inline

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com>&#10;<br>`250` OK |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Spec | 🟧 **MailTransactionStarted** → 🟦 **RcptTo** → 🟧 **RecipientAccepted** |

## V5 · Relabel — the spec *becomes* the rows

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com>&#10;<br>`250` OK |
| **Given** | 🟧 **MailTransactionStarted** · 🟩 `is_local`: true |
| **When** | 🟦 **RcptTo** \<Jones@foo.com> |
| **Then** | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

*No duplication at all. Costs the blank-middle-row convention, since the command row becomes `When`.*

---

# Family 2 — a separate table below the slice

## V6 · One table per scenario, named

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com>&#10;<br>`250` OK |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

| Recipient is local | ✓ |
|:--|:--|
| **Given** | 🟧 **MailTransactionStarted** · 🟩 `is_local`: true |
| **When** | 🟦 **RcptTo** \<Jones@foo.com> |
| **Then** | 🟧 **RecipientAccepted** |

| Recipient is not local | ✗ |
|:--|:--|
| **Given** | 🟧 **MailTransactionStarted** · 🟩 `is_local`: false |
| **When** | 🟦 **RcptTo** \<Green@foo.com> |
| **Then** | 🟥 `550` No such user here |

## V7 · Scenarios as columns — Dilger's board shape

Matches Fig. 13.6, where `Given`/`When`/`Then` are row labels and each scenario is a column.

| | Local | Not local | No transaction |
|:--|:--|:--|:--|
| **Given** | 🟧 **MailTransactionStarted**&#10;<br>🟩 `is_local`: true | 🟧 **MailTransactionStarted**&#10;<br>🟩 `is_local`: false | 🟧 **ClientIdentified** |
| **When** | 🟦 **RcptTo** \<Jones@foo.com> | 🟦 **RcptTo** \<Green@foo.com> | 🟦 **RcptTo** \<Jones@foo.com> |
| **Then** | 🟧 **RecipientAccepted** | 🟥 `550` No such user here | 🟥 `503` Bad sequence of commands |

*Scales sideways, which is the wrong axis for a phone. Three scenarios already need a scroll.*

## V8 · One row per scenario — scales down the page

| Scenario | Given | When | Then |
|:--|:--|:--|:--|
| Local | 🟧 **MailTransactionStarted** · 🟩 `is_local`: true | 🟦 **RcptTo** \<Jones@foo.com> | 🟧 **RecipientAccepted** |
| Not local | 🟧 **MailTransactionStarted** · 🟩 `is_local`: false | 🟦 **RcptTo** \<Green@foo.com> | 🟥 `550` No such user here |
| No transaction | 🟧 **ClientIdentified** | 🟦 **RcptTo** \<Jones@foo.com> | 🟥 `503` Bad sequence of commands |

*The corpus says 5–20 scenarios per command handler. This is the only form that survives twenty.*

---

# Family 3 — not a table

## V9 · Blockquote per scenario

> **Recipient is local** ✓
> **Given** 🟧 **MailTransactionStarted** · 🟩 `is_local`: true
> **When** 🟦 **RcptTo** \<Jones@foo.com>
> **Then** 🟧 **RecipientAccepted**

> **No transaction open** ✗
> **Given** 🟧 **ClientIdentified**
> **When** 🟦 **RcptTo** \<Jones@foo.com>
> **Then** 🟥 `503` Bad sequence of commands

## V10 · Nested list

- **Recipient is local** ✓
  - **Given** 🟧 **MailTransactionStarted** · 🟩 `is_local`: true
  - **When** 🟦 **RcptTo** \<Jones@foo.com>
  - **Then** 🟧 **RecipientAccepted**
- **Recipient is not local** ✗
  - **Given** 🟧 **MailTransactionStarted** · 🟩 `is_local`: false
  - **When** 🟦 **RcptTo** \<Green@foo.com>
  - **Then** 🟥 `550` No such user here

## V11 · Fenced block, Gherkin-style

```gherkin
Scenario: Recipient is local
  Given MailTransactionStarted(reverse_path: <Smith@bar.com>)
    And RecipientDirectory(is_local: true)
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
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com>&#10;<br>`250` OK |
| | 🟦 **RcptTo** |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

Specs collected at the end of the walk, keyed back:

| ① `RcptTo` | Given | When | Then |
|:--|:--|:--|:--|
| Local | 🟧 **MailTransactionStarted** · 🟩 `is_local`: true | 🟦 **RcptTo** \<Jones@foo.com> | 🟧 **RecipientAccepted** |
| Not local | 🟧 **MailTransactionStarted** · 🟩 `is_local`: false | 🟦 **RcptTo** \<Green@foo.com> | 🟥 `550` No such user here |

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

1. **Does a step get one scenario or many?** ⚠️ **Open, leaning few.** The inclination is that a step
   carries only what is at hand for *that* walk — its own data, examples and specifications — rather
   than the full set belonging to the handler. If that holds, the 5-to-20 figure never lands on a
   step at all and V1–V5 are not ruled out by size.
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
