# FnEmail — Event Model (RFC 5321, inbound, HELO)

Status: **draft v0.3 — scoped down to the HELO path**
Scope: **inbound SMTP, `HELO` only.** No ESMTP, no relay, no outbound.

Method reference: `research/METHOD-REFERENCE.md`. Corrections from v0.1: `research/CORRECTIONS-v0.1.md`.
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

## Changes from v0.2

- Dropped the `StartTls` slice; `Ehlo` → `Helo`, `protocol` fixed at `SMTP`.
- `MailTransactionStarted` loses `declared_size` and `body_type`; the declared-size error scenario
  goes with them. Size enforcement now only happens on `actual_octets`.
- **Added step contracts** to every slice. These are not new work — a contract is a GWT with the
  example data removed, so `GIVEN` is the precondition and `THEN` the postcondition.
- **H3 reframed.** An unsourced read model is the method working, not a defect: the hole becomes
  known and names a slice that must exist upstream. It is now marked as a *typed* hole so that a
  path crossing it fails loudly.

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
| **Remote MTA** | system actor | **automation node**, labelled with the verb | Drives every write column |
| **Operator** | human actor | **view screen** — session transcript | Reads; never issues a command |

Dilger 2026: *"Human roles get SCREEN nodes. System actors and processors get AUTOMATION nodes."*
The node is labelled with the wire line — `MAIL FROM:<…>` — sitting where an endpoint route would.

Nuance kept visible: the kit describes an automation as something that *"reacts to events
automatically."* A remote MTA initiates rather than reacts. Right element, slightly outside the
described case.

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
Post   ConnectionAccepted{peer_address, local_address, occurred_at}
       OR  reply 554, no event
```
Exists to carry `peer_address`, which the `Received:` header requires and nothing else supplies.

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
Post   MessageAccepted{queue_id, reverse_path, recipients[], content_ref, actual_octets}
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
| `ConnectionAccepted` | `peer_address`, `local_address`, `occurred_at` |
| `ClientIdentified` | `claimed_domain`, `protocol` (always `SMTP`) |
| `SessionReset` | — |
| `SessionClosed` | `cause` (quit \| timeout \| abort \| shutdown) |
| `MailTransactionStarted` | `reverse_path` |
| `RecipientAccepted` | `forward_path`, `is_local` |
| `RecipientRejected` | `forward_path`, `reply_code`, `reason` |
| `DataPhaseEntered` 🔴 | — |
| `MessageAccepted` | `queue_id`, `reverse_path`, `recipients[]`, `content_ref`, `actual_octets` |
| `MessageRejected` | `reply_code`, `reason` |
| `TransactionAborted` | `cause` (rset \| quit \| disconnect) |

Message bodies stay out of the log behind `content_ref` — the book's *forgettable payload*.

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
| timestamp | `MessageAccepted.occurred_at` |

**Forward — the transcript**

> If the session transcript cannot be reconstructed from the event stream alone, the model is
> incomplete.

Covers the whole session rather than one field set, and exercises the direction that exposes
events nothing consumes. `DataPhaseEntered` lives or dies by it.

---

## Hotspots

**H1 — Does `DataPhaseEntered` earn its place?** Its only candidate consumer is the transcript
rendering `354`. A question about what the operator needs to see.

**H2 — Are protocol errors events?** `RecipientRejected` is; a `503` is not. The line drawn is
"policy decisions are facts, protocol slips are not" — defensible, unverified.

**H3 — `RecipientDirectory` is unsourced, deliberately.** Not a defect. The hole is now typed, and
it names a slice that must exist in an upstream model.

**H4 — Stream design.** One stream per session, per transaction, or per message? The session
outlives the transaction, which argues they are not one stream.

**H5 — `AcceptConnection` altitude.** Domain fact or infrastructure noise? It carries
`peer_address`, which `Received:` needs — that is the argument, not an assumption.

---

## Deferred

`EHLO`/ESMTP and extension negotiation · `STARTTLS` · `SIZE` · `8BITMIME` · enhanced status codes ·
pipelining · `VRFY`/`EXPN`/`HELP`/`NOOP` · outbound delivery, relay, retry, bounce · DNS/MX
translation · submission on :587 · authentication.

---

## Method caveat

No source in the corpus applies Event Modeling to a protocol server. The two places this strains
hardest are the trigger row (a machine actor, not a person) and the absence of any real screen.
Both have canonical footing — automation nodes for system actors, screens optional — but neither
is a case its authors worked through.

Structure sound; application unproven.
