# Path — HELO, direct, single recipient, clean · v2

The same scenario as [`helo-direct-single-recipient.md`](helo-direct-single-recipient.md), walked
under the semantics settled on 2026-08-07. **This file is the result; the reasoning is in
[`EXPLORE-rewalk-cadence.md`](EXPLORE-rewalk-cadence.md)** and is not repeated here.

Derived from **RFC 5321 Appendix D.1** — *A Typical SMTP Transaction Scenario* — reduced to one
recipient, with `HELO` substituted for `EHLO`. Verified against
[`../rfc/rfc5321.txt`](../rfc/rfc5321.txt).

⚠️ **The dialogue is ours, the topology is the RFC's.** D.1 sends to three recipients and rejects
one; no appendix scenario is *stated* to be both direct and single-recipient — D.4 has one
recipient and every reply succeeds, but it carries no prose, so its topology is unstated. This
walk is RFC-derived in structure and RFC-conformant in every reply code, but the exchange as
written does not appear in the specification.

**The form, in one paragraph.** A path opens with a **Required first** block — the path-level
`Given`, declaring every event the walk needs but does not walk, with the fields its steps consume.
🟨 appears only there, and makes exactly one claim: *needed, not walked here* — the path is
indifferent to provenance. Steps cite every event 🟧, walked or seeded. A server reply is a
**rendered view** — and a view is **the dataset provided to the actor to complete its task**, the
way a GUI takes a dataset and produces the finished page: the wire row is what the renderer
produces from the view's fields, so every varying value on the wire line must be a field, while
**the renderer owns the constants and the view owns the facts**. A view read only internally is
**consulted** and has no top row; a step whose actor is outside the model has no top row either.
A step is one table, no heading. The `Given` is a **labeled block, not a row**: the label stands
alone and its events follow beneath it. The **Required first** block shares half that shape — the
empty left label on content rows, the label read as a sub-header above its events — which is what
makes the path-level `Given` and the step-level one read as one device at two scales.
Dependencies point backward only.

**No *not in the model* markers.** Paths are the source and slices are derived — what a path
needs, it defines, and the union across paths populates the model
([`../DECISIONS.md`](../DECISIONS.md)). The six views and two seeded events this walk defines
beyond the current model are simply part of this path; reconciling the model is model work.
Hotspots appear only where open — see *Hotspots* at the end.

**The layering — ruled by Ivan, 2026-08-08.** A **step** is an instance of a slice, carrying
example data. The example shown is one *selected* for display — where Dymitruk and Dilger
typically show a single set of example data on a model, the true slice here composes **many**:
every walk that traverses it contributes its scenarios, and a step that revisits a slice —
`TransactionState`, steps 6 and 9 — extends the composed slice's GWT set rather than repeating it.
A **path** is an instance of a composed timeline part with specific data: a specific timeline. A
**workflow** is that composed, generic part of the model; a path of a workflow is the workflow
with the data filled in. Three layers, three concerns: the **path** speaks protocol with real
values and scopes by position on its own timeline; the **slice** aggregates across walks, which is
where selection among many instances becomes real and query keys become contracts; the **store**
is where H4 rules on streams and envelopes. A path carries only values the protocol itself moves.

⚠️ **"Ruled" was retracted the same day.** The method repo's
[`layering.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/layering.md) now opens
with a status banner: the layering is a working hypothesis, not settled — calling it ruled
overstated it (method repo commit fe0891e, 2026-08-08). The paragraph above stands as the frame
this walk worked under; read "ruled" as the hypothesis walks like this one are still testing, not
a closed decision.

⚠️ **The `Where` rows and the session ULID are gone, and the removal is the finding.** The first
draft of this file instantiated `session_id` as a real ULID — 00T22SDG4R9FQ3ZK7VWX2M5N8P, its
first ten characters encoding the scene's instant — and keyed seven `Where` rows with it.
Instantiating it is what exposed it, in four moves: no payload originates it; the RFC names no
session identifier at all, because while a session lives the channel is its identity; no emitted
output ever carries it, so it can never have a destination; and on a path even the *selection* it
powers is vacuous — a key exists to pick among interleaved sessions, interleaving cannot occur on
a single timeline, and the page is already the selection. Scoping on a path is **positional**:
everything between `ConnectionAccepted` and `SessionClosed` is the session, and `cause` enumerates
every way that bracket closes. The two non-session `Where` rows dissolved with it — step 7's
restated its own `Given`'s `forward_path`, and step 13's selected among a population of one.
`queue_id` stays, but as payload, because the protocol moves it. Where keys re-materialize is the
slice layer, as query contracts. Rule 8, again: the value did its job by dying.

*(The generic form these steps use is the method repo's
[`path-and-step-form.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/path-and-step-form.md);
[`STEP-FORM.md`](STEP-FORM.md) carries what SMTP adds, and caught up with this walk 2026-08-08.)*

---

## Scene

| | |
|:--|:--|
| Server (us) | foo.com at 192.0.2.10:25 |
| Client | bar.com at 203.0.113.20 — **the originating host, contacting us directly** |
| Sender | \<Smith@bar.com> — **same domain as the client** |
| Recipient | \<Jones@foo.com> — one, local |
| Time | Tue, 19 May 1998 09:14:07 -0700 |

**The alignment is the point.** `claimed_domain` and the `reverse_path` domain are both bar.com,
and the model still does not relate them — see *What this walk tested* in the
[original](helo-direct-single-recipient.md), which this file inherits rather than restates.

## The dialogue

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

Seven server replies, each a rendered view — and one eighth output that never crosses the wire,
written into the stored message at step 13.

---

## Required first

The path-level `Given`: every event this walk requires but does not walk, with the fields its
steps consume. Needed is the only claim made here.

| 🟨 Required first | *before this walk's timeline* |
|:--|:--|
| | 🟨 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`greeting_text`: Simple Mail Transfer Service Ready |
| | 🟨 **RecipientResolved**&#10;<br>&nbsp;&nbsp;`is_local`: true&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

Both names are path-local placeholders — the path does not care whose events these are or where
the values came from, only that a replay must seed them before step 1 can run.

---

## The walk

| 🟦 C · Step 1 | `AcceptConnection` |
|:--|:--|
| | 🟦 **AcceptConnection**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |
| Event | 🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |
| Given | |
| | 🟤 |

*No top row — the actor is the transport, outside the model. `peer_address` is "determined from
the TCP connection" (§4.4); `local_address` is the listening socket's, and H6's question.*

| 🟩 V · Step 2 | `ServiceReady` |
|:--|:--|
| MTA Client | ⬛ `220` foo.com Simple Mail Transfer Service Ready |
| | 🟩 **ServiceReady**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`greeting_text`: Simple Mail Transfer Service Ready |
| Given | |
| | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`greeting_text`: Simple Mail Transfer Service Ready&#10;<br>🟧 **ConnectionAccepted** |

*Ruled 2026-08-08, both ways at once: `greeting_text` joins the view, `accepting` leaves it. The
view is the dataset the renderer draws from, so the text the wire shows had to be a field — and a
boolean nothing renders had no business being one. `accepting`'s only origin was an existence
fold standing on the contract this file already flags as wrong (the refused greeting still has a
session); it is constant-true in every walked path, and the wire carries the decision as the
choice of code, which is scenario selection — a slice-level concern. The same test hung over four
more sites until it closed 2026-08-09 — ruled: "Don't invent a new test — the view's own
definition already decides it" — and all four fell with it: `DataPrompt.awaiting_content`,
`MessageQueued.accepted`, `MessageQueued.queue_id` and `SessionClosing.cause`. The notes at
steps 11, 14 and 16 record the grounds.*

| 🟦 C · Step 3 | `Helo` |
|:--|:--|
| MTA Client | ⬛ `HELO` bar.com |
| | 🟦 **Helo**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com |
| Event | 🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com |
| Given | |
| | 🟧 **ConnectionAccepted** |

*Ruled 2026-08-08: `protocol` leaves the command and the event. Unlike `session_id` and
`queue_id`, the word has RFC grounding — `Protocol` is the `WITH` clause's own grammar term, and
SMTP its registered value, decided by which hello verb arrived — but in a `HELO`-only charter it
cannot vary, and a constant is a rule, not a fact. The `WITH` value joins the renderer's
constants. If the charter ever admits `EHLO`, the field re-enters as a fact originated by the
verb that actually arrived.*

| 🟩 V · Step 4 | `SessionReady` |
|:--|:--|
| MTA Client | ⬛ `250` foo.com |
| | 🟩 **SessionReady**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com |
| Given | |
| | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>🟧 **ClientIdentified** |

*Renamed and reduced, ruled 2026-08-08. This view spent the walk's whole history as
`SessionState` — a name belonging to the model's consulted precondition view, which this walk
never consults: every command's `Given` cites events directly. The only job performed here is
rendering `250` foo.com, and §4.1.1.1 names both halves of it — the reply confirms *"the initial
state"*, carried by the code, renderer territory; and *"The SMTP server identifies itself to the
SMTP client in the connection greeting reply and in the response to this command"*, carried by
the data. So: `SessionReady`, the `220`'s sibling — service ready, then session ready — with a
one-field dataset, `server_domain` (which had joined earlier the same day when the trial's open
choice closed). `identified` and `transaction_open` leave by the `accepting` precedent, booleans
nothing renders; `ConnectionAccepted` leaves the `Given` with them, having only ever been there
for their fold. `ClientIdentified` is the occasion, `ServiceConfigured` the data.*

| 🟦 C · Step 5 | `MailFrom` |
|:--|:--|
| MTA Client | ⬛ `MAIL FROM:`\<Smith@bar.com> |
| | 🟦 **MailFrom**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com> |
| Event | 🟧 **ReversePathDeclared**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com> |
| Given | |
| | 🟧 **ClientIdentified** |

*Renamed, adopted 2026-08-08: the event is the client's declaration, not our state change. The
candidates were screened in [`EXPLORE-declaration-vs-status.md`](EXPLORE-declaration-vs-status.md)
— `Declared` over `Specified` and `Accepted` — and that file stays the citation while its prefix
holds (rule 13). The old name's register, `MailTransactionStarted`, is §3.3's own; where that
altitude belongs is part of what the exploration leaves open.*

| 🟩 V · Step 6 | `TransactionState` |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **TransactionState** |
| Given | |
| | 🟧 **ReversePathDeclared** |

*Emptied and collapsed, adopted 2026-08-08 with the rename. The dataset restated the `Given`
(`open`, `reverse_path`) and encoded position as data (`recipient_count`, 0 against 1 — the
`session_id` finding again); the `250` OK carries no varying value, so the view renders from
existence, the `354`'s own shape. Empty here is not empty forever: §3.3 gives this occasion a
550/553 branch, and the rejection walk seats whatever selects the reply code at the slice layer.
The method repo parked exactly this question here — "whose dataset is empty, a question parked
with the walk's step 6" —
[`state-view-todo-list-decision-model.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/state-view-todo-list-decision-model.md),
this repo's first citation of it, discharged by this note. The view's own name is still open in
the exploration: `TransactionState` describes state it no longer carries.*

| 🟩 V · Step 7 | `RecipientDirectory` &nbsp;*(consulted)* |
|:--|:--|
| | 🟩 **RecipientDirectory**&#10;<br>&nbsp;&nbsp;`is_local`: true |
| Given | |
| | 🟧 **RecipientResolved**&#10;<br>&nbsp;&nbsp;`is_local`: true&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

*No top row — consulted. Nothing is drawn to any actor; `RcptTo` declares this dependency in its
own `Given`.*

| 🟦 C · Step 8 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Given | |
| | 🟧 **ReversePathDeclared**&#10;<br>🟧 **RecipientResolved**&#10;<br>&nbsp;&nbsp;`is_local`: true&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

Traversed **once**. No `RecipientRejected` anywhere in this walk.

| 🟩 V · Step 9 | `TransactionState` &nbsp;*(second traversal)* |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **TransactionState** |
| Given | |
| | 🟧 **RecipientAccepted** |

*Collapsed with step 6, same adoption: the traversals are distinguished by which event occasioned
them, not by field values. `ReversePathDeclared` leaves this `Given` entirely, its citation having
existed only to feed the dataset
([`EXPLORE-declaration-vs-status.md`](EXPLORE-declaration-vs-status.md)).*

| 🟦 C · Step 10 | `BeginData` |
|:--|:--|
| MTA Client | ⬛ `DATA` |
| | 🟦 **BeginData** |
| Event | 🟧 **DataPhaseEntered**&#10;<br>&nbsp;&nbsp;*no payload* |
| Given | |
| | 🟧 **ReversePathDeclared**&#10;<br>🟧 **RecipientAccepted** |

| 🟩 V · Step 11 | `DataPrompt` |
|:--|:--|
| MTA Client | ⬛ `354` Start mail input; end with `<CRLF>.<CRLF>` |
| | 🟩 **DataPrompt** |
| Given | |
| | 🟧 **DataPhaseEntered** |

*Emptied under the step 16 ruling — `awaiting_content` was a constant-true boolean on
`accepting`'s footing; the `354` renders from `DataPhaseEntered`'s existence, as the backward
table has said since the collapse.*

| 🟦 C · Step 12 | `SubmitContent` |
|:--|:--|
| MTA Client | ⬛ `Date:` Tue, 19 May 1998 09:14:02 -0700&#10;<br>`From:` Smith \<Smith@bar.com>&#10;<br>`To:` Jones@foo.com&#10;<br>`Subject:` Tuesday&#10;<br>(blank)&#10;<br>Blah blah blah...&#10;<br>`.` |
| | 🟦 **SubmitContent**&#10;<br>&nbsp;&nbsp;`content`: 126 octets, dot-unstuffed |
| Event | 🟧 **MessageAccepted**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipients`: [\<Jones@foo.com>]&#10;<br>&nbsp;&nbsp;`content_ref`: blob:sha256:9c1e…&#10;<br>&nbsp;&nbsp;`actual_octets`: 126&#10;<br>&nbsp;&nbsp;`received_at`: 1998-05-19T09:14:07-07:00 |
| Given | |
| | 🟧 **DataPhaseEntered**&#10;<br>🟧 **ReversePathDeclared**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

*The `Given` grew 2026-08-08, seated by the payload check — completeness's third. `reverse_path`
and `recipients` fold in from the two events now cited; `queue_id` is minted here, the walk's
only mint; `content_ref` and `actual_octets` derive from `content`; `received_at` is the
receiver's clock, a boundary fact like the transport's addresses. The hole was found twice
independently in one day — a docs-review session asked where an emitted event's payload values
come from, and the steps 6/9 collapse orphaned `reverse_path` because this step's fold was
undeclared.*

⚠️ **The count read 194 until 2026-08-08, and 194 was never the message's length.** The number was
asserted at the walk's creation and never recomputed against the message shown. The six content
lines above, each CRLF-terminated, terminating dot line excluded, total **126** octets —
dot-unstuffing is a no-op here, since no line begins with a dot. Rule 8's point in miniature: a
value posing as real data hid in the one field the walk itself declares derived from `content`.

| 🟩 V · Step 13 | `MessageTrace` &nbsp;🟥 **H6** |
|:--|:--|
| Stored message | `Received: from` bar.com ([203.0.113.20])&#10;<br>&nbsp;&nbsp;`by` foo.com `with` SMTP&#10;<br>&nbsp;&nbsp;`id` f2C8D14&#10;<br>&nbsp;&nbsp;`for` \<Jones@foo.com>;&#10;<br>&nbsp;&nbsp;Tue, 19 May 1998 09:14:07 -0700 |
| | 🟩 **MessageTrace**&#10;<br>&nbsp;&nbsp;`from_domain`: bar.com&#10;<br>&nbsp;&nbsp;`address_literal`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`by`: foo.com&#10;<br>&nbsp;&nbsp;`id`: f2C8D14&#10;<br>&nbsp;&nbsp;`for`: \<Jones@foo.com>&#10;<br>&nbsp;&nbsp;`at`: 1998-05-19T09:14:07-07:00 |
| Given | |
| | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com>&#10;<br>🟧 **MessageAccepted**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`received_at`: 1998-05-19T09:14:07-07:00 |

The eighth output — §4.4's MUST, fired at receipt, drawn into the **stored message** rather than
the socket, which is why its top row carries no wire chip. Its `Given` is the walk's deepest —
four walked events and a seeded one, each field a value the header renders. **🟥 H6 bites here**: the `by` clause renders from `ServiceConfigured.server_domain`
(the config arm); on the multi-homed arm it would render from `ConnectionAccepted.local_address`
instead — one of that field's two candidate consumers; H6 under *Hotspots* names both.

| 🟩 V · Step 14 | `MessageQueued` |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **MessageQueued** |
| Given | |
| | 🟧 **MessageAccepted** |

*Emptied under the step 16 ruling. `accepted` was a constant-true boolean on `accepting`'s own
footing. `queue_id` needed the ruling: it is no constant, but the `250` OK's task — telling the
client the handoff occurred — consumes no id, and position suffices for which message (at most
one transaction per session, H4). The constitutive weight §2.1 gives this reply lives in its
issuance, which is the renderer's territory, not the dataset's. The value keeps its origin and
its step 13 destination, so nothing orphans. One product option recorded, not held for: real
MTAs often echo the id in the reply text — `250` Ok: queued as F2C8D14 — and reply text is
operator territory like `greeting_text`, so if FnEmail ever chooses that text the field
re-enters with a real consumer, exactly as `protocol` re-enters under `EHLO`.*

**The responsibility boundary is here — at the issuance of this `250`.** Left of it, abandoning
costs nothing. §2.1 places the handoff on the reply itself: *"once the server has issued a
success response at the end of the mail data, a formal handoff of responsibility for the message
occurs: the protocol requires that a server MUST accept responsibility for either delivering the
message or properly reporting the failure to do so"*. §6.1 says the same from the server's side —
accepting a piece of mail *is* the sending of the `250` in response to `DATA`. `MessageAccepted`
records the decision; issuing the reply is what makes the handoff formal.

⚠️ **This note originally placed the boundary at the event, not the reply.** It read: "at
`MessageAccepted` we have accepted responsibility for delivering or reporting failure — RFC 5321
§2.1. The `250` is the moment the client learns that." That demotes the reply to a notification,
and §2.1 makes it constitutive — the handoff occurs when the success response is issued. The
failure case is concrete: the server writes `MessageAccepted` and crashes before the `250`
leaves; under §2.1 no handoff occurred and the client legitimately retries, while the old
placement had this server owning a message the client still owned. Corrected 2026-08-08.

| 🟦 C · Step 15 | `Quit` |
|:--|:--|
| MTA Client | ⬛ `QUIT` |
| | 🟦 **Quit** |
| Event | 🟧 **SessionClosed**&#10;<br>&nbsp;&nbsp;`cause`: quit |
| Given | |
| | 🟧 **ConnectionAccepted** |

| 🟩 V · Step 16 | `SessionClosing` &nbsp;🟥 **H8** |
|:--|:--|
| MTA Client | ⬛ `221` foo.com Service closing transmission channel |
| | 🟩 **SessionClosing**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com |
| Given | |
| | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>🟧 **SessionClosed** |

*Ruled 2026-08-09: "Don't invent a new test — the view's own definition already decides it." A
view is the dataset provided to the actor to complete its task; the `221`'s task consumes
`server_domain` and nothing else (§4.2.2 varies only the domain), so `cause` was never part of
this dataset. Render-failure for a non-constant field removes it from the view but never from
its originating event — the fact survives on `SessionClosed`, where its unconsumed-ness is
flagged (🟥 H8). Its consumers wait on the unwalked closure branches, where `cause` selects
which closure renders at all — scenario selection, the slice layer's: the `session_id` arc
again, a value dying on the path to re-materialize at the slice as a selection contract.*

Every reply is 2xx or 3xx. No error branch is taken anywhere.

---

## Accounting

**16 steps over 15 distinct slices** — `TransactionState` is traversed twice. Two seeded events,
never walked.

| | Original walk | This walk |
|:--|:--|:--|
| Steps | 10 | **16** |
| Command steps | 7 | 7 |
| View steps | 3 | **9** |
| Replies shown as wire text inside a command | 7 | **0** |
| Outputs with a step behind them | 0 of 8 | **8 of 8** |
| Events seeded by the preamble | — | 2 |
| Model slices touched (of 12) | 10 | **9** — `Reset`, `SessionTranscript`, and now `SessionState`: no `Given` here ever consults it |
| Slices this path defines beyond the model | 0 | 6 views + 2 seeded events |

---

## Completeness, instantiated

### Backward — every output to its origin

| Output | Drawn by | Origin of every varying value |
|:--|:--|:--|
| `220` foo.com Simple Mail Transfer Service Ready | `ServiceReady` (step 2) | `ServiceConfigured.server_domain`, `.greeting_text` — both carried by the view; the `220` code itself is this scenario's rendering |
| `250` foo.com | `SessionReady` (step 4) | `ServiceConfigured.server_domain`, carried by the view; the `250` code is this scenario's rendering |
| `250` OK | `TransactionState` (step 6) | `ReversePathDeclared`'s existence |
| `250` OK | `TransactionState` (step 9) | `RecipientAccepted`'s existence |
| `354` prompt | `DataPrompt` (step 11) | `DataPhaseEntered`'s existence |
| `Received:` trace line | `MessageTrace` (step 13) | four walked events and the seeded `server_domain` — see the step |
| `250` OK | `MessageQueued` (step 14) | `MessageAccepted`'s existence |
| `221` foo.com Service closing transmission channel | `SessionClosing` (step 16) | `ServiceConfigured.server_domain`, carried by the view; `SessionClosed`'s existence |

**Every varying output value has an origin on the page** — in the original walk, zero of these
eight had a step behind them. The reply texts that carry no field — the two `250`s' OK, the
`354`'s and the `221`'s — are the RFC's own example texts (§4.2.2; OK is Appendix D's own
abbreviation), adopted verbatim as implementation constants — a constant is a rule, not a fact,
so none needs an origin. The two `250`s joined 2026-08-08, their datasets emptied by the steps 6
and 9 collapse. The trace's `with` SMTP joined them
2026-08-08: in a `HELO`-only charter the value cannot vary. The greeting's text is configuration
on purpose: §3.1 invites the operator to extend it with software and version, which is why
`greeting_text` is seeded and those are not.

### Payloads — every emitted value to its origin

The third check, added 2026-08-08. The backward check stops at rendered outputs and the forward
check counts consumers, so neither asks where an **emitted** event's payload values come from —
found twice independently in one day: the docs-review session named the gap, and the steps 6/9
collapse fell into it when `reverse_path` lost its last apparent consumer. An origin is a command
field, a `Given` fold, a derivation from either, a mint at the step, or a named boundary fact.
Seeded events are declared, not emitted — the preamble already rules their provenance out of
scope.

| Event | Value | Origin |
|:--|:--|:--|
| `ConnectionAccepted` | `peer_address` · `local_address` | `AcceptConnection` — transport-supplied, per step 1's note |
| `ClientIdentified` | `claimed_domain` | `Helo.claimed_domain` |
| `ReversePathDeclared` | `reverse_path` | `MailFrom.reverse_path` |
| `RecipientAccepted` | `forward_path` | `RcptTo.forward_path` |
| `DataPhaseEntered` | *no payload* | — |
| `MessageAccepted` | `reverse_path` · `recipients` | `Given` folds — `ReversePathDeclared.reverse_path`, `RecipientAccepted.forward_path` |
| | `content_ref` · `actual_octets` | derived from `SubmitContent.content` |
| | `queue_id` | minted at this step — the walk's only mint |
| | `received_at` | the receiver's clock — a boundary fact, like the transport's addresses |
| `SessionClosed` | `cause`: quit | the arriving verb — `cause` records which way the session bracket closed |

Every emitted value has an origin on the page. The check's first run is what seated step 12's
`Given`: before it, `reverse_path` and `recipients` materialized from a `Given` citing only the
payload-free `DataPhaseEntered`.

### Forward — every event to its consumers

| Event | Consumed at |
|:--|:--|
| `ServiceConfigured` *(seeded)* | steps 2, 4, 13, 16 (`server_domain`; step 2 also `greeting_text`) |
| `RecipientResolved` *(seeded)* | steps 7, 8 (`is_local`, `forward_path`) |
| `ConnectionAccepted` | steps 2, 3, 15 (existence); step 13 (`peer_address`) — `local_address` has **no consumer**, 🟥 H6 |
| `ClientIdentified` | steps 4, 5 (existence); step 13 (`claimed_domain`) |
| `ReversePathDeclared` | steps 6, 8, 10 (existence); step 12 (`reverse_path`) |
| `RecipientAccepted` | steps 9, 10 (existence); steps 12, 13 (`forward_path`) |
| `DataPhaseEntered` | steps 11, 12 |
| `MessageAccepted` | step 13 (`queue_id`, `received_at`); step 14 (existence) — `reverse_path`, `recipients`, `content_ref`, `actual_octets` are consumed by delivery, right of the responsibility boundary, out of scope |
| `SessionClosed` | step 16 (existence) — `cause` has **no consumer on this walk**, 🟥 H8 |

Every event has at least one consumer inside the walk. Two fields are unconsumed, each flagged
where it bites: `ConnectionAccepted.local_address` (🟥 H6) and `SessionClosed.cause` (🟥 H8) —
the second is the cascade ruling's accepted cost, its consumers waiting on the closure branches.
The steps 6/9 collapse had briefly made `ReversePathDeclared.reverse_path` another: its
consumer-in-fact was step 12's emitted `MessageAccepted.reverse_path`, which no `Given` declared.
The payload check seated that `Given` — the check working, not the finding patched — and the arc
is recorded in [`EXPLORE-declaration-vs-status.md`](EXPLORE-declaration-vs-status.md).

### The `Received:` header, clause by clause

| Clause | Value | Source |
|:--|:--|:--|
| `FROM` domain | bar.com | `ClientIdentified.claimed_domain` |
| address literal | 203.0.113.20 | `ConnectionAccepted.peer_address` |
| `BY` | foo.com | `ServiceConfigured.server_domain` — **the config arm; 🟥 H6 open** |
| `WITH` | SMTP | renderer constant — `HELO` is the only hello in this charter |
| `ID` | f2C8D14 | `MessageAccepted.queue_id` |
| `FOR` | \<Jones@foo.com> | `RecipientAccepted.forward_path` — emitted, because exactly one |
| timestamp | 1998-05-19T09:14:07-07:00 | `MessageAccepted.received_at` |

Unchanged from the original except `BY` and `WITH`. `BY` the original sourced from unmodeled
config and this walk sources from the seeded event — same fact, now with an origin the
completeness check can see. `WITH` the original sourced from `ClientIdentified.protocol`, and the
ruling that removed `protocol` (commit 2673b1b, recorded at step 3) made it a renderer constant.

---

## Against the existing walks

**The bytes are identical.** Same scene, same dialogue, same reply codes, same message, same
`Received:` header values as the original ten-step walk. What changed is what the page admits to:

- Seven replies and one trace line now each have a step; the original had all seven inside command
  wire rows and the trace only in an accounting table.
- The preamble replaces two silences: configuration that nothing declared, and a translation the
  original marked yellow mid-walk. Both are seeded events now, cited 🟧 where consumed.
- `Given` rows are minimal and carry fields exactly where a value, not just existence, is used —
  steps 2, 4, 7, 8, 12, 13 and 16; existence alone suffices at steps 3, 5, 6, 9, 10, 11, 14
  and 15.
- The `Where` rows and their session key lived and died inside this file's own history:
  instantiated as a real ULID on 2026-08-07, removed on 2026-08-08 once instantiation exposed
  them as slice-level apparatus — the block under *The layering* records the arc. The
  explorations' placeholder 01J8Z… had been called a rule 8 break in
  [`EXPLORE-view-slice.md`](EXPLORE-view-slice.md); the fix turned out to be deletion, not a
  value.
- Nothing this file asserts contradicts the original's findings; the original's *What this walk
  tested* items (field independence, the `FOR` clause rule, happy-for-every-actor) all still hold
  and are not restated.

---

## What this walk tested

1. **Completeness closes in three checks on one page.** Backward: eight outputs, every varying
   value with an origin. Payloads: every emitted value with an origin — the check added
   2026-08-08, after two sessions independently caught step 12 emitting values from nowhere.
   Forward: every event consumed, one field flagged. None of the three was checkable inside the
   original form.
2. **The preamble carries a whole path.** Two seeded events satisfy every `Given` no walked step
   could — including the trace header's, which needs four walked events *and* a seeded one.
3. **Three scopes are visible without a single key on the page** — `ServiceConfigured` folds in
   from service lifetime, the session is the positional bracket from `ConnectionAccepted` to
   `SessionClosed`, and the message is `queue_id`, payload born at `MessageAccepted`. Evidence for
   H4, found by walking rather than argued — and passed upward, where it belongs.

## What it did not test

- `Reset` — still uncovered by any path. D.2 remains the only thing that will close it.
- `SessionTranscript` — no operator in this scene.
- **Any error branch** — no 4xx, no 5xx, no sequencing error. H2 stays untouched.
- **Received-header stacking** — the relay path's job, not this one's.

---

## Hotspots

Only what is open or new. H3 is resolved and therefore absent. H5's classification is settled but
its residue is H7's decision — the two close together, so H5 carries no entry of its own here.
The six path-defined views and two seeded events carry no markers, because *paths are the source*
— they are this path's contribution, not doubts about it.

**🟥 H6 — open, and it bites at step 13.** Is `Received:`'s `BY` sourced from configuration or
from `local_address`? This walk renders the config arm. `local_address` has two candidate
consumers — the multi-homed `BY` arm at step 13, and the nameless-server greeting, since the `220`
identity slot admits an address literal (§4.2). This walk exercises neither, so the field stays
its one unconsumed value.

**🟥 H4 — open, and it is a store-level ruling; this path's evidence passes upward.** A stream
is the unit of ordering, of atomic invariants, and of a fold's scope — concerns of the layer that
stores many conversations at once, which is exactly why no stream machinery survives on these
pages. What the walk hands the ruling:

1. **Three scopes in one conversation, none needing a key on the page**: service lifetime
   (`ServiceConfigured`, folded in from before the timeline), the session (the positional bracket
   from `ConnectionAccepted` to `SessionClosed`), and the message (`queue_id`, payload born at
   `MessageAccepted` — the only identity that outlives the session).
2. **No transaction identifier exists anywhere in the model.** At most one transaction is open
   per session, so position suffices even across two traversals of `TransactionState`;
   per-transaction streams would mint an id no step consumes.
3. **§4.3.1 serializes each session.** *"Unless other arrangements are negotiated through
   service extensions, the sender MUST wait for this response before sending further commands"*
   — and for the greeting, the walk's own step 2, the sender only *"SHOULD wait"*. This charter
   negotiates no extensions, so inside one session there is never write contention, and
   stream-splitting buys no concurrency inbound.
4. **The responsibility boundary is the natural stream boundary**: step 13 is the walk's only
   message-scoped element, and delivery must read the message long after the connection is gone.

⚠️ **Evidence item 3 originally read "the client MUST wait for every reply."** That overreached
twice: the greeting reply is a SHOULD, not a MUST — and the greeting is this walk's own step 2 —
and even the command-reply MUST is conditioned on no service extensions being negotiated, which
the flat paraphrase erased. The serialization conclusion survives restated, because this charter
negotiates no extensions; the citation as written did not. Corrected 2026-08-08.

Pending at model level: **(i)** a per-session stream or envelope metadata — either way the key
lives above the path; **(ii)** transaction as stream or as phase, for which the
aborted-transaction walk (D.2, `Reset`) is the test; **(iii)** the message as its own stream from
`MessageAccepted` onward.

**And on names: `queue_id` is manufactured, like `session_id` was.** Verified 2026-08-08: queue
appears in the RFC only as retry behavior, never as an identifier; the value f2C8D14 appears
nowhere in it; and even the `ID` clause the value feeds is optional —
`Opt-info = [Via] [With] [ID] [For] [Additional-Registered-Clauses]` (§4.4; the RFC wraps the
production across two lines). What keeps `queue_id` in the walk when `session_id`
fell is the completeness check, not the specification: this server chooses to emit the optional
clause, and once emitted the value has an origin and a destination on the page. A chosen
destination, not a mandated one — the same footing as `peer_address`, whose clause is
MUST-supplied but whose contents are SHOULD.

**Ruled 2026-08-08: this path carries no `correlation_id`.** No step consumes such a value, so
under this path's own discipline it does not appear — *needed* is the only claim that seats a
value here. The model-level question is flagged upward rather than settled: the model's metadata
table carries `session_id`, `correlation_id` and `queue_id` — three correlation schemes against
its own warning that the RFC's `ID` clause and `queue_id` *"are one mechanism, not two"* — and
with session = connection verified 1:1, `correlation_id` duplicates `session_id` inside this
scope while `queue_id` supersedes it beyond. Something to deal with at a higher level, not here.
`session_id` followed the same day — no key survives on a path; the block under *The layering*
records it.

**🟥 H8 — open, and the cascade ruling sharpened it.** Registered in the model from
`model-altitude.md` Q6 — does `SessionClosed` earn its place? The event is consumed: step 16
folds its existence for the `221`. But `cause` now has no consumer on this walk — the 2026-08-09
ruling removed it from `SessionClosing`'s dataset, and its consumers-in-waiting are the closure
branches this walk does not take, the `421` shutdown and the timeout close, where `cause` selects
which closure renders at all. Scenario selection is the slice layer's; the D.2 and error walks
are where H8 closes.

**🟥 H7 — open, product.** Does FnEmail ever refuse mail entirely (RFC 7504 `521`)? It decides
whether `ServiceReady` ever renders anything but `220` at connection time beyond the `554`
blocklist case the model already carries.

**🟥 New — the refused greeting still has a session.** Found by the backward walk and verified
against the RFC: a `554` given at connection opening obliges the server to *"still wait for the
client to send a QUIT"* (§3.1, a MUST) — so a refused session still runs, and `ConnectionAccepted`
must exist on that path. That contradicts the model's recorded contract for `AcceptConnection`,
*reply 554, no event*. A rule 4 correction is pending in the model; unnumbered here because
hotspot numbering belongs to the model.

**✅ H1 — verified answered by this walk.** *Does `DataPhaseEntered` earn its place?* On this page
it has two consumers: `DataPrompt` folds it to render the `354`, and `SubmitContent` declares it
as `Given`. The event is used. Formal closure is model work under rule 9 — this path supplies the
evidence, not the edit.
