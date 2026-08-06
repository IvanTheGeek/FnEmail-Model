# Diagrams — FnEmail inbound event model

Renderings of `../event-model.md` v0.3, as **markdown tables**. They render on GitHub, on desktop,
and in the Android app.

**Colors** — 🟧 Event · 🟦 Command · 🟩 Read Model · ⬜ Screen · 🟨 external · 🟥 hotspot.

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

**No arrows.** Meaning is carried by row position and left-to-right time. Rows are fixed:

```
row 1   Actor / Screen
row 2   Command (blue)  ·or·  Read Model (green)   ← never both; that is the slice type
row 3   Event (orange)
```

A Command Slice reads **top to bottom**. A View Slice reads **bottom to top**. Same three rows.

> ⚠️ Arrows are notation, not semantics, and this project does not use them — but that forgoes
> something. Their job in the corpus is **information-completeness tracing**: following one field
> from where it is captured to where it is used. This project does that check in prose instead. See
> `../HANDOFF.md` §4.

---

## Why tables, and why the line breaks look like that

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

Slice 4, `MailFrom`.

| | **4 · MailFrom** |
|:--|:--|
| ⬜ **Screen** | **Remote client**&#10;<br>`MAIL FROM:<Smith@bar.com>` |
| 🟦 **Command** | **MailFrom**&#10;<br>`reverse_path` |
| 🟧 **Event** | **MailTransactionStarted**&#10;<br>`reverse_path` |

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

Slice 3, `SessionState`. Same three rows, read **bottom to top**.

| | **3 · SessionState** |
|:--|:--|
| ⬜ **Consumed by** | **MailFrom · RcptTo · BeginData** |
| 🟩 **Read Model** | **SessionState**&#10;<br>`identified`&#10;<br>`transaction_open` |
| 🟧 **Events** | **ConnectionAccepted**&#10;<br>**ClientIdentified**&#10;<br>**SessionReset** |

A View Slice may draw on events from **anywhere earlier** on the timeline. It is placed at its
**consumer**, not next to its sources — which is why `SessionState` sits at slice 3 while its
sources are slices 1 and 2.

Many events feeding one read model is the *"right chair"* shape. Expected for a view that
accumulates; worth watching if it grows without a matching growth in the question it answers.

---

## 3. Both slice types, side by side

The shared row structure, and why the two never mix. Row 2 holds a command **or** a read model —
that choice *is* the slice type.

| | **COMMAND SLICE**&#10;<br>read top → bottom | **VIEW SLICE**&#10;<br>read bottom → top |
|:--|:--|:--|
| ⬜ **Row 1** | Actor / Processor | Actor / Processor |
| **Row 2** | 🟦 **Command** | 🟩 **Read Model** |
| 🟧 **Row 3** | **Event** | **Event(s)** |

Every slice in the model is one of these two. Automation and Translation are **compositions** — a
View Slice feeding a Command Slice — which is why Dymitruk specifies an automation as *two*
Given-When-Thens rather than one.

---

## 4. Inbound timeline — session establishment

Slices 1–3. Columns are slices; rows are element types; `—` marks a row a slice does not use.

| | **1 · AcceptConnection** | **2 · Helo** | **3 · SessionState** |
|:--|:--|:--|:--|
| ⬜ **Screen** | tcp connect | `HELO bar.com` | **consumed by**&#10;<br>slices 4, 6, 8 |
| **Cmd / View** | 🟦 **AcceptConnection** | 🟦 **Helo** | 🟩 **SessionState**&#10;<br>`identified` |
| 🟧 **Event** | **ConnectionAccepted**&#10;<br>`peer_address` | **ClientIdentified**&#10;<br>`claimed_domain` | — |

---

## 5. Inbound timeline — the transaction

Slices 4–7. `RecipientDirectory` is the Translation boundary onto the Directory context (H3
resolved), so its source is outside this model — shown 🟨 **yellow**.

| | **4 · MailFrom** | **5 · RecipientDirectory** | **6 · RcptTo** | **7 · TransactionState** |
|:--|:--|:--|:--|:--|
| ⬜ **Screen** | `MAIL FROM` | **consumed by**&#10;<br>slice 6 | `RCPT TO` | **consumed by**&#10;<br>slices 6, 8, 9 |
| **Cmd / View** | 🟦 **MailFrom** | 🟩 **RecipientDirectory**&#10;<br>`is_local` | 🟦 **RcptTo** | 🟩 **TransactionState**&#10;<br>`recipient_count` |
| **Event** | 🟧 **MailTransactionStarted**&#10;<br>`reverse_path` | 🟨 **Directory context**&#10;<br>translated — external | 🟧 **RecipientAccepted**&#10;<br>🟧 **RecipientRejected** `550` | — |

The 🟨 **yellow** cell is what distinguishes Translation from Automation — the source events belong
to another context.

---

## 6. Inbound timeline — content and close

Slices 8–12. `DataPhaseEntered` is 🟥 **red**: hotspot **H1**, on trial because its only candidate
consumer is the transcript.

| | **8 · BeginData** | **9 · SubmitContent** | **10 · Reset** | **11 · Quit** | **12 · SessionTranscript** |
|:--|:--|:--|:--|:--|:--|
| ⬜ **Screen** | `DATA` | content · then dot | `RSET` | `QUIT` | **Operator**&#10;<br>reads transcript |
| **Cmd / View** | 🟦 **BeginData** | 🟦 **SubmitContent** | 🟦 **Reset** | 🟦 **Quit** | 🟩 **SessionTranscript** |
| **Event** | 🟥 **DataPhaseEntered**&#10;<br>H1 — consumer? | 🟧 **MessageAccepted**&#10;<br>`queue_id` · `received_at` | 🟧 **TransactionAborted** | 🟧 **SessionClosed** | — |

`SessionTranscript` is the **"right chair"** shape — one read model fed by every event above.
Expected here; worth watching if it grows.

---

## 7. Worked paths as sequences

A path is a conversation over time. Two parties, one turn per row, read top to bottom — the one
place in this document where time runs **down** rather than across, because a protocol trace is a
dialogue rather than a model.

**Path 1** — [`../paths/helo-multi-recipient.md`](../paths/helo-multi-recipient.md). Three
recipients, one rejected mid-flow. 13 steps over 11 slices.

| # | `bar.com` → | ← `foo.com` (us) |
|--:|:--|:--|
| 1 | | `220 foo.com Service Ready` |
| 2 | `HELO bar.com` | |
| 3 | | `250 foo.com` |
| 4 | `MAIL FROM:<Smith@bar.com>` | |
| 5 | | `250 OK` |
| 6 | `RCPT TO:<Jones@foo.com>` | |
| 7 | | `250 OK` |
| 8 | `RCPT TO:<Green@foo.com>` | |
| 9 | | 🟥 `550 No such user here` |
| 10 | `RCPT TO:<Brown@foo.com>` | |
| 11 | | `250 OK` |
| 12 | `DATA` | |
| 13 | | `354 Start mail input` |
| 14 | content · then dot | |
| 15 | | `250 OK queued as q7F3A21` |
| 16 | `QUIT` | |
| 17 | | `221 closing channel` |

**Path 2** — [`../paths/helo-single-recipient.md`](../paths/helo-single-recipient.md). One
recipient, no errors. 11 steps over 11 slices.

| # | `foo.com` (relay) → | ← `xyz.com` (us) |
|--:|:--|:--|
| 1 | | `220 xyz.com Service Ready` |
| 2 | `HELO foo.com` | |
| 3 | | `250 xyz.com is on the air` |
| 4 | `MAIL FROM:<JQP@bar.com>` | |
| 5 | | `250 OK` |
| 6 | `RCPT TO:<Jones@XYZ.COM>` | |
| 7 | | `250 OK` |
| 8 | `DATA` | |
| 9 | | `354 Start mail input` |
| 10 | headers, body, then dot | |
| 11 | | `250 OK queued as x91B4C7` |
| 12 | `QUIT` | |
| 13 | | `221 closing channel` |

Outcome is relative to the actor. Path 1 is a complete success for the server, **partial** for the
sender, and a failure for `Green`. Path 2 succeeds for everyone. `Reset` — slice 10 — remains the
only slice no path has touched.
