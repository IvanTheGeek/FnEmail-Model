# Path — HELO, direct, single recipient, clean · v2

The same scenario as [`helo-direct-single-recipient.md`](helo-direct-single-recipient.md), walked
under the semantics settled on 2026-08-07. **This file is the result; the reasoning is in
[`EXPLORE-rewalk-cadence.md`](EXPLORE-rewalk-cadence.md)** and is not repeated here.

Derived from **RFC 5321 Appendix D.1** — *A Typical SMTP Transaction Scenario* — reduced to one
recipient, with `HELO` substituted for `EHLO`. Verified against
[`../rfc/rfc5321.txt`](../rfc/rfc5321.txt).

⚠️ **The dialogue is ours, the topology is the RFC's.** D.1 sends to three recipients and rejects
one; no appendix scenario is *both* direct and single-recipient. This walk is RFC-derived in
structure and RFC-conformant in every reply code, but the exchange as written does not appear in
the specification.

**The form, in one paragraph.** A path opens with a **Required first** block — the path-level
`Given`, declaring every event the walk needs but does not walk, with the fields its steps consume.
🟨 appears only there, and makes exactly one claim: *needed, not walked here* — the path is
indifferent to provenance. Steps cite every event 🟧, walked or seeded. A server reply is a
**rendered view**; a view read only internally is **consulted** and has no top row; a step whose
actor is outside the model has no top row either. A step is one table, no heading. Dependencies
point backward only.

**No *not in the model* markers.** Paths are the source and slices are derived — what a path
needs, it defines, and the union across paths populates the model
([`../HANDOFF.md`](../HANDOFF.md) §1). The five views and two seeded events this walk defines
beyond the current model are simply part of this path; reconciling the model is model work.
Hotspots appear only where open — see *Hotspots* at the end.

*(Steps intentionally do not follow [`STEP-FORM.md`](STEP-FORM.md), which predates these semantics
and will catch up.)*

---

## Scene

| | |
|:--|:--|
| Server (us) | foo.com at 192.0.2.10:25 |
| Client | bar.com at 203.0.113.20 — **the originating host, contacting us directly** |
| Sender | \<Smith@bar.com> — **same domain as the client** |
| Recipient | \<Jones@foo.com> — one, local |
| Time | Tue, 19 May 1998 09:14:07 -0700 |
| Session | `session_id`: 00T22SDG4R9FQ3ZK7VWX2M5N8P — a ULID whose first ten characters encode the scene's instant |

**The alignment is the point.** `claimed_domain` and the `reverse_path` domain are both `bar.com`,
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
| Given | 🟤 |

*No top row — the actor is the transport, outside the model. `peer_address` is "determined from
the TCP connection" (§4.4); `local_address` is the listening socket's, and H6's question.*

| 🟩 V · Step 2 | `ServiceReady` |
|:--|:--|
| MTA Client | ⬛ `220` foo.com Simple Mail Transfer Service Ready |
| | 🟩 **ServiceReady**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`accepting`: true |
| Given | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`greeting_text`: Simple Mail Transfer Service Ready&#10;<br>🟧 **ConnectionAccepted** |
| Where | `session_id` = 00T22SDG4R9FQ3ZK7VWX2M5N8P |

*The `Where` key appears in no `Given` payload, deliberately. The RFC names no session identifier
anywhere — while a session lives, the TCP channel is its identity — so `session_id` is ours:
minted at `AcceptConnection` (the ULID's first ten characters encode that instant) and carried by
every later event either as envelope metadata or as the name of the stream they append to. Which
of those two is H4's question — see* Hotspots. *A payload home is ruled out by the completeness
check itself: no output the server ever emits carries a session id, so the field could never have
a destination, and a fact needs both.*

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
| Where | `session_id` = 00T22SDG4R9FQ3ZK7VWX2M5N8P |

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
| Given | 🟧 **MailTransactionStarted**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com> |
| Where | `session_id` = 00T22SDG4R9FQ3ZK7VWX2M5N8P |

| 🟩 V · Step 7 | `RecipientDirectory` &nbsp;*(consulted)* |
|:--|:--|
| | 🟩 **RecipientDirectory**&#10;<br>&nbsp;&nbsp;`is_local`: true |
| Given | 🟧 **RecipientResolved**&#10;<br>&nbsp;&nbsp;`is_local`: true&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Where | `forward_path` = \<Jones@foo.com> |

*No top row — consulted. Nothing is drawn to any actor; `RcptTo` declares this dependency in its
own `Given`.*

| 🟦 C · Step 8 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Given | 🟧 **MailTransactionStarted**&#10;<br>🟧 **RecipientResolved**&#10;<br>&nbsp;&nbsp;`is_local`: true&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |

Traversed **once**. No `RecipientRejected` anywhere in this walk.

| 🟩 V · Step 9 | `TransactionState` &nbsp;*(second traversal)* |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **TransactionState**&#10;<br>&nbsp;&nbsp;`open`: true&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipient_count`: 1 |
| Given | 🟧 **MailTransactionStarted**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>🟧 **RecipientAccepted** |
| Where | `session_id` = 00T22SDG4R9FQ3ZK7VWX2M5N8P |

| 🟦 C · Step 10 | `BeginData` |
|:--|:--|
| MTA Client | ⬛ `DATA` |
| | 🟦 **BeginData** |
| Event | 🟧 **DataPhaseEntered**&#10;<br>&nbsp;&nbsp;*no payload* |
| Given | 🟧 **MailTransactionStarted**&#10;<br>🟧 **RecipientAccepted** |

| 🟩 V · Step 11 | `DataPrompt` |
|:--|:--|
| MTA Client | ⬛ `354` Start mail input; end with `<CRLF>.<CRLF>` |
| | 🟩 **DataPrompt**&#10;<br>&nbsp;&nbsp;`awaiting_content`: true |
| Given | 🟧 **DataPhaseEntered** |
| Where | `session_id` = 00T22SDG4R9FQ3ZK7VWX2M5N8P |

| 🟦 C · Step 12 | `SubmitContent` |
|:--|:--|
| MTA Client | ⬛ `Date:` Tue, 19 May 1998 09:14:02 -0700&#10;<br>`From:` Smith \<Smith@bar.com>&#10;<br>`To:` Jones@foo.com&#10;<br>`Subject:` Tuesday&#10;<br>(blank)&#10;<br>Blah blah blah...&#10;<br>`.` |
| | 🟦 **SubmitContent**&#10;<br>&nbsp;&nbsp;`content`: 194 octets, dot-unstuffed |
| Event | 🟧 **MessageAccepted**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipients`: [\<Jones@foo.com>]&#10;<br>&nbsp;&nbsp;`content_ref`: blob:sha256:9c1e…&#10;<br>&nbsp;&nbsp;`actual_octets`: 194&#10;<br>&nbsp;&nbsp;`received_at`: 1998-05-19T09:14:07-07:00 |
| Given | 🟧 **DataPhaseEntered** |

| 🟩 V · Step 13 | `MessageTrace` &nbsp;🟥 **H6** |
|:--|:--|
| Stored message | `Received: from` bar.com ([203.0.113.20])&#10;<br>&nbsp;&nbsp;`by` foo.com `with` SMTP&#10;<br>&nbsp;&nbsp;`id` f2C8D14&#10;<br>&nbsp;&nbsp;`for` \<Jones@foo.com>;&#10;<br>&nbsp;&nbsp;Tue, 19 May 1998 09:14:07 -0700 |
| | 🟩 **MessageTrace**&#10;<br>&nbsp;&nbsp;`from_domain`: bar.com&#10;<br>&nbsp;&nbsp;`address_literal`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`by`: foo.com&#10;<br>&nbsp;&nbsp;`with`: SMTP&#10;<br>&nbsp;&nbsp;`id`: f2C8D14&#10;<br>&nbsp;&nbsp;`for`: \<Jones@foo.com>&#10;<br>&nbsp;&nbsp;`at`: 1998-05-19T09:14:07-07:00 |
| Given | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP&#10;<br>🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com>&#10;<br>🟧 **MessageAccepted**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`received_at`: 1998-05-19T09:14:07-07:00 |
| Where | `queue_id` = f2C8D14 |

The eighth output — §4.4's MUST, fired at receipt, drawn into the **stored message** rather than
the socket, which is why its top row carries no wire chip. Its `Given` is the walk's deepest —
four walked events and a seeded one, each field a value the header renders. **🟥 H6 bites here**: the `by` clause renders from `ServiceConfigured.server_domain`
(the config arm); on the multi-homed arm it would render from `ConnectionAccepted.local_address`
instead — which is that field's only candidate consumer in this walk.

| 🟩 V · Step 14 | `MessageQueued` |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **MessageQueued**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`accepted`: true |
| Given | 🟧 **MessageAccepted**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14 |
| Where | `session_id` = 00T22SDG4R9FQ3ZK7VWX2M5N8P |

**The responsibility boundary is here.** Left of it, abandoning costs nothing; at `MessageAccepted`
we have accepted responsibility for delivering or reporting failure — RFC 5321 §2.1. **The `250` is
the moment the client learns that.**

| 🟦 C · Step 15 | `Quit` |
|:--|:--|
| MTA Client | ⬛ `QUIT` |
| | 🟦 **Quit** |
| Event | 🟧 **SessionClosed**&#10;<br>&nbsp;&nbsp;`cause`: quit |
| Given | 🟧 **ConnectionAccepted** |

| 🟩 V · Step 16 | `SessionClosing` |
|:--|:--|
| MTA Client | ⬛ `221` foo.com Service closing transmission channel |
| | 🟩 **SessionClosing**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`cause`: quit |
| Given | 🟧 **ServiceConfigured**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>🟧 **SessionClosed**&#10;<br>&nbsp;&nbsp;`cause`: quit |
| Where | `session_id` = 00T22SDG4R9FQ3ZK7VWX2M5N8P |

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
| Model slices touched (of 12) | 10 | 10 — `Reset` and `SessionTranscript` still untouched |
| Slices this path defines beyond the model | 0 | 5 views + 2 seeded events |

---

## Completeness, instantiated

### Backward — every output to its origin

| Output | Drawn by | Origin of every varying value |
|:--|:--|:--|
| `220` foo.com Simple Mail Transfer Service Ready | `ServiceReady` (step 2) | `ServiceConfigured.server_domain`, `.greeting_text`; `accepting` folds from `ConnectionAccepted`'s existence |
| `250` foo.com | `SessionState` (step 4) | `ServiceConfigured.server_domain`; booleans fold from `ConnectionAccepted`, `ClientIdentified` |
| `250` OK | `TransactionState` (step 6) | `MailTransactionStarted.reverse_path` |
| `250` OK | `TransactionState` (step 9) | `MailTransactionStarted`, `RecipientAccepted` |
| `354` prompt | `DataPrompt` (step 11) | `DataPhaseEntered`'s existence |
| `Received:` trace line | `MessageTrace` (step 13) | four walked events and the seeded `server_domain` — see the step |
| `250` OK | `MessageQueued` (step 14) | `MessageAccepted.queue_id` |
| `221` foo.com Service closing transmission channel | `SessionClosing` (step 16) | `ServiceConfigured.server_domain`; `SessionClosed.cause` |

**Every varying output value has an origin on the page** — in the original walk, zero of these
eight had a step behind them. The two trailing reply texts that carry no field, `354`'s and
`221`'s, are the RFC's own example texts (§4.2.2), adopted verbatim as implementation constants —
a constant is a rule, not a fact, so neither needs an origin. The greeting's text is configuration
on purpose: §3.1 invites the operator to extend it with software and version, which is why
`greeting_text` is seeded and those two are not.

### Forward — every event to its consumers

| Event | Consumed at |
|:--|:--|
| `ServiceConfigured` *(seeded)* | steps 2, 4, 13, 16 (`server_domain`; step 2 also `greeting_text`) |
| `RecipientResolved` *(seeded)* | steps 7, 8 (`is_local`, `forward_path`) |
| `ConnectionAccepted` | steps 2, 3, 4, 15 (existence); step 13 (`peer_address`) — `local_address` has **no consumer**, 🟥 H6 |
| `ClientIdentified` | steps 4, 5 (existence); step 13 (`claimed_domain`, `protocol`) |
| `MailTransactionStarted` | steps 8, 10 (existence); steps 6, 9 (`reverse_path`) |
| `RecipientAccepted` | steps 9, 10 (existence); step 13 (`forward_path`) |
| `DataPhaseEntered` | steps 11, 12 |
| `MessageAccepted` | steps 13, 14 (`queue_id`; step 13 also `received_at`) — `reverse_path`, `recipients`, `content_ref`, `actual_octets` are consumed by delivery, right of the responsibility boundary, out of scope |
| `SessionClosed` | step 16 (`cause`) |

Every event has at least one consumer inside the walk. The single unconsumed field is
`ConnectionAccepted.local_address` — that is H6, not an oversight.

### The `Received:` header, clause by clause

| Clause | Value | Source |
|:--|:--|:--|
| `FROM` domain | bar.com | `ClientIdentified.claimed_domain` |
| address literal | 203.0.113.20 | `ConnectionAccepted.peer_address` |
| `BY` | foo.com | `ServiceConfigured.server_domain` — **the config arm; 🟥 H6 open** |
| `WITH` | SMTP | `ClientIdentified.protocol` |
| `ID` | f2C8D14 | `MessageAccepted.queue_id` |
| `FOR` | \<Jones@foo.com> | `RecipientAccepted.forward_path` — emitted, because exactly one |
| timestamp | 1998-05-19T09:14:07-07:00 | `MessageAccepted.received_at` |

Unchanged from the original except `BY`, which the original sourced from unmodeled config and this
walk sources from the seeded event — same fact, now with an origin the completeness check can see.

---

## Against the existing walks

**The bytes are identical.** Same scene, same dialogue, same reply codes, same message, same
`Received:` header values as the original ten-step walk. What changed is what the page admits to:

- Seven replies and one trace line now each have a step; the original had all seven inside command
  wire rows and the trace only in an accounting table.
- The preamble replaces two silences: configuration that nothing declared, and a translation the
  original marked yellow mid-walk. Both are seeded events now, cited 🟧 where consumed.
- `Given` rows are minimal and carry fields exactly where a value, not just existence, is used —
  steps 2, 4, 6, 7, 8, 9, 13, 14 and 16; existence alone suffices at steps 3, 5, 10, 11, 12
  and 15.
- The `session_id` the `Where` rows always needed is instantiated at last — the original never
  named one, and the explorations carried the placeholder 01J8Z…, called a rule 8 break when it
  first appeared in [`EXPLORE-view-slice.md`](EXPLORE-view-slice.md).
- Nothing this file asserts contradicts the original's findings; the original's *What this walk
  tested* items (field independence, the `FOR` clause rule, happy-for-every-actor) all still hold
  and are not restated.

---

## What this walk tested

1. **The completeness check closes in both directions on one page.** Backward: eight outputs,
   every varying value with an origin. Forward: every event consumed, one field flagged. Neither
   direction was checkable inside the original form.
2. **The preamble carries a whole path.** Two seeded events satisfy every `Given` no walked step
   could — including the trace header's, which needs four walked events *and* a seeded one.
3. **Three stream scopes are visible in one walk** — the `Where` rows select by `session_id`
   (session scope) and `queue_id` (message scope), while `ServiceConfigured` folds in from service
   lifetime. Evidence for H4, found by walking rather than argued.

## What it did not test

- `Reset` — still uncovered by any path. D.2 remains the only thing that will close it.
- `SessionTranscript` — no operator in this scene.
- **Any error branch** — no 4xx, no 5xx, no sequencing error. H2 stays untouched.
- **Received-header stacking** — the relay path's job, not this one's.

---

## Hotspots

Only what is open or new. H3 and H5 are resolved and therefore absent; the five path-defined views
and two seeded events carry no markers, because *paths are the source* — they are this path's
contribution, not doubts about it.

**🟥 H6 — open, and it bites at step 13.** Is `Received:`'s `BY` sourced from configuration or
from `local_address`? This walk renders the config arm. `local_address` has two candidate
consumers — the multi-homed `BY` arm at step 13, and the nameless-server greeting, since the `220`
identity slot admits an address literal (§4.2). This walk exercises neither, so the field stays
its one unconsumed value.

**🟥 H4 — open, and this walk sharpened what needs ruling.** A stream is the unit of ordering,
of atomic invariants, and of a fold's scope — so H4 decides every `Where` key on this page. Five
pieces of evidence this walk adds:

1. **Three scopes appear side by side**: service lifetime (`ServiceConfigured`), session
   (`session_id` = 00T22SDG4R9FQ3ZK7VWX2M5N8P), and message (`queue_id` = f2C8D14) — in one
   sixteen-step conversation.
2. **`session_id` has no protocol existence.** The RFC names no session identifier; the channel
   is the identity. So the key is either envelope metadata or the session stream's name — it can
   never be payload, because no emitted output carries it: no destination, and a fact needs both.
3. **No transaction identifier exists anywhere.** Every session-scoped `Where` uses `session_id`;
   `TransactionState` included. That works because at most one transaction is open per session at
   a time — but it means per-transaction streams would require minting an id no step consumes.
4. **Step 13 is the only non-session `Where`.** The message, keyed by the RFC's own trace
   identifier, is the one thing that outlives the session — the responsibility boundary is the
   natural stream boundary.
5. **§4.3.1 serializes the session** — the client MUST wait for each reply — so there is never
   write contention inside one session, and splitting streams buys no concurrency inbound.

What needs ruling, in order: **(i)** is `session_id` the session stream's name or envelope
metadata; **(ii)** does the transaction get its own stream — and therefore a minted identifier —
or is it a phase inside the session stream, which the aborted-transaction walk (D.2, `Reset`) is
the test for; **(iii)** does the message become its own stream at `MessageAccepted`, named
`queue_id`, crossing the boundary into delivery's scope.

**Ruled 2026-08-08: this path carries no `correlation_id`.** No step consumes such a value, so
under this path's own discipline it does not appear — *needed* is the only claim that seats a
value here. The model-level question is flagged upward rather than settled: the model's metadata
table carries `session_id`, `correlation_id` and `queue_id` — three correlation schemes against
its own warning that the RFC's `ID` clause and `queue_id` *"are one mechanism, not two"* — and
with session = connection verified 1:1, `correlation_id` duplicates `session_id` inside this
scope while `queue_id` supersedes it beyond. Something to deal with at a higher level, not here.

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
