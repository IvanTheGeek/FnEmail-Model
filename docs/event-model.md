# FnEmail — Event Model (RFC 5321)

Status: **draft v0.1** — first pass, expected to change.

An [Event Modeling](https://eventmodeling.org/) blueprint (Adam Dymitruk's method)
for an SMTP system implementing [RFC 5321](https://www.rfc-editor.org/rfc/rfc5321).

This document is the design surface. Code should follow it; where code and model
disagree, one of them is wrong and we fix it here first.

---

## 1. Method

Time runs left to right. Four horizontal lanes:

| Lane | Contents |
|---|---|
| **Wireframe** | What the actor sees |
| **Command** | Intents entering the system |
| **Read model** | Projections built *from* events — feed UI and command decisions |
| **Event** | The immutable timeline; the only source of truth |

Built from exactly four patterns:

- **Command** — wireframe → command → event(s)
- **View** — event(s) → read model → wireframe
- **Automation** — read model as todo-list → processor → command → event(s)
- **Translation** — an external system's facts translated into our events

### 1.1 The wire is the wireframe

SMTP has no screen. The **reply codes are the UI** — `250`, `354`, `550`, `421`
are what the peer "sees". Every reply is a *view*: a projection of current
session and transaction state, rendered back to the client.

This is the reframing that makes the rest of the model work.

---

## 2. The pivot event

RFC 5321 §2.1:

> when an SMTP server accepts a message, it is accepting responsibility for
> delivering or reporting the failure to do so.

**`MessageAccepted`** — emitted the instant we write `250` after the final
`<CRLF>.<CRLF>` — is the fulcrum of the entire system.

- **Left of it:** negotiation. Abandon at any point and nothing is owed.
  Dropped connection mid-`RCPT` costs nobody anything.
- **Right of it:** an obligation. We must deliver, bounce, or keep retrying
  until the give-up window closes.

The model is split into two lanes at this event. A large share of SMTP
implementation bugs are a failure to respect that split.

---

## 3. Inbound — receiving

```
time ────────────────────────────────────────────────────────────────────▶

           │ EHLO     │ MAIL FROM │ RCPT TO   │ DATA    │ <CRLF>.<CRLF>    │
WIRE       │ 250-SIZE │ 250 ok    │ 250 ok    │ 354     │ 250 queued as X  │
           │ 250-STARTTLS         │ 550 no relay        │                  │
───────────┼──────────┼───────────┼───────────┼─────────┼──────────────────┤
COMMAND    │ Ehlo     │ MailFrom  │ RcptTo    │BeginData│ SubmitContent    │
───────────┼──────────┼───────────┼───────────┼─────────┼──────────────────┤
READ MODEL │ Session  │ Session   │ Txn +     │ Txn     │ Txn              │
 (decide)  │  State   │  State    │ RelayPol  │ State   │ State            │
───────────┼──────────┼───────────┼───────────┼─────────┼──────────────────┤
EVENT      │ Client   │ MailTxn   │ Recipient │DataPhase│ ContentReceived  │
           │Identified│ Started   │ Accepted  │ Entered │ MessageAccepted  │
           │          │           │           │         │ DeliveryRequested│
                                                          (one per rcpt)
```

### 3.1 Session-lifetime events

| Event | Carries |
|---|---|
| `ConnectionAccepted` | `peer_address`, `local_address`, `occurred_at` |
| `ServiceGreetingSent` | `server_domain`, `reply_code` |
| `ServiceRefused` | `reply_code` (554), `reason` |
| `ClientIdentified` | `claimed_domain`, `protocol` (SMTP\|ESMTP), `extensions_offered` |
| `TlsNegotiated` | `cipher`, `protocol_version`, `peer_cert_subject?` |
| `SessionReset` | — |
| `SessionClosed` | `cause` (quit \| timeout \| abort \| shutdown) |

### 3.2 Transaction-lifetime events

| Event | Carries |
|---|---|
| `MailTransactionStarted` | `reverse_path`, `declared_size?`, `body_type?` |
| `RecipientAccepted` | `forward_path`, `is_local` |
| `RecipientRejected` | `forward_path`, `reply_code`, `reason` |
| `DataPhaseEntered` | — |
| `MessageContentReceived` | `content_ref`, `actual_octets`, `line_count` |
| `MessageAccepted` | `queue_id`, `reverse_path`, `recipients[]` |
| `MessageRejected` | `reply_code` (5yz), `reason` |
| `MessageDeferred` | `reply_code` (4yz), `reason` |
| `TransactionAborted` | `cause` (rset \| quit \| disconnect) |

### 3.3 Fan-out at the pivot

One `MessageAccepted`, **N** `DeliveryRequested` — one per accepted recipient.

Per-recipient outcomes are why partial delivery works at all. Modeling the
fan-out at acceptance time makes partial success fall out for free rather than
being retrofitted after the first multi-recipient bug.

---

## 4. Outbound — automation, not command

No user here. A read model acts as a todo-list; a processor drains it.

```
EVENT      MessageAccepted ─▶ DeliveryRequested
                                     │
READ MODEL                    ┌──────▼───────────────────┐
                              │  DeliveryQueue (todo)    │
                              │  msg, rcpt, attempts,    │
                              │  next_attempt_at         │
                              └──────┬───────────────────┘
COMMAND                              ▼
                              ResolveRoute ─▶ AttemptRelay
EVENT                         RouteResolved   RelayAccepted   → done
                              RouteFailed     RelayDeferred   → reschedule
                                              RelayFailed     → bounce
                                              DeliveryGivenUp → bounce
```

### 4.1 Delivery events

| Event | Carries |
|---|---|
| `DeliveryRequested` | `queue_id`, `forward_path` |
| `RouteResolved` | `forward_path`, `hosts[]` (MX priority order) |
| `RouteResolutionFailed` | `forward_path`, `dns_status` |
| `RelayAttempted` | `host`, `attempt_no` |
| `RelayAccepted` | `host`, `remote_reply` |
| `RelayDeferred` | `host`, `reply_code` (4yz), `next_attempt_at` |
| `RelayFailed` | `host`, `reply_code` (5yz), `reason` |
| `DeliveryGivenUp` | `forward_path`, `first_attempt_at`, `attempts` |
| `MessageDelivered` | `forward_path`, `mailbox` (local delivery) |
| `BounceMessageGenerated` | `to`, `reverse_path` (always `<>`), `failed_recipients[]` |

### 4.2 Retry needs no scheduler

`RelayDeferred` carries `next_attempt_at`. The todo-list projection filters on
it. The processor picks up whatever is due.

RFC 5321 §4.5.4.1 ("at least 4–5 days before giving up") becomes a **predicate on
a projection**, not timer state scattered across the codebase.

### 4.3 Translation boundary

The remote MTA's replies and DNS answers are *external facts*, translated into
our events (`RelayAccepted` / `RelayDeferred` / `RelayFailed` / `RouteResolved`).

Our timeline never contains a foreign system's events verbatim.

---

## 5. Given / When / Then slices

Each slice is one testable spec and one unit of work.

### 5.1 Sequencing — `RCPT` before `MAIL`

```
GIVEN  ConnectionAccepted, ServiceGreetingSent, ClientIdentified
WHEN   RcptTo(<bob@example.org>)
THEN   ⊘ no transaction event  →  503 Bad sequence of commands
```

### 5.2 `RSET` clears the transaction, keeps the session (§4.1.1.5)

```
GIVEN  ClientIdentified(mail.acme.com),
       MailTransactionStarted(<alice@acme.com>),
       RecipientAccepted(<bob@fn.email>)
WHEN   Reset
THEN   TransactionAborted(cause=rset)            → 250 OK
       TransactionState projection ⟶ empty
       SessionState projection     ⟶ unchanged   ← no re-EHLO required
```

This single slice justifies the two-lane split. Implementations that model SMTP
as one flat state machine get this wrong and wrongly demand another `EHLO`.

### 5.3 Responsibility transfer

```
GIVEN  MailTransactionStarted(<alice@acme.com>),
       RecipientAccepted(<bob@fn.email>),
       RecipientAccepted(<carol@other.org>),
       DataPhaseEntered
WHEN   SubmitContent(octets, dot-unstuffed)
THEN   MessageContentReceived(content_ref, actual_octets)
       MessageAccepted(queue_id)                 → 250 OK queued as <queue_id>
       DeliveryRequested(bob@fn.email)
       DeliveryRequested(carol@other.org)
```

### 5.4 Partial failure and the bounce guard (§6.1)

```
GIVEN  MessageAccepted(q1, reverse_path=<alice@acme.com>),
       RelayAccepted(carol@other.org),
       RelayFailed(bob@fn.email, 550)
THEN   BounceMessageGenerated(to=<alice@acme.com>, reverse_path=<>)
```

```
GIVEN  MessageAccepted(q2, reverse_path=<>),
       RelayFailed(...)
THEN   ⊘ nothing — a null reverse-path MUST NOT generate a bounce
```

The guard is a predicate over the timeline, not a flag someone must remember
to check.

---

## 6. Information completeness audit

Every wireframe field traces back through read models to events. Every event
field traces to a command or a prior event. No orphans in either direction.

Run it against the `Received:` header RFC 5321 §4.4 **requires** us to prepend:

| Header clause | Sourced from |
|---|---|
| `FROM` domain | `ClientIdentified.claimed_domain` |
| `FROM` address literal | `ConnectionAccepted.peer_address` |
| `BY` | `ServiceGreetingSent.server_domain` |
| `WITH ESMTPS` | `ClientIdentified.protocol` + `TlsNegotiated` |
| `ID` | `MessageAccepted.queue_id` |
| `FOR` | `RecipientAccepted.forward_path` *(only when exactly one)* |
| timestamp | `MessageAccepted.occurred_at` |

Every clause resolves. Now invert the audit — that is where the value is:

- **Drop `peer_address` from `ConnectionAccepted`** and a conformant `Received`
  header becomes impossible to write. Found on the model, not three weeks into
  implementation.
- **Loop detection (§6.3)** needs a hop count, so `Received` lines must be
  countable — which means trace headers are part of stored content, not
  regenerated.
- **`SIZE` enforcement** needs *both* `MailTransactionStarted.declared_size` and
  `MessageContentReceived.actual_octets`. Declared and actual are different
  facts; conflating them is a classic bug.

---

## 7. Slice boundaries (Conway's law step)

| Boundary | Owns |
|---|---|
| **Edge** | Protocol parsing, session state, reply rendering |
| **Policy** | Relay authorization, recipient validation, size/rate limits |
| **Content Store** | Message blobs — deliberately *not* in the event stream |
| **Delivery** | Queue, retry, bounce generation |
| **DNS** | MX resolution (translation boundary) |

**Message bodies stay out of the event log.** `MessageContentReceived` carries a
`content_ref` and a size; the octets live in the store. Otherwise the timeline
becomes a mail spool and replay becomes impossible.

---

## 8. Open questions

Carried into the next refinement pass:

1. **Are rejections events?** `RecipientRejected` / `MessageRejected` are
   modeled as events here for auditability. The alternative — a command that
   simply fails with a synchronous reply and writes nothing — is more orthodox
   Event Modeling. Which do we want, and does the answer differ for policy
   rejections vs. protocol errors?
2. **Is `ReplyIssued` an event?** Currently replies are treated as a *view*
   (projection). Making every wire reply an event would give a perfect protocol
   audit log at the cost of a very noisy timeline.
3. **Stream boundaries.** One stream per session? Per transaction? Per message
   post-acceptance? The pivot event suggests a handoff between two streams.
4. **`DataPhaseEntered`** — does it earn its place, or is it derivable?
5. **Submission (RFC 6409) vs. relay.** Authenticated submission on :587 has
   different policy but the same protocol shape. One model with a flag, or two?
6. **Pipelining (RFC 2920)** collapses the command round-trips the timeline
   currently assumes one-per-column.
