# Diagrams — FnEmail inbound event model

Renderings of `../event-model.md` v0.3, as **markdown tables**. They render on GitHub, on desktop,
and in the Android app. Sections 4–6 were reconciled 2026-08-08 to the walked timeline of
[`WORKING-helo-direct-single-recipient-v2.md`](../paths/WORKING-helo-direct-single-recipient-v2.md),
on the owner's waiver of the rule-13 hold. The 2026-08-09 rulings are reflected throughout: the
acknowledgment views named (`ReversePathAllowed`, `RecipientConfirmed`), the event renamed
`DataRequested` (H1 closed), the dataset cascade closed, and the consulted box at
`RecipientDirectory` dissolved; §§5–6 carry the emptied views and the dissolution.

**Colors** — 🟧 Event · 🟦 Command · 🟩 Read Model · ⬜ rendered UI (Screen) · ⬛ wire ·
🟨 external / required-first · 🟥 hotspot · 🟤 nothing, only ever in a `Given`.

**The top row is the Actor row, not the Screen row.** *Screen* names an **element**; *Actor* names
the **lane**, and the lane is what a row is. The corpus supports the distinction — Dymitruk's step
3.1 calls them *"swim-lanes to show different people (or sometimes systems)"* and adds *"we also
show any **automation** here with a symbol like gears."* A lane holding both screens and automations
cannot be named after one of them. The ⬜ chip still marks a Screen where one sits in the lane.

The chip carries the element type where a row is mixed; elsewhere the row label carries it. A chip
is not decoration — in Event Modeling **color is the element type**, and Command-blue and
ReadModel-green are invariant across every source.

**Terminology.** *Slice*, not column — Adam's own word. The two slice types are named here as this
project prefers them, with the corpus synonyms in parentheses:

| This project | Corpus synonyms |
|---|---|
| **Command Slice** | state change · write column · `state-change` |
| **View Slice** | state view · read model · query · `state-view` |

**Rendering vocabulary.** Settled empirically — see *Why tables* below. Columns are slices, rows
are element types, `—` marks a row a slice does not use. Cell line breaks are written `&#10;<br>`;
both halves are required, for reasons recorded below.

**Slices are not numbered here.** Position on the timeline already carries order, so a number adds
nothing but fragility — slice numbering is an artifact of the current altitude and scope, and would
renumber wholesale if either moved. Slices are referred to **by name**; counts like "twelve slices"
are still fine, because a count is a fact about the model rather than a handle on one part of it.

**No arrows, and no branching.** Meaning is carried by row position and left-to-right time; a
timeline holds no branch points. Rows are fixed:

```
row 1   Actor      screen, or automation for a processor
row 2   Command (blue)  ·or·  Read Model (green)   ← never both; that is the slice type
row 3   Event (orange)
```

A Command Slice reads **top to bottom**. A View Slice reads **bottom to top**. Same three rows.

> ⚠️ Arrows are notation, not semantics, and this project does not use them — but that forgoes
> something. Their job in the corpus is **information-completeness tracing**: following one field
> from where it is captured to where it is used. This project does that check in prose instead. See
> `../DECISIONS.md`, the no-arrows caveat.

---

## Why tables, and why the line breaks look like that

*(The rendering conventions are now stated canonically in the method repo's `rendering.md`; this
section and the two-renderer matrix below are the 2026-08-06 record that produced them. The
three-renderer verdict lives in [`EXPERIMENT-line-break.md`](EXPERIMENT-line-break.md).)*

Replaced Mermaid on 2026-08-06. Mermaid needs a **diagram engine**; a table needs only a markdown
renderer. The Android app has the second and not the first, so the diagrams were invisible there —
which for a model meant to be read anywhere is a real cost.

The blocks were already tabular. `block / columns 3` with one row per element type **is** a table;
Mermaid was a rendering layer over a structure that did not need one. Each block also carried five
duplicated `classDef` lines that had to stay in sync across nine diagrams — the exact drift that
produced the upstream color defect this project documents.

**Tested empirically, in the Android app and against GitHub's own renderer:**

| feature | GitHub | Android |
|:--|:--|:--|
| bordered tables, bold, left align, empty cells | ✅ | ✅ |
| code spans, incl. `<angle brackets>` | ✅ | ✅ |
| emoji color chips 🟧🟦🟩⬜🟨🟥 | ✅ | ✅ |
| long cells wrap | ✅ | ✅ |
| `<br>` and `<br/>` | ✅ | ❌ **stripped silently** |
| `&#10;` | ❌ collapses to whitespace | ✅ |
| **`&#10;<br>`** | ✅ | ✅ |

Each renderer ignores exactly what the other one uses, so **both halves are required**. Write
`&#10;<br>`, never one alone. A lone `<br>` silently joins two lines into one on a phone —
`ConnectionAcceptedpeer_address` — with no error to notice.

Tables **scroll sideways** on a phone at three or more columns. That is accepted: real work happens
one or two slices at a time, and the wide timelines are zoomed-out overviews.

---

## 1. Command Slice

*(aka state change, write column)*

The `MailFrom` slice.

| | **MailFrom** |
|:--|:--|
| ⬜ **Actor** | **MTA Client**&#10;<br>`MAIL FROM:`\<Smith@bar.com> |
| 🟦 **Command** | **MailFrom**&#10;<br>`reverse_path` |
| 🟧 **Event** | **ReversePathDeclared**&#10;<br>`reverse_path` |

### One event per command

**The target is exactly one.** More than one is legal, but it usually signals that several
responsibilities have been folded into a single command, and it should be investigated rather than
accepted.

This is a stronger rule than the corpus states. Dilger names the shape as an anti-pattern —
*"left chair"*, one command to many events — but calls the four anti-patterns *"not red flags, but
something to keep an eye on."* Here it is a design target with a stated reason: **multiple events
usually means multiple responsibility.**

It already earned its keep. In v0.1 `SubmitContent` emitted three event types plus a per-recipient
fan-out. Scoping to inbound removed the fan-out and the completeness check removed a redundant
event, leaving one command, one event — and the model got simpler, not poorer.

| | ✅ **one command → one event** | ⚠️ **one command → many events**&#10;<br>*"left chair"* |
|:--|:--|:--|
| 🟦 **Command** | **Command** | **Command** |
| 🟧 **Event** | **Event** | **Event** · **Event** · **Event** |

**`RcptTo` does not trip this rule.** `RecipientAccepted` and `RecipientRejected` are
*alternatives* — accepted **or** rejected, never both — not a fan-out.

---

## 2. View Slice

*(aka state view, read model, query)*

The `SessionReady` slice — the view that renders the `250` foo.com reply to `HELO` (ruled
2026-08-08, commit cf3227d). Same three rows, read **bottom to top**.

| | **SessionReady** |
|:--|:--|
| ⬜ **Actor** | **MTA Client**&#10;<br>⬛ `250` foo.com |
| 🟩 **Read Model** | **SessionReady**&#10;<br>`server_domain` |
| 🟧 **Events** | **ServiceConfigured**&#10;<br>**ClientIdentified** |

A View Slice may draw on events from **anywhere earlier** on the timeline. It is placed at its
**consumer**, not next to its sources — which is why `SessionReady` sits at the reply it renders
while its sources lie further left: `ClientIdentified` is the occasion, and `ServiceConfigured`,
seeded before the timeline, carries the data.

Many events feeding one read model is the *"right chair"* shape. Expected for a view that
accumulates; worth watching if it grows without a matching growth in the question it answers.

---

## 3. Both slice types, side by side

The shared row structure, and why the two never mix. Row 2 holds a command **or** a read model —
that choice *is* the slice type.

| | **COMMAND SLICE**&#10;<br>read top → bottom | **VIEW SLICE**&#10;<br>read bottom → top |
|:--|:--|:--|
| ⬜ **Actor** | Actor / Processor | Actor / Processor |
| **Row 2** | 🟦 **Command** | 🟩 **Read Model** |
| 🟧 **Row 3** | **Event** | **Event(s)** |

Every slice in the model is one of these two. Automation and Translation are **compositions** — a
View Slice feeding a Command Slice — which is why Dymitruk specifies an automation as *two*
Given-When-Thens rather than one.

---

## 4. Inbound timeline — session establishment

`AcceptConnection` → `SessionReady`. Columns are slices; rows are element types; `—` marks a row a
slice does not use. Every server reply has a view step behind it, per the walked timeline.

| | **AcceptConnection** | **ServiceReady** | **Helo** | **SessionReady** |
|:--|:--|:--|:--|:--|
| ⬜ **Actor** | tcp connect | ⬛ `220` foo.com Simple Mail Transfer Service Ready | `HELO` bar.com | ⬛ `250` foo.com |
| **Cmd / View** | 🟦 **AcceptConnection** | 🟩 **ServiceReady**&#10;<br>`server_domain`&#10;<br>`greeting_text` | 🟦 **Helo** | 🟩 **SessionReady**&#10;<br>`server_domain` |
| 🟧 **Event** | **ConnectionAccepted**&#10;<br>`peer_address`&#10;<br>`local_address` 🟥 H6 | — | **ClientIdentified**&#10;<br>`claimed_domain` | — |

Both views fold in the seeded 🟨 `ServiceConfigured`, the path-level `Given` that carries
`server_domain` and `greeting_text` from before the timeline. `local_address` is 🟥 **H6** — two
candidate consumers on the walk, the multi-homed `Received:` BY arm and the nameless-server `220`
greeting, and the walk exercises neither; no arm is picked here.

---

## 5. Inbound timeline — the transaction

`MailFrom` → `RcptTo`. The Translation boundary onto the Directory context (H3 resolved) enters
as the seeded, 🟨 **yellow** `RecipientResolved` — a source outside this model, folded directly
by `RcptTo`'s handler.

| | **MailFrom** | **ReversePathAllowed** | **RcptTo** | **RecipientConfirmed** |
|:--|:--|:--|:--|:--|
| ⬜ **Actor** | `MAIL FROM` | ⬛ `250` OK | `RCPT TO` | ⬛ `250` OK |
| **Cmd / View** | 🟦 **MailFrom** | 🟩 **ReversePathAllowed** | 🟦 **RcptTo** | 🟩 **RecipientConfirmed** |
| **Event** | 🟧 **ReversePathDeclared**&#10;<br>`reverse_path` | — | 🟨 **RecipientResolved**&#10;<br>translated — external, folded by the handler&#10;<br>🟧 **RecipientAccepted**&#10;<br>🟧 **RecipientRejected** `550` | — |

The 🟨 **yellow** cell is what distinguishes Translation from Automation — the source event belongs
to another context.

`ReversePathAllowed` renders its `250` OK from event existence alone: the dataset was emptied on
the walked path (commit cd06274), and the 2026-08-09 naming — a view names the server's response
to its occasioning event — split what was one view traversed twice. The walk's second
acknowledgment, after `RcptTo`, is `RecipientConfirmed` (named later the same day): it confirms
the adjudication `RecipientAccepted` already made, and renders its `250` OK from that event's
existence ([`EXPLORE-declaration-vs-status.md`](../paths/EXPLORE-declaration-vs-status.md)).

The consulted `RecipientDirectory` box that stood between `ReversePathAllowed` and `RcptTo`
**dissolved 2026-08-09**: no `Given` can cite a view, the box performed no fold, and the
three-fates discipline gives a handler's fold no box. The translation boundary survives in the
seeded event above; in the expanded model the consultation returns as a processing slice.

---

## 6. Inbound timeline — content and close

`BeginData` → `SessionClosing`. `DataRequested`'s 🟥 chip is cleared: H1 closed 2026-08-09 —
the earn-its-place half on the v2 walk's two consumers, the name half by the two-kinds lens
(capture of the client's request, replacing the status-register `DataPhaseEntered`;
[`EXPLORE-declaration-vs-status.md`](../paths/EXPLORE-declaration-vs-status.md)).

| | **BeginData** | **DataPrompt** | **SubmitContent** | **MessageTrace** | **MessageQueued** | **Quit** | **SessionClosing** |
|:--|:--|:--|:--|:--|:--|:--|:--|
| ⬜ **Actor** | `DATA` | ⬛ `354` Start mail input; end with `<CRLF>.<CRLF>` | content · then dot | **Stored message**&#10;<br>the `Received:` header | ⬛ `250` OK | `QUIT` | ⬛ `221` foo.com Service closing transmission channel |
| **Cmd / View** | 🟦 **BeginData** | 🟩 **DataPrompt**&#10;<br>*(existence-fold, no dataset)* | 🟦 **SubmitContent** | 🟩 **MessageTrace** 🟥 H6&#10;<br>`from_domain` · `address_literal` · `by` · `id` · `for` · `at` | 🟩 **MessageQueued**&#10;<br>*(existence-fold, no dataset)* | 🟦 **Quit** | 🟩 **SessionClosing** 🟥 H8&#10;<br>`server_domain` |
| **Event** | 🟧 **DataRequested** | — | 🟧 **MessageAccepted**&#10;<br>`queue_id` · `reverse_path` · `recipients` · `content_ref` · `actual_octets` · `received_at` | — | — | 🟧 **SessionClosed**&#10;<br>`cause` 🟥 H8 | — |

**The dataset cascade closed 2026-08-09** — ruled: "Don't invent a new test — the view's own
definition already decides it." `awaiting_content`, `accepted` and the `queue_id` copy left
their views, and `cause` left `SessionClosing` — surviving on `SessionClosed`, where 🟥 H8
flags it unconsumed until a closure-branch walk consumes it. The grounds are in the walk's
steps 11, 14 and 16 notes.

**🟥 H6 bites at `MessageTrace`**: the `by` clause renders from `ServiceConfigured.server_domain`
on the config arm, and would render from `ConnectionAccepted.local_address` on the multi-homed
arm — two candidate consumers, neither exercised by the walk, neither picked here.

`Reset` (`RSET` → `TransactionAborted`) and `SessionTranscript` are model slices this walked
timeline does not touch: no walked path exercises `Reset` — the aborted-transaction scenario,
D.2, would — and the v2 walk renders every reply from a named view rather than through
`SessionTranscript`.

---

## 7. Worked paths

Three paths are walked with real data in [`../paths/`](../paths/), and the direct one is being
re-walked under the settled semantics as
[`WORKING-helo-direct-single-recipient-v2.md`](../paths/WORKING-helo-direct-single-recipient-v2.md).
They are **not redrawn here as sequence diagrams**, for two reasons.

**The top row already is the conversation.** Read the ⬜ Actor row of §4 → §5 → §6 left to right
and you have the exchange: `tcp connect`, `HELO`, `MAIL FROM`, `RCPT TO`, `DATA`, content, `QUIT`.
A separate sequence diagram restates what the timeline's first row carries, and restating it means
two artifacts that can disagree.

**And a sequence diagram is the wrong notation to reach for here.** The Dymitruk quotations in
this section are machine transcripts. He describes an event model as one — *"it kind of looks
like a **sideways sequence diagram**, but it cuts out the how"* (`14KWuOH9nSk`), *"sort of like a
sequence diagram but turned sideways"* (`8Uz4370F_KQ`) — but he is pointed about why the original
will not do:

> *"it will not do branching. **Sequence diagrams have branching.** … the problem is the human mind
> can't remember a graph"* — `14KWuOH9nSk`

> *"sequence diagrams start to talk about your **implementation abstractions** which is very useless
> for someone that's looking at the top level from the business perspective"* — `technologist8`

This document's own header says *no branching*. Importing the notation he rejects, for the
properties he rejects it for, was inconsistent — so the sequence diagrams that stood here are gone
rather than converted.

| Path | Shape |
|:--|:--|
| [`helo-multi-recipient.md`](../paths/helo-multi-recipient.md) | Three recipients, one rejected mid-flow with 🟥 `550`. Complete success for the server, **partial** for the sender, failure for `Green` — outcome is relative to the actor. |
| [`helo-single-recipient.md`](../paths/helo-single-recipient.md) | One recipient, no errors, relayed — the client is a relay, not the origin. Succeeds for everyone. |
| [`helo-direct-single-recipient.md`](../paths/helo-direct-single-recipient.md) | One recipient, no errors, direct — client and sender domains align. The default illustration, being re-walked as v2. |

`Reset` is the only slice no path has touched — the gap the aborted-transaction scenario, D.2,
would close.
