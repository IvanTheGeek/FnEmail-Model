# FnEmail — Event Model (RFC 5321, inbound, HELO)

Status: **draft v0.3 — scoped down to the HELO path**
Scope: **inbound SMTP, `HELO` only.** No ESMTP, no relay, no outbound.

Method reference: `research/METHOD-REFERENCE.md`. Corrections from v0.1: `research/CORRECTIONS-v0.1.md`.
Altitude rules — what belongs in this model at all: `model-altitude.md`.
Extensions are parked in `event-model-extensions.md` and **not** applied here — this document is
orthodox Dymitruk/Dilger by intent, so extensions can be measured against it.

---

## Scope

`HELO` only. That removes more than one verb, because the whole extension mechanism is ESMTP:

| Removed | Why |
|---|---|
| `EHLO` / extension negotiation | ESMTP (§2.2) |
| `STARTTLS` | RFC 3207 extension |
| `SIZE=` on `MAIL FROM` | RFC 1870 extension |
| `BODY=8BITMIME` | RFC 6152 extension |
| Relay, outbound, DNS, bounce | deferred by scope |

Twelve slices remain. Every reply is 3-digit only — no enhanced status codes (RFC 3463 is also an
extension).

## Normative base

The hard-lined rules for this model are **RFC 5321 + RFC 7504**, and nothing else.

RFC 7504 (*SMTP 521 and 556 Reply Codes*, June 2015) is the **sole** update to RFC 5321 — the RFC
Editor's entry for 5321 lists exactly one. Both are archived under `research/archive/rfc/`.

The normative set is therefore a small, versioned graph rather than one document, and conformance
claims must name the set. It can grow: a future update would change what "RFC-conformant" means
without any change to our model.

| RFC | Role |
|---|---|
| 5321 | base specification (obsoletes 2821; updates 1123) |
| 7504 | adds reply codes 521 and 556 |

## Changes from v0.2

- Dropped the `StartTls` slice; `Ehlo` → `Helo`, `protocol` fixed at `SMTP`.
- `MailTransactionStarted` loses `declared_size` and `body_type`; the declared-size error scenario
  goes with them. Size enforcement now only happens on `actual_octets`.
- **Added step contracts** to every slice. These are not new work — a contract is a GWT with the
  example data removed, so `GIVEN` is the precondition and `THEN` the postcondition.
- **H3 reframed.** An unsourced read model is the method working, not a defect: the hole becomes
  known and names a slice that must exist upstream. It is now marked as a *typed* hole so that a
  path crossing it fails loudly.
- **Metadata section added**, and two payload defects fixed — both found by walking the path with
  real data rather than by reading the model. `MessageAccepted` gained `received_at`;
  `ConnectionAccepted` lost `occurred_at`; `local_address` is flagged as an orphan (H6).

---

## Conventions

Event **orange** · Command **blue** · Read Model **green** · Screen **white** · hotspot **red**.
Command is blue and Read Model is green in every source; Event is orange here (see
`research/UPSTREAM-DEFECTS.md` for the one file that disagrees).

**Arrows are not used.** Meaning comes from lane position, left-to-right time, and which pattern a
column matches. Legal adjacency still binds: `command → event → view → trigger/processor →
command`; **an event may never connect directly to a command.**

---

## Actor swimlanes

| Lane | Kind | Element | Role |
|---|---|---|---|
| **Remote client** | actor role | **command screens** — the terminal exchange | Drives every write column |
| **Operator** | actor role | **view screen** — session transcript | Reads; never issues a command |

**There is no automation node in this model.** An earlier draft put the remote MTA in the actor row
as one. That was wrong, for two reasons.

**Dymitruk defines an automation as a todo list belonging to a processor *in our system*:**

> *"We can make the concept of how this occurs with the idea of a 'todo list' for some processor in
> our system."*

The remote client has no todo list of ours. It is not a processor we drive; it is an actor driving
us. He also notes automations *"may even actually be manual todo lists that our employees use"* —
so human-versus-machine was never the distinguishing axis. Working a todo list is.

**And the server cannot tell a human from a script.** `HELO bar.com` is identical whether it came
from Postfix or from someone typing into `telnet host 25`. SMTP is a human-typeable text protocol
by design. An element type that flips on a property the system cannot observe is not well formed.

So the remote client is an ordinary **actor role**, and roles get screens. Each write column carries
a command screen: the prior reply as displayed context, and the line the actor sends.

```
┌──────────────────────────┐   ┌──────────────────────────┐
│ 250 foo.com              │   │ 250 OK                   │
│ > MAIL FROM:<Smith@bar…> │   │ > RCPT TO:<Jones@foo.com>│
└──────────────────────────┘   └──────────────────────────┘
        MailFrom column                RcptTo column
```

Cheap to draw, which matters — *"Drawing a screen must not take longer than 2 minutes."* One screen
per column per lane, never stacked, per §3.1.

### Where the automation actually belongs

Outbound, not inbound. The roles invert:

| | Inbound *(this model)* | Outbound *(deferred)* |
|---|---|---|
| Actor | remote client — **screens** | our delivery processor — **automation, real todo list** |
| Direction | they command, we reply | we command, they reply |
| Their replies | our command results | **external events → Translation** |

Outbound the delivery agent genuinely is Adam's automation: it works a todo list, which is the
book's *Processor-TODO-List* pattern by name. The earlier draft had the right element in the wrong
model.

## Ownership swimlanes

| Swimlane | Owns |
|---|---|
| **Edge** | `ConnectionAccepted`, `ClientIdentified`, `SessionReset`, `SessionClosed` |
| **Transaction** | `MailTransactionStarted`, `RecipientAccepted`, `RecipientRejected`, `DataPhaseEntered`, `MessageAccepted`, `MessageRejected`, `TransactionAborted` |
| **Directory** 🔴 | Not owned here — see H3 |

---

## The responsibility boundary

RFC 5321 §2.1: *"when an SMTP server accepts a message, it is accepting responsibility for
delivering or reporting the failure to do so."*

`MessageAccepted` is where obligation begins. Left of it, abandoning costs nothing. Right of it is
delivery — out of scope.

A domain observation about SMTP, **not** Event Modeling vocabulary.

---

## Slices and contracts

`W` = write column (one command). `R` = read column (one read model).

### 1 · `AcceptConnection` — W

```
Pre    none
Post   ConnectionAccepted{peer_address, local_address}
       OR  reply 554, no event
```
Exists to carry `peer_address`, which the `Received:` header requires and nothing else supplies.

**`554` here is correct, and must not be "upgraded" to `521`.** RFC 7504 §3 reserves `521` for a
host that *"does not accept mail under any circumstances"* — a dummy server whose only job is to
say so. It *"SHOULD NOT be used for situations in which the server rejects mail from particular
hosts or addresses"*, which is exactly our blocklist case. `554` remains right. See H7 for the
case where `521` would apply.

### 2 · `Helo` — W

```
Pre    ConnectionAccepted exists
Post   ClientIdentified{claimed_domain, protocol:"SMTP"}
       OR  reply 501 (bad syntax) / 500 (unrecognised)
```

### 3 · `SessionState` — R

```
Sources   ConnectionAccepted, ClientIdentified, SessionReset
Answers   Has the client identified itself? Is a transaction open?
Post      SessionState{identified: bool, transaction_open: bool}
```
Placed here because slices 4, 6 and 8 consume it — not next to its sources.

### 4 · `MailFrom` — W

```
Pre    SessionState.identified = true
Post   MailTransactionStarted{reverse_path}
       OR  reply 503 (no HELO) / 550 (sender rejected)
```

### 5 · `RecipientDirectory` — R 🔴 **typed hole, H3**

```
Sources   ⚠️ NONE IN THIS MODEL
Answers   Is this address local? May this client relay?
Post      RecipientDirectory{is_local: bool, relay_permitted: bool}
```
Deliberately left unsourced and typed. Mailbox existence comes from provisioning; relay
authorisation from operator configuration. Neither is an event here. **This is the method
working** — the hole names a slice that must exist upstream.

### 6 · `RcptTo` — W

```
Pre    MailTransactionStarted exists for this session
       RecipientDirectory available
Post   RecipientAccepted{forward_path, is_local}
       OR  RecipientRejected{forward_path, reply_code, reason}
```
Walked once per recipient. The same column, not N columns.

### 7 · `TransactionState` — R

```
Sources   MailTransactionStarted, RecipientAccepted, TransactionAborted
Answers   Open? Reverse path? How many recipients?
Post      TransactionState{open: bool, reverse_path, recipient_count}
```

### 8 · `BeginData` — W 🔴 **H1**

```
Pre    TransactionState.open = true
       TransactionState.recipient_count >= 1
Post   DataPhaseEntered
       OR  reply 503 (no recipients)
```
The *command* is sound — it makes a real decision. The *event* is on trial: its only candidate
consumer is the transcript rendering `354`.

### 9 · `SubmitContent` — W

```
Pre    DataPhaseEntered for the current transaction
Post   MessageAccepted{queue_id, reverse_path, recipients[], content_ref,
                      actual_octets, received_at}
       OR  MessageRejected{reply_code, reason}
```
**One command, one event.** v0.1 emitted three event types plus a per-recipient fan-out — the
"left chair". Inbound scope removed the fan-out; the forward completeness pass removed
`MessageContentReceived` as redundant.

### 10 · `Reset` — W

```
Pre    SessionState.identified = true
Post   TransactionAborted{cause:"rset"}
       Postcondition explicitly does NOT touch SessionState
```

### 11 · `Quit` — W

```
Pre    ConnectionAccepted exists
Post   SessionClosed{cause}
```

### 12 · `SessionTranscript` — R

```
Sources   every event above
Answers   What happened on this session, and why was it rejected?
Post      an ordered rendering of the wire exchange
```
Structurally the **"right chair"** — one read model, many events. Expected; worth watching.

---

## Events

| Event | Fields |
|---|---|
| `ConnectionAccepted` | `peer_address`, `local_address` ⚠️ |
| `ClientIdentified` | `claimed_domain`, `protocol` (always `SMTP`) |
| `SessionReset` | — |
| `SessionClosed` | `cause` (quit \| timeout \| abort \| shutdown) |
| `MailTransactionStarted` | `reverse_path` |
| `RecipientAccepted` | `forward_path`, `is_local` |
| `RecipientRejected` | `forward_path`, `reply_code`, `reason` |
| `DataPhaseEntered` 🔴 | — |
| `MessageAccepted` | `queue_id`, `reverse_path`, `recipients[]`, `content_ref`, `actual_octets`, `received_at` |
| `MessageRejected` | `reply_code`, `reason` |
| `TransactionAborted` | `cause` (rset \| quit \| disconnect) |

Message bodies stay out of the log behind `content_ref` — the book's *forgettable payload*.

### Metadata

Every event carries ambient metadata, **never repeated in the payload**. Dilger: *"The event
payload holds the business-relevant information, while the metadata contains context-specific
details that support the event."*

| Metadata | SMTP meaning |
|---|---|
| `event_id` | — |
| `session_id` | the connection |
| `correlation_id` | ties every event in one session |
| `causation_id` | the verb that produced this event |

SMTP already has this concept under another name: `Received:`'s **`ID` clause** is a trace
identifier, and `queue_id` is what we put in it. These are one mechanism, not two — do not invent
a second correlation scheme alongside the RFC's.

⚠️ Naming caution: Dilger's prose has correlation propagating across steps and causation
identifying the current step, but Axon's Message Origin Provider uses "Correlation ID" for the
origin message and "Trace ID" for the propagated one. Pick names deliberately.

### Why `received_at` is payload, not metadata

Timestamp is absent from Dilger's metadata list, implying the event store supplies it. That is
wrong for this field, for one reason:

> **The `Received:` timestamp must survive replay unchanged.**

RFC 5321 §4.4 requires the date-time as emitted output, and the header is a permanent trace record
on the message. If the timestamp came from store-append time, rebuilding the store would produce
a different `Received:` header. It is business-relevant output, which by Dilger's own test makes
it payload.

`ConnectionAccepted.occurred_at` is removed by the same reasoning inverted — nothing consumes it,
and ambient store time covers the diagnostic need.

### ⚠️ `local_address` is an orphan

Found by walking the path with data. It has an origin and **no destination**: `Received:`'s `BY`
clause is `derived:` from configuration, so nothing reads it.

Either give it a consumer or delete it. The defensible consumer: on a multi-homed server listening
on several addresses with different hostnames, `BY` depends on *which* address was connected to —
in which case `BY` sources from `local_address`, not config, and the field earns its place. That
case is not yet decided, so the field stays flagged rather than removed.

### No `ServiceGreetingSent`

The `220` banner is not an event. Nothing consumes it: the transcript renders it `derived:` from
`ConnectionAccepted` plus config, and `Received:` takes `BY` from config. Origin but no
destination — it does not earn a place. Deleted by the forward completeness pass, which is the
check doing real work rather than ratifying what was drawn.

---

## Scenarios

**Sequencing — `RCPT` before `MAIL`** (§4.1.1)
```
Slice   RcptTo                                            [error]
GIVEN   ConnectionAccepted, ClientIdentified
WHEN    RcptTo(<bob@example.org>)
THEN    error 503 Bad sequence of commands
```

**`RSET` clears the transaction, keeps the session** (§4.1.1.5)
```
Slice   Reset                                      [state change]
GIVEN   ClientIdentified(mail.acme.com),
        MailTransactionStarted(<alice@acme.com>),
        RecipientAccepted(<bob@fn.email>)
WHEN    Reset
THEN    TransactionAborted(cause=rset)                   → 250 OK
```
```
Slice   SessionState                                 [state view]
GIVEN   ClientIdentified(mail.acme.com), SessionReset
WHEN    Query(session)
THEN    SessionState{identified: true, transaction_open: false}
```

**Responsibility transfer**
```
Slice   SubmitContent                              [state change]
GIVEN   MailTransactionStarted(<alice@acme.com>),
        RecipientAccepted(<bob@fn.email>),
        RecipientAccepted(<carol@fn.email>),
        DataPhaseEntered
WHEN    SubmitContent(octets, dot-unstuffed)
THEN    MessageAccepted(queue_id, content_ref, actual_octets)
                                       → 250 OK queued as <queue_id>
```

**Relay refused**
```
Slice   RcptTo                                     [state change]
GIVEN   ClientIdentified(unknown.example),
        MailTransactionStarted(<x@elsewhere>),
        RecipientDirectory{is_local: false, relay_permitted: false}
WHEN    RcptTo(<victim@third-party.org>)
THEN    RecipientRejected(reply_code=550, reason="relay not permitted")
```
Modelled as an event, not a bare error — a refused relay attempt is a fact worth keeping.
Contrast the sequencing slip above, which produces none. **That line is H2.**

---

## Completeness check

Bidirectional: every field needs an origin **and** a destination.

**Backward — `Received:` (§4.4)**

| Clause | Origin |
|---|---|
| `FROM` domain | `ClientIdentified.claimed_domain` |
| `FROM` address literal | `ConnectionAccepted.peer_address` |
| `BY` | config (`derived:`) |
| `WITH SMTP` | `ClientIdentified.protocol` — always `SMTP` in this scope |
| `ID` | `MessageAccepted.queue_id` |
| `FOR` | `RecipientAccepted.forward_path` *(only when exactly one)* |
| timestamp | `MessageAccepted.received_at` — **payload, not metadata** (see below) |

**Forward — the transcript**

> If the session transcript cannot be reconstructed from the event stream alone, the model is
> incomplete.

Covers the whole session rather than one field set, and exercises the direction that exposes
events nothing consumes. `DataPhaseEntered` lives or dies by it.

---

## Hotspots

**H1 — Does `DataPhaseEntered` earn its place?** Its only candidate consumer is the transcript
rendering `354`. A question about what the operator needs to see. See `model-altitude.md` §3, which
resolves this through the destination gate rather than by argument about protocol purity.

**H2 — Are protocol errors events?** `RecipientRejected` is; a `503` is not. The line drawn is
"policy decisions are facts, protocol slips are not" — defensible, unverified.

**H3 — `RecipientDirectory` is unsourced, deliberately.** Not a defect. The hole is now typed, and
it names a slice that must exist in an upstream model.

**H4 — Stream design.** One stream per session, per transaction, or per message? The session
outlives the transaction, which argues they are not one stream.

**H6 — Is `BY` config or `local_address`?** Decides whether `local_address` is an orphan field or
load-bearing. Turns on whether FnEmail will ever be multi-homed with per-address hostnames.

**H7 — Does FnEmail ever refuse mail entirely?** RFC 7504 `521` applies only to a host that never
accepts mail — a null-MX sentinel or a domain configured to accept none. If FnEmail supports that
mode, `AcceptConnection` gains a `521` branch and the model needs a configuration input saying so.
If not, `521` never appears and this can be closed. A product question.

Note the shape of the rule: RFC 7504 hard-lines *when* `521` is used, then leaves the aftermath to
the operator — after `521` the server **MAY** keep replying `521` or **MAY** just close the
connection. A normative frame with an operator choice inside it.

**H5 — `AcceptConnection` altitude.** Domain fact or infrastructure noise? It carries
`peer_address`, which `Received:` needs — that is the argument, not an assumption.

---

## Deferred

`EHLO`/ESMTP and extension negotiation · `STARTTLS` · `SIZE` · `8BITMIME` · enhanced status codes ·
pipelining · `VRFY`/`EXPN`/`HELP`/`NOOP` · outbound delivery, relay, retry, bounce · DNS/MX
translation · submission on :587 · authentication.

**RFC 7504 `556`** is deferred with relay. It is returned by an intermediate system that can tell —
typically from a null MX record (RFC 7505) — that a forward-path's domain accepts no mail, without
opening a connection to it. That only arises when relaying, so it belongs to the deferred outbound
work rather than here.

---

## Method caveat

No source in the corpus applies Event Modeling to a protocol server. The two places this strains
hardest are the trigger row (a machine actor, not a person) and the absence of any real screen.
Both have canonical footing — automation nodes for system actors, screens optional — but neither
is a case its authors worked through.

Structure sound; application unproven.
