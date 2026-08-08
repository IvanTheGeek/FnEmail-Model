# Re-walk — the direct path under command/view cadence

**Open. Nothing here is adopted.** A parallel walk of
[`helo-direct-single-recipient.md`](helo-direct-single-recipient.md) with everything settled on
2026-08-07 applied at once, so the two can be read side by side.

> ⚠️ **2026-08-07, end of day: partially superseded as a walk, kept as the reasoning record.** The
> final semantics this document arrived at are instantiated cleanly in
> [`WORKING-helo-direct-single-recipient-v2.md`](WORKING-helo-direct-single-recipient-v2.md) —
> read that file for the path; read this one for how each decision was reached, corrected and
> ruled. The WORKING- prefix stays while Ivan walks through it.

What changed, and why a re-walk rather than an edit: **a server reply is a rendered read model, not
part of the command that provoked it.** Splitting each reply out of its command's wire row turns
ten steps into fifteen and puts four replies in front of read models that **do not exist in the
model**. That is not a formatting difference, so it was walked again rather than patched.

| Applied | From |
|:--|:--|
| Commands carry the fields they take from the actor | `STEP-FORM.md` |
| `Given` is minimal, one row, Event-row structure | settled 2026-08-07 |
| 🟤 marks a `Given` with nothing, and only a `Given` | settled 2026-08-07 |
| A rendered view's top row is the wire it draws | this document |
| A consulted view has **no** top row — its readers declare it | `EXPLORE-view-slice.md` |
| `Where` is a query predicate, written with `=` | `EXPLORE-view-slice.md` |
| No `Consumed by` anywhere — dependencies point backward only | `EXPLORE-view-slice.md` |
| A required-events preamble — the path-level `Given` | **trial**, 2026-08-07 — the 🟨 section at the end |

---

## The dialogue being walked

Unchanged from the original path. RFC 5321 D.1 reduced to one recipient, `HELO` for `EHLO`.

```
S: 220 foo.com Simple Mail Transfer Service Ready
C: HELO bar.com
S: 250 foo.com
C: MAIL FROM:<Smith@bar.com>
S: 250 OK
C: RCPT TO:<Jones@foo.com>
S: 250 OK
C: DATA
S: 354 Start mail input; end with <CRLF>.<CRLF>
C: <the message, then a lone dot>
S: 250 OK
C: QUIT
S: 221 foo.com Service closing transmission channel
```

**Seven server replies.** The original walk absorbed all seven into command steps. This one gives
each its own view.

---

## Required first &nbsp;*(trial, 2026-08-07)*

**The path-level `Given`: every event this walk requires but does not walk**, each with the fields
some step consumes. The path is deliberately indifferent to provenance — whose event it is, what
kind, where the value came from. **Needed is the only claim 🟨 makes here**, and this block is the
only place 🟨 appears in a path: steps cite every event 🟧. Trial of the preamble proposed in the
🟨 section at the end of this document, which records the rule 9 amendment this makes.

| 🟨 Required first | *before this walk's timeline* |
|:--|:--|
| | 🟨 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`greeting_text`: Simple Mail Transfer Service Ready |
| | 🟨 **RecipientResolved**&#10;<br>&nbsp;&nbsp;`is_local`: true&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

Both names are path-local placeholders, and that is the point: the path does not care.
`ServiceConfigured`'s real name and shape are open — *Walking backwards*, below. `RecipientResolved`
stands for whatever H3's deferred translation eventually names its events — a concern the model
owns, elsewhere. Here each is only *an event some step's `Given` needs, carrying the values it
needs*.

Three trial decisions, made so they can be judged — and a ruling that followed:

**No forward pointers.** The block states what must exist and with which values — it does not say
which steps consume what. That is the `Consumed by` kill applied at path level: consumers declare
their own dependencies, and a consumption list here would go stale the same way. The first pass
deliberately left steps 2, 4 and 16 rendering `server_domain` **without declaring it** — one
change at a time; a second pass the same day added the `Given` to all three, so the declarations
now live where they belong, on the consumers.

~~**Both kinds of outside are in one block, told apart by annotation.**~~ **Superseded the same
day by the second ruling below.** The entries briefly carried provenance annotations — *ours,
merely earlier* against *another context's* — until Ivan ruled that the path does not make that
distinction at all. Provenance now lives in the prose under the table, as reading aid only, and
carries no weight in the path.

**The `RecipientResolved` entry duplicates step 7's `Given`, and that is accepted as a different
claim.** Step 7 declares a *slice* dependency; this block declares what a **replay of the whole
path must seed** before step 1 can run — the same relationship at two altitudes, the way a
scenario's `Given` restates a contract's precondition without replacing it. If that reading is
wrong, the entry is redundant and the block should hold only dependencies no walked step declares.
Left in so the trial tests the larger block.

**Ruled by Ivan, in two passes.** First: **the preamble seeds this path's event store, so steps
cite 🟧.** Once an event is declared here it is in the store and accessible, and a step's `Given`
cites it exactly as it cites a walked event — 🟧, unannotated. Second, superseding the first
pass's remainder: **there is no step-level 🟨 at all.** The first pass had kept 🟨 at steps 7 and
8 for the Directory translation, on the ground that *no event of ours* was a different claim; Ivan
ruled that the preamble resolves that too. The path does not care where an event came from or what
kind it is — only that it must exist for the steps' `Given` rows to be satisfiable. Every such
requirement is declared here, whatever its provenance, and every step cites every event 🟧 — the
Directory dependency under the path-local name **RecipientResolved**. That amends the settled
*external yellow* meaning for path documents — a rule 9 amendment made deliberately, recorded here
and in the 🟨 section below.

---

## The walk

| 🟦 C · Step 1 | `AcceptConnection` |
|:--|:--|
| | 🟦 **AcceptConnection**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |
| Event | 🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |
| Given | 🟤 |

⚠️ **Reworked 2026-08-07 — this step carried a *known wrong* disclaimer, resolved as follows.** Two
things were wrong, both inherited from the original walk. The top row said **MTA Client**, and the
client is not this command's actor — it sends no bytes; the fields are *"determined from the TCP
connection"*, §4.4's words, in the one place the RFC names the transport. And the wire row held
prose instead of wire, because there is no wire to hold. The fix is the same absence a consulted
view already uses: **a step whose actor is outside the model gets no top row.** The transport stays
unmodeled; its two facts are lifted across the boundary because the trace header — the fifth
output, below — consumes `peer_address`. The `Given` stays 🟤: the listener this accept presupposes
is real, but nothing in this walk consumes the binding fact, and a dependency nothing consumes is
not declared.

| 🟩 V · Step 2 | `ServiceReady` &nbsp;🟥 *not in the model* |
|:--|:--|
| MTA Client | ⬛ `220` foo.com Simple Mail Transfer Service Ready |
| | 🟩 **ServiceReady**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`accepting`: true |
| Given | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`greeting_text`: Simple Mail Transfer Service Ready&#10;<br>🟧 **ConnectionAccepted** |
| Where | `session_id` = 01J8Z… |

⚠️ **No such read model exists.** `server_domain` is configuration, which is **H6**. The greeting
was previously just text on a command's wire row; giving it a view forced the question of what
state produced it — and the answer is now declared: the `Given` carries the preamble-seeded
**ServiceConfigured**, fields included — 🟧, because the preamble puts it in this path's event
store, per the ruling in the trial block.

| 🟦 C · Step 3 | `Helo` |
|:--|:--|
| MTA Client | ⬛ `HELO` bar.com |
| | 🟦 **Helo**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Event | 🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Given | 🟧 **ConnectionAccepted** |

| 🟩 V · Step 4 | `SessionState` |
|:--|:--|
| MTA Client | ⬛ `250` foo.com |
| | 🟩 **SessionState**&#10;<br>&nbsp;&nbsp;`identified`: true&#10;<br>&nbsp;&nbsp;`transaction_open`: false |
| Given | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>🟧 **ConnectionAccepted**&#10;<br>🟧 **ClientIdentified** |
| Where | `session_id` = 01J8Z… |

⚠️ **The reply renders `foo.com` and this view does not carry it.** Was *same gap as step 2*; the
`Given` now declares the origin, which narrows it. What remains: either `SessionState` gains a
`server_domain` field, or the rendering draws the value straight from the folded `ServiceConfigured`.
The trial deliberately does not choose.

| 🟦 C · Step 5 | `MailFrom` |
|:--|:--|
| MTA Client | ⬛ `MAIL FROM:`\<Smith@bar.com> |
| | 🟦 **MailFrom**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com> |
| Event | 🟧 **MailTransactionStarted**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com> |
| Given | 🟧 **ClientIdentified** |

| 🟩 V · Step 6 | `TransactionState` |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **TransactionState**&#10;<br>&nbsp;&nbsp;`open`: true&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipient_count`: 0 |
| Given | 🟧 **MailTransactionStarted** |
| Where | `session_id` = 01J8Z… |

**No top row.** Nothing is drawn to any actor; `RcptTo` declares this dependency in its own `Given`.

| 🟩 V · Step 7 | `RecipientDirectory` &nbsp;*(consulted)* |
|:--|:--|
| | 🟩 **RecipientDirectory**&#10;<br>&nbsp;&nbsp;`is_local`: true |
| Given | 🟧 **RecipientResolved**&#10;<br>&nbsp;&nbsp;`is_local`: true&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Where | `forward_path` = \<Jones@foo.com> |

| 🟦 C · Step 8 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Given | 🟧 **MailTransactionStarted**&#10;<br>🟧 **RecipientResolved**&#10;<br>&nbsp;&nbsp;`is_local`: true&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

| 🟩 V · Step 9 | `TransactionState` &nbsp;*(second traversal)* |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **TransactionState**&#10;<br>&nbsp;&nbsp;`open`: true&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipient_count`: 1 |
| Given | 🟧 **MailTransactionStarted**&#10;<br>🟧 **RecipientAccepted** |
| Where | `session_id` = 01J8Z… |

| 🟦 C · Step 10 | `BeginData` &nbsp;🟥 **H1** |
|:--|:--|
| MTA Client | ⬛ `DATA` |
| | 🟦 **BeginData** |
| Event | 🟧 **DataPhaseEntered**&#10;<br>&nbsp;&nbsp;*no payload* |
| Given | 🟧 **MailTransactionStarted**&#10;<br>🟧 **RecipientAccepted** |

| 🟩 V · Step 11 | `DataPrompt` &nbsp;🟥 *not in the model — see H1 below* |
|:--|:--|
| MTA Client | ⬛ `354` Start mail input; end with `<CRLF>.<CRLF>` |
| | 🟩 **DataPrompt**&#10;<br>&nbsp;&nbsp;`awaiting_content`: true |
| Given | 🟧 **DataPhaseEntered** |
| Where | `session_id` = 01J8Z… |

| 🟦 C · Step 12 | `SubmitContent` |
|:--|:--|
| MTA Client | ⬛ `Date:` Tue, 19 May 1998 09:14:02 -0700&#10;<br>`From:` Smith \<Smith@bar.com>&#10;<br>`To:` Jones@foo.com&#10;<br>`Subject:` Tuesday&#10;<br>(blank)&#10;<br>Blah blah blah...&#10;<br>`.` |
| | 🟦 **SubmitContent**&#10;<br>&nbsp;&nbsp;`content`: 194 octets, dot-unstuffed |
| Event | 🟧 **MessageAccepted**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipients`: [\<Jones@foo.com>]&#10;<br>&nbsp;&nbsp;`content_ref`: blob:sha256:9c1e…&#10;<br>&nbsp;&nbsp;`actual_octets`: 194&#10;<br>&nbsp;&nbsp;`received_at`: 1998-05-19T09:14:07-07:00 |
| Given | 🟧 **DataPhaseEntered** |

| 🟩 V · Step 13 | `MessageTrace` &nbsp;🟥 *not in the model — the fifth output, worked in* |
|:--|:--|
| Stored message | ⬛ `Received: from` bar.com ([203.0.113.20])&#10;<br>&nbsp;&nbsp;`by` foo.com `with` SMTP&#10;<br>&nbsp;&nbsp;`id` f2C8D14&#10;<br>&nbsp;&nbsp;`for` \<Jones@foo.com>;&#10;<br>&nbsp;&nbsp;Tue, 19 May 1998 09:14:07 -0700 |
| | 🟩 **MessageTrace**&#10;<br>&nbsp;&nbsp;`from_domain`: bar.com&#10;<br>&nbsp;&nbsp;`address_literal`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`by`: foo.com&#10;<br>&nbsp;&nbsp;`with`: SMTP&#10;<br>&nbsp;&nbsp;`id`: f2C8D14&#10;<br>&nbsp;&nbsp;`for`: \<Jones@foo.com>&#10;<br>&nbsp;&nbsp;`at`: 1998-05-19T09:14:07-07:00 |
| Given | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP&#10;<br>🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com>&#10;<br>🟧 **MessageAccepted**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`received_at`: 1998-05-19T09:14:07-07:00 |
| Where | `queue_id` = f2C8D14 |

**The fifth output, worked into the walk 2026-08-07** — see *The fifth hidden output* below. §4.4's
MUST fires at receipt, so this sits between the content arriving and the `250` announcing
acceptance. It takes the second of the two shapes recorded there: **a view whose wire row is the
stored message, not the socket** — nothing here reaches the MTA Client. Two things are concrete at
this step and nowhere else:

- **The walk's only `Given` that needs values from a walked event, not existence** — five events,
  fields stated, including the protocol's sole consumption of `peer_address`.
- **H6 stops being abstract here.** The `by` clause renders foo.com from `ServiceConfigured` — the
  config arm. On the multi-homed arm it would render from `ConnectionAccepted.local_address`
  instead, which is that field's only candidate consumer in this walk. **`local_address` is
  consumed by no step in this path**; it stays in step 1's payload only because H6 is open.

| 🟩 V · Step 14 | `MessageQueued` &nbsp;🟥 *not in the model* |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **MessageQueued**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`accepted`: true |
| Given | 🟧 **MessageAccepted** |
| Where | `session_id` = 01J8Z… |

**The responsibility boundary is here.** Left of it, abandoning costs nothing; at `MessageAccepted`
we have accepted responsibility for delivering or reporting failure — RFC 5321 §2.1. **The `250` is
the moment the client learns that**, which the original walk did not make a step of its own.

| 🟦 C · Step 15 | `Quit` |
|:--|:--|
| MTA Client | ⬛ `QUIT` |
| | 🟦 **Quit** |
| Event | 🟧 **SessionClosed**&#10;<br>&nbsp;&nbsp;`cause`: quit |
| Given | 🟧 **ConnectionAccepted** |

| 🟩 V · Step 16 | `SessionClosing` &nbsp;🟥 *not in the model* |
|:--|:--|
| MTA Client | ⬛ `221` foo.com Service closing transmission channel |
| | 🟩 **SessionClosing**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`cause`: quit |
| Given | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>🟧 **SessionClosed** |
| Where | `session_id` = 01J8Z… |

---

## What the re-walk exposed

### 1. The model has four read models and the protocol renders seven replies

| Reply | Rendered by | In the model? |
|:--|:--|:--|
| `220` greeting | `ServiceReady` | ❌ |
| `250` after `HELO` | `SessionState` | ✅ |
| `250` after `MAIL FROM` | `TransactionState` | ✅ |
| `250` after `RCPT TO` | `TransactionState` | ✅ |
| `354` after `DATA` | `DataPrompt` | ❌ |
| `250` after content | `MessageQueued` | ❌ |
| `221` after `QUIT` | `SessionClosing` | ❌ |

**Four of seven replies have no read model behind them.** They were invisible while each reply sat
as text on a command's wire row. This is the completeness check running in the direction it was
always meant to run: *what state produced this output?*

⚠️ **Naming these four is not proposing them.** Some may collapse — `ServiceReady` and
`SessionClosing` both render `server_domain` and may be one lifecycle view; `DataPrompt` may be
`TransactionState` in another state. **Do not add four slices on the strength of this table.**

### 2. H1 may be resolved by the cadence

`DataPhaseEntered` carries no payload and the open question is whether it earns its place. Its only
candidate consumer was *"the transcript rendering `354`"*.

**Under the cadence it has a direct consumer**: step 11's view, whose `Given` is exactly
`DataPhaseEntered` and whose sole job is to render the `354`. An event that a view folds is an event
that is used.

⚠️ **Candidate, not a resolution.** It depends on accepting that a `354` prompt is a rendered read
model rather than an artifact of the command. If that is accepted, H1 dissolves; if it is not, H1
stands untouched. Rule 9 — recorded here, not moved in `event-model.md`.

### 3. The server's own domain has no home

`foo.com` renders in three replies — `220`, `250` after `HELO`, and `221`. **No read model in the
model carries it.** It is configuration, which is **H6**, and the re-walk shows H6 is not a detail
of the `Received:` header but a value the protocol emits at the very first and very last thing the
client ever sees.

### 4. ⚠️ The cadence is not doctrine — the corpus prescribes *dependency*, not *sequence*

Asked for verification 2026-08-07 rather than taken on my assertion. **Nothing in the corpus requires
command and view to alternate.** What it prescribes is narrower and different.

**Slices split by type, and that rule is explicit.** The slicing kit, line 33:

> *"A slice never mixes a COMMAND and a READMODEL — the platform models these as two distinct slice
> types (`state-change` and `state-view`). If a 'feature' needs both a command and a read model …
> that's **two slices**, not one."*

**Two of the four patterns are *already* a view followed by a command.** Automation and Translation
are both composed **read + write**, and Dymitruk specifies an automation as **two** specifications:

> *"The specification for this is always done in 2 parts: The Given-When-Then for creating the
> to-do list and the Given-When-Then for executing the command."*

So **V → C is not merely allowed, it is a named composition.** Steps 7 and 8 of this walk —
`RecipientDirectory` then `RcptTo` — are **one Translation**, which is *"an automation whose input
events belong to someone else's system"*. Step 7 is not a stray view; it is the read half of a
pattern that has a name.

**And slices relate by dependency, not by position.** The kit's step 4 is *"Note Dependencies Between
Slices"*, illustrated as *"`OrderDetailView` depends on `PlaceOrder`'s `OrderPlaced` event"*. That is
a **graph**, not a sequence — and it is exactly what a `Given` row expresses.

**So the V · V at steps 6 and 7 violates nothing.** It is the end of a State View meeting the start
of a Translation: two patterns adjacent, which the corpus neither forbids nor discusses.

⚠️ **Scope of this check.** Searched the method reference and the slicing kit — the two places an
ordering rule would live. **Not an exhaustive sweep**, so this is *no rule found where a rule would
be*, not *proven absent*.

> ### ✅ Full sweep completed 2026-08-07 — verdict: **artifact**, and stronger than "no rule found"
>
> 35 talks, 46 podcast episodes, both sites, the kits and the drawio, with three adversarial passes.
> Detail in `event-modeling-research:research/CADENCE-FINDINGS.md`. Four things change what is
written above.
>
> **⚠️ *Cadence* is his word — and it means the wiring rule, not slice order.** SE Radio 539:
> *"there's definitely a Cadence like an opinion as to the flow and what arrows can connect"* —
> forcing the UI to hit a blue box, then orange, then green. **Anyone grepping for the term lands on
> that and reads it as alternation.** All 33 hits in the 2015-2020 talks carry one of three senses,
> none of them sequence. This document used the word in the wrong sense in its own title.
>
> **The wave is explicitly variable-period.** *"this kind of makes this nice sine wave across this
> time line so they can be longer or shorter sine waves"* — a wave whose wavelength varies is not a
> fixed C·V period.
>
> **The canonical article argues against pairing.** Steps 4 and 5 add **all** commands, then **all**
> views — two bulk passes, which no method prescribing pairing would prescribe — and *"Features can
> be created in any order."* Its actual rule is that a step is tied to **either** a command or a
> view (**typed, not ordered**) and that **all information has to have an origin and a destination**.
>
> **The negatives are the evidence.** *rhythm*, *ping pong*, *flip flop*, *zigzag*, *oscillate*,
> *interleave*, *take turns* — **zero hits corpus-wide**. He never raises adjacent same-type slices
> as a question, so he neither defends nor forbids them.
>
> Two passages that read like prohibitions are not. Podcast 24's *"you can't put another command
> after that"* is followed by *"are you describing the same slice? No"* — it is the
> one-command-per-slice rule **inside** a slice. And *a command cannot kick off a command* is about
> **arrows**, not columns.
>
> ⚠️ **And the binary is not even total.** An Automation is composite — read model, processor and
> command in one step — so it cannot be a term in a C·V alternation at all. There is also a modeled
> navigation step that changes no state and is neither type.

⚠️ **My drawio parse was wrong, and the corrected one still answers the question.** I reported 34
slices, strict alternation for eight, and a run of four Views. Restricting to the timeline band
instead of raw x-containment gives **28 boxes — 12 Command, 16 View — alternation for six, and a run
of three**. The inflation came from the Given/When/Then stacks hanging *below* the timeline, which
repeat the same boxes at successive states. Three parses of one file now disagree on the count and
**all three agree that long adjacent same-type runs are present in both directions**.

⚠️ **And the alternation is an observation of this protocol, not of the method.** RFC 5321 §4.3.1
calls SMTP an *"alternating dialogue"*. That is why this walk alternates. **The regularity comes from
SMTP, not from Event Modeling** — which is the opposite of what I implied when I first wrote the
cadence up as though the method demanded it.

### 4b. Cadence is per lane, not global

Steps 6, 7 and 8 run **V · V · C**, which breaks a strict alternation. They do not break the rhythm,
because the two views are in **different lanes**: step 6 renders to the MTA Client, step 7 is
consulted internally and renders to nobody.

That is what Dymitruk's sysadmin swimlane is for — verified in his own file, whose actor lanes are
Joe the Organizer, Adam the Participant, Alice the misfit and Eugene the Sysadmin. **Read per lane,
the alternation holds.** A flat step list cannot show that, and this document does not solve it.

### 5. Fifteen steps, and the accounting changes

| | Original | Re-walk |
|:--|:--|:--|
| Steps | 10 | **15** |
| Command steps | 7 | 7 |
| View steps | 3 | **8** |
| Distinct slices touched | 10 | **11**, four of which do not exist |
| Replies shown as wire text inside a command | 7 | **0** |

The command steps are unchanged in number and nearly unchanged in content. **The entire growth is
views** — which is the point: the original walk had been hiding read models inside wire rows.

⚠️ **Count updated 2026-08-07.** The trace-header step — `MessageTrace`, step 13 — makes it
**sixteen steps, nine of them views, twelve distinct slices touched, five of which do not exist in
the model**. The table and the heading's fifteen are left as written; they were true of the re-walk
this section describes, and the sixteenth step is not a reply but the output written into the
stored message.

⚠️ **And *twelve* was itself a miscount**, caught by the v2 verification pass the same day:
**fifteen** distinct slices — seven commands, three model views, five path-defined views, with
`TransactionState` traversed twice. The *five of which do not exist* half was right.

---

## Against adopting this

Written before any decision, so the case is on record rather than assembled afterward.

**It invents four slices to make a rhythm work.** The strongest objection. `ServiceReady`,
`DataPrompt`, `MessageQueued` and `SessionClosing` exist here because the form demanded something to
render each reply, not because anything in the charter needs them. That is modeling ahead of need,
and rule 10's whole point is that orthodoxy stays measurable only if extensions are not folded in
quietly.

> ### ⚠️ Withdrawn 2026-08-07 — the RFC says these are required
>
> **That objection is wrong, and the RFC overturns it in the strongest available terms.** Raised by
> Ivan with a role-reversal argument: flip the roles, and as an SMTP *client* we **cannot issue the
> next command until we have received and read the reply**. The reply is not decoration around a
> command; it is a value the next step depends on.
>
> **RFC 5321 §4.3.1, *Sequencing Overview*** — read rather than recalled:
>
> > *"The communication between the sender and receiver is an **alternating dialogue**, controlled by
> > the sender. As such, the sender issues a command and the receiver responds with a reply. Unless
> > other arrangements are negotiated through service extensions, the sender **MUST wait for this
> > response before sending further commands**."*
>
> Three things fall out, and the first is the one that matters most.
>
> **The specification calls SMTP an *alternating dialogue*.** The command/view cadence was not
> imported from Event Modeling and imposed on this protocol — **it is the protocol's own
> description of itself.** The section is titled *Sequencing of Commands and Replies*. A form that
> alternates command and view is not a rhythm being imposed; it is the shape of the thing.
>
> **The reply is a MUST, not a courtesy.** A client that does not wait is non-conforming. So a
> read model behind each reply is not a slice invented to fill a slot — it is the state that a
> **mandatory** synchronization point publishes. Under the completeness check, an output the
> protocol requires must have an origin, and four of ours had none.
>
> **And the escape clause does not apply to us.** *"Unless other arrangements are negotiated through
> service extensions"* is `PIPELINING`, RFC 2920, referenced at line 443. FnEmail's charter is
> `HELO`-only with **no ESMTP**, so nothing can ever be negotiated and **the MUST holds
> unconditionally within this scope**.
>
> ⚠️ **One genuine asymmetry survives, and it weakens exactly one of the four.** The greeting is
> a **SHOULD**: *"The sender SHOULD wait for this greeting message before sending any commands."*
> Every other reply is a MUST. So `ServiceReady` rests on weaker ground than `DataPrompt`,
> `MessageQueued` and `SessionClosing` — and this project has been caught once already treating a
> SHOULD as a MUST, which reclassified a whole event when corrected.
>
> **What remains of the objection:** not that the four are invented, but that naming them here is
> still premature. *That* they must exist is now established. *What they are called, and whether
> some collapse into one lifecycle view, is not.*

**Fifteen steps is a worse read than ten.** A walk is for showing someone the model for the first
time. Half the steps now say *and then the server replied*, which is the least surprising thing in
SMTP.

**The gain may be one-off.** Points 1 and 3 above are findings, and findings can be recorded in
prose without restructuring every path. Once written down, the re-walk has delivered most of its
value and the fifteen-step form has to justify itself on readability alone.

**Against all that:** the four missing read models were **invisible** for the entire life of the
original document, and no amount of re-reading it surfaced them. It took changing the form to see
them. That is worth something even if the form is then discarded.

---

## Walking backwards past Step 1 — asked 2026-08-07

The walk's first view renders `220` foo.com Simple Mail Transfer Service Ready. Its `Given` is
**ConnectionAccepted**, which step 1 satisfies — and step 1's `Given` is 🟤. Working backwards under
the RFC: is that chain actually closed, and if it is not, **what are the previous steps?**

### The greeting, value by value

| On the wire | What fixes it | Origin in the `Given` chain |
|:--|:--|:--|
| `220` rather than `554` or `521` | §3.1 and RFC 7504 §3 — the acceptance posture is a genuine three-way decision | 🟧 **ConnectionAccepted** — under the current stance the event fires only on the accept path, so its **existence** folds to `accepting`: true, exactly as existence folds to `identified`: true in the walk's `SessionState` |
| foo.com | the grammar — see below | ❌ **none** |
| Simple Mail Transfer Service Ready | nothing — `textstring` is optional and operator-chosen; §3.1 lets it carry software and version | ❌ **none** — though omitting it is conformant, so it needs an origin only if rendered |

The identity is not decoration. Three RFC 5321 passages, read rather than recalled:

**The grammar requires it** — §4.2, where an optional element is spelled with square brackets, and
the identity group carries none. The `Greeting` rule's opening line:

```
Greeting       = ( "220 " (Domain / address-literal)
```

The continuation under it brackets only the trailing human text — `[ SP textstring ] CRLF` — and
the rule's multiline form repeats the shape: every alternative opens with the bare identity group,
and `textstring` is the only element the rule ever brackets. A `220` line with no identity does not
parse.

The rule's own six-line layout is deliberately not reproduced. ABNF continuation lines hang at the
`=` column, and lifted out of the RFC's page that layout reads as broken markdown — a dangling
`) /`, bracket fragments floating mid-block. Quoted one line at a time instead, each byte-exact
against [`../rfc/rfc5321.txt`](../rfc/rfc5321.txt) ll. 2588–2593.

**Machines parse it** — §4.2: *"In particular, the 220, 221, 251, 421, and 551 reply codes are
associated with message text that must be parsed and interpreted by machines."*

**It is the server's official name** — §4.3.1: *"all the greeting-type replies have the official
name (the fully-qualified primary domain name) of the server host as the first word following the
reply code."*

And a fourth, smaller than the others: even the greeting's **timing** renders server state.
§4.5.3.2.1: *"Many SMTP servers accept a TCP connection but delay delivery of the 220 message until
their system load permits more mail to be processed."* The five-minute client timeout exists for
exactly the gap between `ConnectionAccepted` and this view rendering.

⚠️ **One escape hatch, and it is load-bearing for H6.** The identity slot admits an address literal
— the RFC's own example is `220` [10.0.0.1] Clueless host service ready — and our literal would be
[192.0.2.10], which **is** in the chain, as `ConnectionAccepted.local_address`. A nameless server
can greet from what the walk already carries. So the grammar alone is satisfiable; **what is
unsatisfiable is this walk's dialogue**, which renders foo.com, a configured name no event
originates. That hands `local_address` a **second candidate consumer** — H6 had given it exactly
one, the multi-homed `BY` clause.

### No further Givens exist inside the timeline

`ConnectionAccepted` is the walk's left edge: §3.1 — *"An SMTP session is initiated when a client
opens a connection to a server and the server responds with an opening message."* Nothing in the
protocol's session precedes it, and every one of the fifteen steps is **per-session**. The value
with no origin is **per-service-lifetime**. So the missing step is not hiding between existing
steps — the chain exits the walk entirely, into a lane that was configured before any client ever
connected. That is also why three walked paths never met it: rule 8 walks sessions with real data,
and this fact predates every session there is.

### The corpus answers this exact question, by name

Walking backwards is Dymitruk's own device, and he names its terminus. Copenhagen DDD talk, machine
transcript:

> *"you can keep going to in the left direction all the way until you specify the installer with
> what the initial values are for your for your config fields"*

The workshop recording, machine transcript, states it as practice: *"walking an event model
backwards asking where does this come from where does this come from will continue to give you the
motivation to keep specifying things to the to the logical start"* — and taken to the extreme,
*"you can even make your project installer or setup scripts um with all the default values that you
need"*. Podcast episode 8, machine transcript, is how his teams deploy: production systems ship with
*"assumed events that are there at the beginning"* instead of setup screens. The canonical article
does it on the page — its view specification opens *"**Given**: hotel is set up with 12 ocean view
rooms"* — the setup is prior events in a `Given`. And his own open-spaces file models it: Eugene the
Sysadmin's lane holds a provisioning workflow — `RequestUniqueID` through `ProvideID` carrying the
real value 1111-2222-3333 — placed on the timeline **before** the participant's first slice, which
consumes that value.

The boundary is equally explicit. Asked what events a running system yields, he names *"started at
the time it stayed up for a week"* and disqualifies them — *"those are non-business specific ones"*
(machine transcript). So the split is: **provisioning that originates information the model
consumes belongs on the timeline at the logical start; bare lifecycle facts do not.** The missing
step here passes that test on the first half — it is named by what it originates, the configured
identity, not by the lifecycle fact that a process started.

### What the previous step would be — two shapes, neither chosen

**Shape A — configuration is ours.** An operator-lane Command Slice, walked once per service
lifetime, and the model's true origin:

| 🟦 C · Step 0 | `ConfigureService` — *sketch, not in the model* |
|:--|:--|
| Operator | ⬛ *no SMTP bytes — the wire-row defect step 1 already carries* |
| | 🟦 **ConfigureService**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`listen_address`: 192.0.2.10:25&#10;<br>&nbsp;&nbsp;`greeting_text`: Simple Mail Transfer Service Ready |
| Event | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`listen_address`: 192.0.2.10:25&#10;<br>&nbsp;&nbsp;`greeting_text`: Simple Mail Transfer Service Ready |
| Given | 🟤 |

The 🟤 moves back one step and is finally carrying its full weight: this is the element with no
inbound dependency, which the corpus calls the model's origin.

**Shape B — configuration is someone else's.** The `RecipientDirectory` precedent: a 🟨 `Given`,
translated from a **Configuration context**, external and deferred. No new slice of ours; a typed
boundary instead.

The corpus supports both and then collapses the distinction — the book, on a value arriving by
notification or by hand: *"From the system perspective - it could as well be a simple API Call, a
Webhook or a manual price configuration. From the process-perspective, it´s basically all the same
operation."* It is also in verified conflict with itself about whether technical configuration
belongs on the model at all — the installer passages above against *"it can be put into a side
project called infrastructure"* (DDD Greece meetup, machine transcript) — so the choice is genuinely
ours to make, and it is not made here.

### What the repaired chain looks like

⚠️ **Implemented later the same day.** The *Required first* trial at the top of this document is
this table's repair: the preamble declares the origin, and steps 2, 4 and 16 carry the declaration
🟧 per the trial's ruling. Step 1 keeps 🟤 exactly as the last row argues. The table is left as
written — it is the reasoning that produced the preamble.

| Step | `Given` today | Under either shape |
|:--|:--|:--|
| `ServiceReady` (step 2) | 🟧 **ConnectionAccepted** | gains the origin — **required**; it renders `server_domain` and the text |
| `SessionState` (step 4) | 🟧 two events | gains the origin — required **only while the walk renders** `250` foo.com; see the tier note below |
| `SessionClosing` (step 16) | 🟧 **SessionClosed** | gains the origin — required; `221` is greeting-type |
| `AcceptConnection` (step 1) | 🟤 | **separable, and not forced.** A listener bound to `listen_address` is a real precondition, but the binding and the naming are different facts, and the binding has no consumer anywhere in this walk — under *admitted by consumer* it does not earn the dependency on its own |

⚠️ **The three renderings of foo.com sit on three different strengths, and the walk had them flat.**
The `220` is **grammar** — (`Domain` / `address-literal`), mandatory. The `221` is **greeting-type**
— the §4.3.1 note plus §4.2.2's function group, where `220`, `221` and `421` are the only codes shown
\<domain>-first. But the `250` after `HELO` rests on **prose alone**: the `ehlo-ok-rsp` ABNF is
introduced for EHLO's reply and nothing defines a HELO reply syntax — it falls back to `Reply-line`,
which requires no domain. What remains is §4.1.1.1: *"The SMTP server identifies itself to the SMTP
client in the connection greeting reply and in the response to this command"* — descriptive, no
MUST. Step 4's flagged gap is real, and it is the weakest of the three. (Appendix D corroborates
nothing either way: all four of its scenarios open with `EHLO`; no HELO exchange appears anywhere in
it. The original path's *the dialogue is ours* flag was already carrying that fact.)

⚠️ **The refused-greeting renderings still need the session to exist.** §3.1 on the `554` opening:
the server **MUST** *"still wait for the client to send a QUIT"* and **SHOULD** *"respond to any
intervening commands with "503 bad sequence of commands""* — one MUST, one SHOULD, kept apart on
purpose. A refused session therefore still runs to `Quit`, which means `ConnectionAccepted` still
fires on that path — and that collides with the model's older contract, *reply 554, no event*. Not
resolved here; if this form is adopted, that contract takes a rule 4 correction rather than a quiet
edit. (RFC 7504 differs for `521`: after it, the server *"MAY either continue sending 521 reply
codes or simply close the connection"* — the wait is not a MUST there.)

### What this is adjacent to, and deliberately does not touch

**H6, adjacent but not identical.** H6 as recorded chooses between two *sources* for one output —
is `Received:`'s `BY` config or `local_address`. This finding is a different decision about the same
topic: the *entry mechanism* for configuration itself. Either arm of H6 can close and the greeting
still needs the configured name; either shape here can be picked and `BY`'s source is still open.
What the backward walk adds to H6 is scope: the same missing origin feeds the `220`, the `221`, the
`250` after `HELO` at prose strength, and — on H6's config arm — the `BY` clause.

**H7, narrower than it looks from here.** H7 governs only the `521` rendering, the
never-accepts-mail mode. The `554` blocklist branch is a separate product policy and survives H7
closing either way — so `accepting` does **not** collapse to a constant merely because H7 closes as
*never*.

**And no slice is being proposed.** Same discipline as the four named views above: *that* a step
precedes step 1 is established — by the greeting's grammar, the completeness check, and the corpus's
own installer terminus. *What it is called, which lane holds it, and whether it is a slice of ours
or a translation boundary* is a decision this document deliberately leaves open.

---

## The fifth hidden output — the trace header. Asked 2026-08-07

Asked directly: does any step in this walk ever reference `ConnectionAccepted`'s **fields**?
**No.** The event is named in four `Given` rows — `ServiceReady`, `Helo`, `SessionState`, `Quit` —
and every one is the existence degree: the event happened, nothing about its contents.
`peer_address` and `local_address` appear at step 1 and never again. Wider than that: **every
`Given` in the fifteen steps is existence-only** — the whole walk runs without a single data-degree
dependency. *(True when written. The preamble's later passes added data-degree entries to steps 2,
4 and 16, and step 13 now consumes `peer_address` — the repair this section called for.)*

The two fields sit in that position for two different reasons.

**`local_address` is the H6 orphan**, already flagged. The backward walk's address-literal escape
hatch is only a candidate consumer, and this dialogue renders foo.com, so here it stays unconsumed.

**`peer_address` has a real consumer, and this document lost it in the re-walk.** The original path
carries a *Completeness, instantiated* section whose table sources `Received:`'s address literal
from `ConnectionAccepted.peer_address`. This document dropped that section — and the completeness
direction it ran against the seven wire replies, *what state produced this output?*, was never run
against the one required output that is **not** a wire reply. RFC 5321 §4.4:

> *"When an SMTP server receives a message for delivery or further processing, it MUST insert
> trace ("time stamp" or "Received") information at the beginning of the message content"*

Receipt is acceptance, so the MUST fires **inside this walk's timeline**, at `SubmitContent`. An
output the protocol requires, produced by no step — the same class of finding as the four hidden
reply views, and the fifth member. The seven-reply table under *What the re-walk exposed* was
counting wire lines, and the protocol has an eighth output that never crosses the wire: it is
written into the stored message.

**What distinguishes the fifth from the four: its `Given` cannot be existence-only.** Rendering

```
Received: from bar.com ([203.0.113.20]) by foo.com with SMTP id f2C8D14
```

needs values, not existence — which would make it the walk's **first data-degree `Given`**,
whatever element it turns out to live on:

| Given | 🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP&#10;<br>🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com>&#10;<br>🟧 **MessageAccepted**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`received_at`: 1998-05-19T09:14:07-07:00 |
|:--|:--|

Plus the `BY` value — which is the backward walk's missing configuration origin, making the trace
header that section's fourth consumer in whichever shape the origin lands.

**The element's shape is deliberately not picked.** Two candidates, different timelines: the trace
line is part of what `SubmitContent` stores — in which case the fields are consumed inside the
command step and `content_ref` covers trace plus content — or it is a view rendered later from the
event stream, whose wire row is not the socket but the stored message. Both shapes carry the same
`Given`; choosing is model work, not walk work.

⚠️ **Picked for the walk later the same day — step 13, `MessageTrace`.** Ivan called it in. It
takes the second shape, a view whose wire row is the stored message; the first shape remains
arguable for the *model*. `MessageQueued`, `Quit` and `SessionClosing` renumbered to 14, 15 and 16
behind it.

**This is also H5 closing its loop in a walk for the first time.** `AcceptConnection` is admitted
to the model for exactly one reason — `peer_address` has a destination. Fifteen steps never showed
that destination, which made step 1 look like it feeds nothing. The fifth output is where the
admission ticket gets punched — now on the page, as step 13.

---

## 🟨 as *outside this path*, and a path-level `Given` — raised by Ivan, 2026-08-07

Recorded as direction, not decision — Ivan has his own plans for how the tool will represent
Translation, and this section captures the reframing rather than adopting it.

**The reading: 🟨 means *outside this path definition*.** Today the documents use 🟨 in the corpus
sense — another system's events, the Translation boundary. The reframing widens it, and doing so
surfaces a distinction the current form has no way to write:

| Outside **what**? | Example | How the form shows it today |
|:--|:--|:--|
| the model — someone else's system | the Directory context | 🟨, the settled sense |
| this path — ours, but not in this walk's timeline | the configuration origin; `ConnectionAccepted`, for a walk scoped narrower than a session | nothing — a walk either walks it or silently starts late |

⚠️ **Rule 9 note.** *External yellow* is a settled convention. Widening 🟨 to carry both outsides —
or splitting the second outside onto its own marker — is an amendment to record, not a quiet
change. Flagged here so the tension is on the page: the two outsides are different claims, and the
Directory translation must stay distinguishable from *merely earlier than this walk*.

⚠️ **Ruled 2026-08-07, after the preamble trial ran.** An event the preamble declares is
thereafter in this path's event store, and steps cite it 🟧 like any walked event. The first pass
of this ruling kept step-level 🟨 for the Directory translation, reading *no event of ours* as a
different claim.

⚠️ **Ruled again the same day, going further — and superseding the line above.** Step-level 🟨 is
not kept; it is **amended away for path documents entirely**. The path does not care where an
event or value came from, only that it is needed for the steps' `Given` rows — every requirement
is declared in the preamble, and every step cites 🟧, the Directory dependency included, under the
path-local name `RecipientResolved`. *External yellow* remains available to the **model**, where
the Translation boundary is real work; in a **path** it is superseded by the preamble. That is the
rule 9 amendment, now recorded in full rather than flagged as tension.

**The proposal: a path opens with the events it requires.** A predefined block before step 1 —
🟨-marked events with the fields some step in the walk will need — so a path declares its
prerequisites instead of inheriting them invisibly. That is the GWT `Given` lifted to path level.
The corpus already runs its deployment half — Dymitruk ships production systems with *"assumed
events that are there at the beginning"* (podcast episode 8, machine transcript, quoted in the
backward walk above) — and it hands the backward walk's origin a home that costs nothing: the
configuration fact lives in the preamble of every session-scoped path, fields stated, instead of
every walk paying a step for it.

**And it answers the question it arrived with.** Is `AcceptConnection` valid in this path, or is
`ConnectionAccepted` preamble material? **For this walk: it stays a step.** This walk is a whole
session, and the connection is the session's own first fact — inside the timeline, so it is walked,
with `peer_address` as a needed field whose consuming element is the fifth output above. The 🟨
conversion is the right shape **when a path's scope starts later** — a transaction-only walk would
open by declaring 🟨 **ConnectionAccepted** and 🟨 **ClientIdentified** with exactly the fields its
steps consume — and for the per-service-lifetime facts no session-scoped walk can ever contain.

The rule that falls out, stated so it can be argued with:

> **Walk what is inside the path's timeline; declare, with fields, what is before it.**
