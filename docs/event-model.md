# FnEmail — Event Model (RFC 5321, inbound)

Status: **draft v0.2 — rebuilt against primary sources**
Scope: **incoming SMTP only.** Outbound relay, retry, bounce and DNS are deliberately deferred.

Method reference: `research/METHOD-REFERENCE.md`. Corrections from v0.1: `research/CORRECTIONS-v0.1.md`.
Extension ideas are parked in `event-model-extensions.md` and are **not** applied here — this
document is orthodox Dymitruk/Dilger by intent, so that extensions can be measured against it.

---

## What changed from v0.1

v0.1 was written from recall. Reading the primary sources overturned much of it.

| v0.1 | Correction |
|---|---|
| Four "swimlanes" = wireframe/command/read-model/event rows | **Swimlanes** are actor lanes (step 3.1) or event-ownership lanes (step 6). The rows are not swimlanes. |
| "The wire IS the wireframe" | Wrong construct, right instinct. The canonical element is a **Trigger**, which explicitly admits *"the route of an http endpoint"* — so an SMTP verb qualifies with no adaptation. |
| Reply codes are the UI lane | Reply codes are the **synchronous result of a command**. The operator's transcript is a separate **view screen** in an Admin actor lane. |
| Narrative phases | **Slices.** One column, one command *or* one read model. This is the structural fix. |
| "Pivot event" | Not a term in the corpus. The RFC §2.1 insight is real and kept — relabelled as a **responsibility boundary**, not presented as method vocabulary. |
| 1 command → many events (celebrated) | The **"left chair"** anti-pattern. Inbound-only scope dissolves it. |
| Completeness traced backward only | **Bidirectional** — every field needs an origin *and* a destination. |
| Four patterns as primitives | Two **column types**; four patterns are the workshop vocabulary over them. |
| Open questions in an appendix | **Hotspots**, on the model. |
| Processor implicit in prose | **Processor** is a first-class element. |

---

## Conventions

Colours: Event **orange** · Command **blue** · Read Model **green** · Screen **white** ·
external event **yellow** · hotspot **red**

Command is always blue and Read Model is always green across every source. Event is legitimately
orange (current) or yellow (older material); **this project uses orange**. One file in the
upstream kit transposes Event and View — it is wrong, and is documented in
`research/UPSTREAM-DEFECTS.md`.

**Arrows are not used.** Meaning is carried by lane position, left-to-right time, and which
pattern a column matches. Legal adjacency still binds (Dymitruk): `command → event → view →
trigger/processor → command`; **an event may never connect directly to a command.**

---

## Actor swimlanes

Two actors, top of the board (step 3.1).

| Lane | Kind | Element | Role |
|---|---|---|---|
| **Remote MTA** | system actor | **automation node** — labelled with the protocol verb | Drives every write column |
| **Operator** | human actor | **view screen** — session transcript | Reads; never issues a command |

The remote MTA gets an automation node, not a screen. Dilger, 2026: *"Human roles get SCREEN
nodes. System actors and processors get AUTOMATION nodes."* The node is labelled with the wire
command line — `MAIL FROM:<…>` — which sits exactly where an endpoint route would.

One nuance worth recording: the kit's phrasing describes a processor that *"reacts to events
automatically."* The remote MTA does not react to our events, it initiates. The element is still
right — a machine actor with no human interaction — but this is slightly outside the case the
kit describes.

Constraint observed: *"There are no screens that appear above one another"* — the Operator lane's
transcript gets its own column; it is not stacked onto a command column.

---

## Ownership swimlanes

Step 6. Events organised by owner:

| Swimlane | Owns |
|---|---|
| **Edge** | `ConnectionAccepted`, `ClientIdentified`, `TlsNegotiated`, `SessionReset`, `SessionClosed` |
| **Transaction** | `MailTransactionStarted`, `RecipientAccepted`, `RecipientRejected`, `DataPhaseEntered`, `MessageAccepted`, `MessageRejected`, `MessageDeferred`, `TransactionAborted` |
| **Directory** 🔴 | Mailbox existence and relay authorisation — **hotspot**, see H3 |

---

## The responsibility boundary

RFC 5321 §2.1:

> when an SMTP server accepts a message, it is accepting responsibility for delivering or
> reporting the failure to do so.

`MessageAccepted` — emitted with the `250` after the final `<CRLF>.<CRLF>` — is where obligation
begins. Left of it, abandoning the session costs nothing. Right of it lies delivery, which is out
of scope for v0.2.

This is a **domain observation about SMTP**, not Event Modeling vocabulary. v0.1 called it a
"pivot event" and implied the method sanctioned the term. It does not.

---

## Slice inventory

Timeline runs left to right. Each row is one column. `W` = write column (command),
`R` = read column (read model).

| # | Slice | Type | Pattern | Produces / projects |
|---|---|---|---|---|
| 1 | `AcceptConnection` | W | State Change | `ConnectionAccepted` \| error `554` |
| 2 | `Ehlo` | W | State Change | `ClientIdentified` \| error `500/501` |
| 3 | `StartTls` | W | State Change | `TlsNegotiated` \| error `454` |
| 4 | `SessionState` | R | State View | ← `ConnectionAccepted`, `ClientIdentified`, `TlsNegotiated`, `SessionReset` |
| 5 | `MailFrom` | W | State Change | `MailTransactionStarted` \| error `503/550/552` |
| 6 | `RecipientDirectory` 🔴 | R | State View | ← *unresolved — see H3* |
| 7 | `RcptTo` | W | State Change | `RecipientAccepted` \| `RecipientRejected` |
| 8 | `TransactionState` | R | State View | ← `MailTransactionStarted`, `RecipientAccepted`, `TransactionAborted` |
| 9 | `BeginData` 🔴 | W | State Change | `DataPhaseEntered` \| error `503/554` — *see H1* |
| 10 | `SubmitContent` | W | State Change | `MessageAccepted` \| `MessageRejected` \| `MessageDeferred` |
| 11 | `Reset` | W | State Change | `TransactionAborted` |
| 12 | `Quit` | W | State Change | `SessionClosed` |
| 13 | `SessionTranscript` | R | State View | ← every event above |

Thirteen slices, each independently buildable, each communicating only via events.

**Read models are placed at their consumer, not their source.** `SessionState` projects
`ClientIdentified` from early in the session but sits at column 4 because that is where the
sequencing decision needs it. `TransactionState` likewise serves columns 7, 9 and 10.

### No `ServiceGreetingSent` event

The `220` banner is not modelled as an event. Nothing consumes it: the transcript can render it
via `derived:` from `ConnectionAccepted` plus configuration, and the `Received:` header takes
`server_domain` from config. Under the bidirectional check it has an origin but no destination,
so it does not earn a place. `ServiceRefused` **is** a real decision (`554`) and appears as
`AcceptConnection`'s error scenario.

This is the completeness check doing actual work rather than ratifying what was already drawn.

---

## Events

Past tense, facts only, no computed fields.

| Event | Fields |
|---|---|
| `ConnectionAccepted` | `peer_address`, `local_address`, `occurred_at` |
| `ClientIdentified` | `claimed_domain`, `protocol` (SMTP\|ESMTP), `extensions_offered[]` |
| `TlsNegotiated` | `cipher`, `protocol_version`, `peer_cert_subject?` |
| `SessionReset` | — |
| `SessionClosed` | `cause` (quit \| timeout \| abort \| shutdown) |
| `MailTransactionStarted` | `reverse_path`, `declared_size?`, `body_type?` |
| `RecipientAccepted` | `forward_path`, `is_local` |
| `RecipientRejected` | `forward_path`, `reply_code`, `reason` |
| `DataPhaseEntered` 🔴 | — |
| `MessageAccepted` | `queue_id`, `reverse_path`, `recipients[]`, `content_ref`, `actual_octets` |
| `MessageRejected` | `reply_code`, `reason` |
| `MessageDeferred` | `reply_code`, `reason` |
| `TransactionAborted` | `cause` (rset \| quit \| disconnect) |

`declared_size` (from `MAIL FROM … SIZE=`) and `actual_octets` (measured at receipt) are
**different facts** and both are kept. Conflating them is a classic `SIZE`-enforcement bug.

Message bodies stay out of the event log behind `content_ref` — the book's *forgettable payload*.
Keeping octets in the stream turns the log into a mail spool and makes replay impossible.

---

## Scenarios

### 1 — Sequencing: `RCPT` before `MAIL` (§4.1.1)

```
Slice:  RcptTo                                              [error]
GIVEN   ConnectionAccepted, ClientIdentified
WHEN    RcptTo(<bob@example.org>)
THEN    error 503 Bad sequence of commands
```

### 2 — `RSET` clears the transaction, keeps the session (§4.1.1.5)

```
Slice:  Reset                                        [state change]
GIVEN   ClientIdentified(mail.acme.com),
        MailTransactionStarted(<alice@acme.com>),
        RecipientAccepted(<bob@fn.email>)
WHEN    Reset
THEN    TransactionAborted(cause=rset)                     → 250 OK
```

```
Slice:  SessionState                                   [state view]
GIVEN   ClientIdentified(mail.acme.com), TlsNegotiated(...), SessionReset
WHEN    Query(session)
THEN    SessionState{ identified: true, tls: true, transaction: none }
```

The session survives; only the transaction resets. Two slices, two specs — this is why the
split into separate read and write columns matters rather than being ceremony.

### 3 — Responsibility transfer

```
Slice:  SubmitContent                                [state change]
GIVEN   MailTransactionStarted(<alice@acme.com>),
        RecipientAccepted(<bob@fn.email>),
        RecipientAccepted(<carol@fn.email>),
        DataPhaseEntered
WHEN    SubmitContent(octets, dot-unstuffed)
THEN    MessageAccepted(queue_id, content_ref, actual_octets)
                                          → 250 OK queued as <queue_id>
```

**One command, one event.** In v0.1 this emitted three event types plus a per-recipient fan-out —
the "left chair". Scoping to inbound removed the fan-out, and the completeness check removed
`MessageContentReceived` as redundant with `MessageAccepted`.

### 4 — `SIZE` exceeded (§4.5.3.1.10, RFC 1870)

```
Slice:  SubmitContent                                       [error]
GIVEN   MailTransactionStarted(<alice@acme.com>, declared_size=1000),
        RecipientAccepted(<bob@fn.email>), DataPhaseEntered
WHEN    SubmitContent(octets where actual_octets=52_000_000)
THEN    error 552 Message exceeds fixed maximum message size
```

### 5 — Relay refused

```
Slice:  RcptTo                                       [state change]
GIVEN   ClientIdentified(unknown.example), MailTransactionStarted(<x@elsewhere>)
WHEN    RcptTo(<victim@third-party.org>)
THEN    RecipientRejected(reply_code=550, reason="relay not permitted")
```

Modelled as an event, not a bare error: a refused relay attempt is a fact worth keeping. Contrast
scenario 1, where a protocol sequencing slip produces no event. **That distinction is a live
question — see H2.**

---

## Completeness check

Bidirectional: every field needs an origin **and** a destination.

### Backward — the `Received:` header (§4.4)

| Clause | Origin |
|---|---|
| `FROM` domain | `ClientIdentified.claimed_domain` |
| `FROM` address literal | `ConnectionAccepted.peer_address` |
| `BY` | config (`derived:`) |
| `WITH ESMTPS` | `ClientIdentified.protocol` + `TlsNegotiated` |
| `ID` | `MessageAccepted.queue_id` |
| `FOR` | `RecipientAccepted.forward_path` *(only when exactly one)* |
| timestamp | `MessageAccepted.occurred_at` |

Every clause resolves. Inverting it is where the value is: drop `peer_address` from
`ConnectionAccepted` and a conformant `Received:` header becomes impossible to write — caught on
the model, not three weeks into implementation.

### Forward — the session transcript

The `SessionTranscript` read model is what makes the forward direction testable:

> **If the session transcript cannot be reconstructed from the event stream alone, the model is
> incomplete.**

Stronger than the header check in two ways: it covers the whole session rather than one field
set, and it exercises the direction that exposes events nothing consumes. `DataPhaseEntered`
survives or dies by this test (H1).

---

## Hotspots

Open questions, on the model rather than in an appendix. *"Unknowns become hotspots."*

**H1 — Does `DataPhaseEntered` earn its place?**
It is a genuine fact, but its only candidate consumer is the transcript rendering `354 Start mail
input`. If the transcript begins at `MAIL FROM`, nothing consumes it and it should go. This is a
question about what the operator needs to see, not about protocol purity.

**H2 — Are protocol errors events?**
`RecipientRejected` is modelled as an event; a `503` sequencing error is not. The line drawn is
"policy decisions are facts, protocol slips are not" — defensible but unverified. Auditing and
abuse detection may want both.

**H3 — Where does recipient knowledge come from?**
`RecipientDirectory` has no event source in this model. Mailbox existence and relay authorisation
originate in provisioning, which is a different concern entirely. Candidates: a Logic Read Model
over configuration, or a genuinely separate model. This is the strongest pull toward the
multiple-models idea parked in `event-model-extensions.md`.

**H4 — Stream design.**
Unresolved: one stream per session, per transaction, or per message? The session outlives the
transaction (scenario 2), which argues they are not the same stream. Needs the book's
stream-design guidance (p137–151) applied properly.

**H5 — Pipelining (RFC 2920).**
The timeline assumes one command per column. Pipelining collapses round trips. Whether that
changes the model or only the transport is unexamined.

**H6 — `AcceptConnection` altitude.**
Is a TCP accept a domain fact or infrastructure noise? It carries `peer_address`, which the
`Received:` header needs, so it currently earns its place — but that is the argument, not an
assumption.

---

## Deferred

Out of scope for v0.2, to be modelled once inbound is settled:

- Outbound delivery, relay, retry, bounce generation
- DNS/MX resolution (the Translation pattern's natural home)
- The Processor-TODO-List pattern for the delivery queue
- Submission (RFC 6409) on :587 — same protocol shape, different policy
- Authentication (RFC 4954)
- Workflow step contracts — pre/postconditions per slice, for parallel implementation

---

## Method caveat

No source in the corpus applies Event Modeling to a protocol server. The method is demonstrated
on user-facing business systems throughout. This model is therefore an **extension of its
demonstrated range**, and the two places that strain hardest are the trigger row (a machine actor
rather than a person) and the absence of any real screen. Both have canonical footing — triggers
admit API routes, screens are optional — but neither is a case its authors worked through.

Treat the structure as sound and the application as unproven.
