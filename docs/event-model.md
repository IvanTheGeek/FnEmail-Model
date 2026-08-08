# FnEmail — Event Model (RFC 5321, inbound, HELO)

Status: **draft v0.3 — scoped down to the HELO path**
Scope: **inbound SMTP, `HELO` only.** No ESMTP, no relay, no outbound.

Method reference: `event-modeling-research:research/METHOD-REFERENCE.md`. Corrections from v0.1: `event-modeling-research:research/CORRECTIONS-v0.1.md`.
Altitude rules — what belongs in this model at all: `model-altitude.md`.
Diagrams (markdown tables, render on GitHub and the Claude apps): `diagrams/`.
Extensions are parked in the method repo — `EventModeling/docs/extensions.md`, moved there
2026-08-08 from this repo — and **not** applied here — this document is orthodox Dymitruk/Dilger
by intent, so extensions can be measured against it.

> ⚠️ **This document is derived, and it lags the active walk — deliberately.** Paths are the
> source and slices are derived ([`DECISIONS.md`](DECISIONS.md); canonical statement in the method
> repo's `layering.md`), so this file is the union of what the walked paths have contributed,
> reconciled in passes. The active working stance is
> [`paths/WORKING-helo-direct-single-recipient-v2.md`](paths/WORKING-helo-direct-single-recipient-v2.md),
> and nothing is reconciled to it until the `WORKING-` prefix drops (AGENTS.md rule 13). Known
> pending against this text, listed so it is not lost to commit messages:
>
> - `AcceptConnection`'s contract *reply 554, no event* is contradicted by RFC 5321 §3.1 — a
>   refused greeting still has a session and MUST wait for `QUIT`. Rule 4 correction pending.
> - `ClientIdentified.protocol` fell to the constant test (commit 2673b1b); `WITH`'s value joined
>   the renderer's constants in a `HELO`-only charter.
> - No `Given` in the v2 walk ever consults `SessionState`; its rendering role became
>   `SessionReady{server_domain}` (commit cf3227d).
> - The metadata table carries three correlation schemes against its own one-mechanism warning;
>   `session_id` and `correlation_id` died on the path (commit 0e31109 and the walk's *The
>   layering* block).
> - The `WHEN Query(session)` scenario below ships a form only three podcast episodes support,
>   and it names no key — flagged in [`paths/EXPLORE-view-slice.md`](paths/EXPLORE-view-slice.md)
>   before the v2 walk existed.
> - H1 is answered on the walk's page — `DataPhaseEntered` has two consumers there; formal closure
>   here is pending. H8 (`SessionClosed` — `model-altitude.md` Q6) is proposed but never
>   registered in the hotspot list below.
> - The walk defines six views and two seeded events this document does not yet carry.

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
Editor's entry for 5321 lists exactly one. Both are archived under `rfc/`.

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
- **Terminology.** Command Slice / View Slice replace write column / read column throughout.
- **Metadata section added**, and two payload defects fixed — both found by walking the path with
  real data rather than by reading the model. `MessageAccepted` gained `received_at`;
  `ConnectionAccepted` lost `occurred_at`; `local_address` is flagged as an orphan (H6).

---

## Conventions

**Slice types.** Every slice is a **Command Slice** (one command) or a **View Slice** (one read
model). Never both — a feature needing each is two slices.

Event **orange** · Command **blue** · Read Model **green** · Screen **white** · hotspot **red**.
Command is blue and Read Model is green in every source; Event is orange here (see
`event-modeling-research:research/UPSTREAM-DEFECTS.md` for the one file that disagrees).

**Arrows are not used.** Meaning comes from lane position, left-to-right time, and which pattern a
slice matches. Legal adjacency still binds: `command → event → view → trigger/processor →
command`; **an event may never connect directly to a command.**

---

## Actor swimlanes

| Lane | Kind | Element | Role |
|---|---|---|---|
| **Remote client** | actor role | **command screens** — the terminal exchange | Drives every Command Slice |
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

So the remote client is an ordinary **actor role**, and roles get screens. Each Command Slice carries
a command screen: the prior reply as displayed context, and the line the actor sends.

```
+-----------------------------+   +---------------------------+
| 250 foo.com                 |   | 250 OK                    |
| > MAIL FROM:<Smith@bar.com> |   | > RCPT TO:<Jones@foo.com> |
+-----------------------------+   +---------------------------+
         MailFrom slice                    RcptTo slice
```

Cheap to draw, which matters — *"Drawing a screen must not take longer than 2 minutes."* One screen
per slice per lane, never stacked, per §3.1.

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
| **Directory** | ✅ **A separate context**, not a swimlane here — see *H3 resolved* |

---

## The responsibility boundary

RFC 5321 §2.1: *"when an SMTP server accepts a message, it is accepting responsibility for
delivering or reporting the failure to do so."*

`MessageAccepted` is where obligation begins. Left of it, abandoning costs nothing. Right of it is
delivery — out of scope.

A domain observation about SMTP, **not** Event Modeling vocabulary.

---

## Slices and contracts

**C** = **Command Slice** (exactly one command). **V** = **View Slice** (exactly one read model).

Slice, not column — Adam's own word. Corpus synonyms: a Command Slice is *state change* /
*write column* / `state-change`; a View Slice is *state view* / *read model* / *query* /
`state-view`. Quotations from the sources keep their own terms.

### `AcceptConnection` — C

```
Pre    none
Post   ConnectionAccepted{peer_address, local_address}
       OR  reply 554, no event
```
Exists to carry `peer_address`, which the `Received:` header requires and nothing else supplies.
That one sentence is the whole justification for the slice, and it is worth spelling out, because
the shape of the argument is not the shape of the answer — see **H5** below.

**`554` here is correct, and must not be "upgraded" to `521`.** RFC 7504 §3 reserves `521` for a
host that *"does not accept mail under any circumstances"* — a dummy server whose only job is to
say so. It *"SHOULD NOT be used for situations in which the server rejects mail from particular
hosts or addresses"*, which is exactly our blocklist case. `554` remains right. See H7 for the
case where `521` would apply.

### `Helo` — C

```
Pre    ConnectionAccepted exists
Post   ClientIdentified{claimed_domain, protocol:"SMTP"}
       OR  reply 501 (bad syntax) / 500 (unrecognized)
```

### `SessionState` — V

```
Sources   ConnectionAccepted, ClientIdentified, SessionReset
Answers   Has the client identified itself? Is a transaction open?
Post      SessionState{identified: bool, transaction_open: bool}
```
Placed here because `MailFrom`, `RcptTo` and `BeginData` consume it — not next to its sources.

### `MailFrom` — C

```
Pre    SessionState.identified = true
Post   MailTransactionStarted{reverse_path}
       OR  reply 503 (no HELO) / 550 (sender rejected)
```

### `RecipientDirectory` — V  *(translation boundary — H3 resolved)*

```
Sources   translated events from the Directory context (deferred)
Answers   Is this address a local mailbox?
Post      RecipientDirectory{is_local: bool}
```
**No longer a hole.** H3 asked whether Directory has its own charter. It does — see *H3 resolved*
below — so this is an ordinary **Translation** boundary: the Directory context's events arrive as
external (yellow), are translated into our vocabulary, and this read model projects from the
translated events rather than from foreign ones directly.

The translation slice itself is deferred with the Directory model. What is *not* deferred is that
the boundary is now typed and orthodox rather than unexplained.

`relay_permitted` **removed.** Relay is out of scope, so it is constant-false — a rule, not a fact.
Under G1 nothing varies and nothing consumes the variation.

### `RcptTo` — C

```
Pre    MailTransactionStarted exists for this session
       RecipientDirectory available
Post   RecipientAccepted{forward_path}
       OR  RecipientRejected{forward_path, reply_code, reason}
```
Walked once per recipient. The same slice, not N slices.

`is_local` **removed** from `RecipientAccepted`. In this scope an accepted recipient is necessarily
local — relay is deferred — so the field is constant-true. A constant is a rule, not a fact.

### `TransactionState` — V

```
Sources   MailTransactionStarted, RecipientAccepted, TransactionAborted
Answers   Open? Reverse path? How many recipients?
Post      TransactionState{open: bool, reverse_path, recipient_count}
```

### `BeginData` — C 🔴 **H1**

```
Pre    TransactionState.open = true
       TransactionState.recipient_count >= 1
Post   DataPhaseEntered
       OR  reply 503 (no recipients)
```
The *command* is sound — it makes a real decision. The *event* is on trial: its only candidate
consumer is the transcript rendering `354`.

### `SubmitContent` — C

```
Pre    DataPhaseEntered for the current transaction
Post   MessageAccepted{queue_id, reverse_path, recipients[], content_ref,
                      actual_octets, received_at}
       OR  MessageRejected{reply_code, reason}
```
**One command, one event.** v0.1 emitted three event types plus a per-recipient fan-out — the
"left chair". Inbound scope removed the fan-out; the forward completeness pass removed
`MessageContentReceived` as redundant.

### `Reset` — C

```
Pre    SessionState.identified = true
Post   TransactionAborted{cause:"rset"}
       Postcondition explicitly does NOT touch SessionState
```

### `Quit` — C

```
Pre    ConnectionAccepted exists
Post   SessionClosed{cause}
```

### `SessionTranscript` — V

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
| `RecipientAccepted` | `forward_path` |
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
identifying the current step. Other frameworks invert this — the same two ideas ship under
"Correlation ID" and "Trace ID" with the meanings swapped. **Pick names deliberately and define
them here**, because the words alone will not tell a reader which convention is in force.

For this model the RFC settles it: `Received:`'s **`ID` clause** is the trace identifier that
propagates onto the message, and `queue_id` is what goes in it. Name the propagating one after the
RFC rather than after any framework.

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

### Case sensitivity — settled from the RFC

**Question:** if FnEmail receives `helo bar.com`, or `HeLo bar.com`, is that valid?

**Answer: valid, and rejecting it would put FnEmail in violation.** RFC 5321 §2.4 is explicit:

> *"Verbs and argument values (e.g., "TO:" or "to:" in the RCPT command and extension name
> keywords) are **not case sensitive**, with the sole exception in this specification of a mailbox
> local-part … a command verb, an argument value other than a mailbox local-part, and free form
> text **MAY be encoded in upper case, lower case, or any mixture of upper and lower case with no
> impact on its meaning**."*

The specification anticipates servers that get this wrong and names them:

> *"A few SMTP servers, **in violation of this specification** (and RFC 821) require that command
> verbs be encoded by clients in upper case."*

So `helo`, `HELO` and `HeLo` are the same command. There is no reply code for wrong-case verbs
because there is no such thing.

**Three consequences for this model.**

**1. Verb case is not a fact.** The `Helo` slice is the same command whatever case arrived, so there
is no `claimed_verb_case` field and there must never be one. The case that reached the wire fails
**G1** — nothing consumes it — and fails **G2**, since it changes nothing about what the system will
accept next. It is not low-altitude detail; it is *not information*.

**2. `claimed_domain` is case-insensitive**, because §2.4 puts mailbox domains under
*"normal DNS rules"*. `BAR.COM` and `bar.com` are the same claim. The model may normalize it, and
any comparison must not be case-sensitive.

**3. ⚠️ Local-parts MUST preserve case, and this is a real constraint the model was not recording.**

> *"The local-part of a mailbox **MUST BE** treated as case sensitive. Therefore, SMTP
> implementations **MUST take care to preserve the case** of mailbox local-parts. In particular, for
> some hosts, the user "smith" is different from the user "Smith"."*

That is a **MUST**, so by **G3** it is **domain**, not product. It binds every field carrying a
mailbox: `reverse_path`, `forward_path`, and `recipients` on `MessageAccepted`. Those values may not
be case-folded on the way in, on the way out, or in any read model that compares them. It also
propagates into output — `Received:`'s `FOR` clause carries `forward_path` verbatim.

The walked paths already exercise this without having been designed to: `Smith`, `Jones`, `JQP`,
`Green` and `Brown` are all mixed-case local-parts, so any implementation that folded case would
produce visibly wrong `Received:` headers. That is rule 8 working — real data catching a constraint
nobody had written down.

Note the asymmetry inside one address. In `<Jones@XYZ.COM>` from
[`paths/helo-single-recipient.md`](paths/helo-single-recipient.md), **`Jones` is case-sensitive and
`XYZ.COM` is not** — the same string, two different rules, split at the `@`.

⚠️ **One caution the RFC adds against itself:** *"exploiting the case sensitivity of mailbox
local-parts impedes interoperability and is **discouraged**."* So the rule is *preserve*, not
*depend on* — MUST-preserve paired with SHOULD-NOT-rely-upon.

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
Modeled as an event, not a bare error — a refused relay attempt is a fact worth keeping.
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

**H3 — RESOLVED. Directory is a separate context.** See below.

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

**H5 — `AcceptConnection` altitude.** *Domain fact or infrastructure noise?* The question is
malformed, and answering it properly reclassifies the event.

**A TCP accept is infrastructure by G4.** It survives no reimplementation as a *fact* — only as an
event in the operating system's sense. Nothing about accepting a connection is about being a
conformant mail server; it is about how we happen to run one. On shape alone it does not belong.

**It is in the model for exactly one reason: `peer_address` has a destination.** RFC 5321 §4.4
requires trace information, and its FROM clause is where the address literal goes. That is **G1**,
destination — and G1 is unforgiving enough that it already deleted `ServiceGreetingSent`. **If the
`Received:` requirement vanished, this event goes the same way.**

So the answer is not a judgement call between two labels. It is:

> **Infrastructure in shape, admitted by consumer.**

⚠️ **Corrected 2026-08-06 — and the correction changes the tier.** An earlier reading had §4.4
*mandating* the address literal, which would make `peer_address` domain by **G3**. It does not.
Read the clause precisely:

> *"The FROM clause, which **MUST** be supplied in an SMTP environment, **SHOULD** contain both
> (1) the name of the source host as presented in the EHLO command and (2) an address literal
> containing the IP address of the source, determined from the TCP connection."*

**The clause is MUST; its contents are SHOULD.** A server that supplies a FROM clause carrying only
the EHLO name is conformant. By G3 — *MUST → domain, MAY/SHOULD → product* — `peer_address` is
therefore **product, not domain**, and `ConnectionAccepted` is a **Product-tier** event rather than
the "Domain (fragile)" it was classified as in `model-altitude.md` §3 before that table took the
same correction.

**That explains the fragility.** The model kept flagging this event as uncomfortable while calling
it domain. It was not fragile domain; it was product misfiled as domain. Product facts are expected
to churn, which is exactly the instability that kept being noticed.

**One detail is not a coincidence.** RFC 5321 says *transmission channel* throughout and abstracts
the transport — except here, where it says the address literal is *"determined from the **TCP
connection**."* The single place the specification names the transport is the single place this
model admits a transport fact. The RFC is marking where the layer boundary leaks, and the model
followed it without knowing that was why.

**What remains open.** Not the classification, which is now settled, but the consequence: as a
product fact `peer_address` may be dropped by a conformant reimplementation, and if it were, this
slice would carry nothing. Whether FnEmail *commits* to emitting the address literal is a product
decision nobody has made — and it is the same decision as **H7**. If FnEmail ever refuses at
connection level, that refusal is a product rule that needs `peer_address`, and both hotspots close
together. If it never refuses, this event survives on a SHOULD alone.

**Transport itself stays unmodeled.** Not black-boxed — there is no stream standing in for it —
simply outside the charter. One fact is lifted across the boundary because something inside
consumes it. `local_address` is the control: same origin, no consumer, orphaned (**H6**).

---

## H3 resolved — Directory is a separate context

The question was whether `RecipientDirectory` names a *layer* inside FnEmail's charter or a
separate context. Applying the criterion in
`event-modeling-research:research/model-altitude-theory.md` §2.0c *(that section moved out with
the 2026-08-06 split; this reference had not caught up)* — subdivide within a context
and you have layers; subdivide until the pieces have separate charters and owners and you have
separate contexts:

| | Inbound SMTP | Directory |
|---|---|---|
| **Actors** | remote MTA; operator | administrator provisioning mailboxes |
| **Lifecycle** | a session lasts seconds | a mailbox lasts years |
| **Events** | per-connection | `MailboxProvisioned`, `AliasCreated`, `DomainAdded` |
| **Normative base** | RFC 5321 + 7504 | RFC constrains address *syntax* (§4.1.2) only; **who may hold a mailbox is pure operator policy** |
| **Changes when** | the RFC set changes — once since 2008 | operator policy changes |
| **Exists independently?** | no — needs Directory | **yes** — exists whether or not mail ever arrives |

Different actors, different timescales, a different normative source, a different change cadence,
and independent existence. **Separate context.**

### Three consequences

**1. The interface is orthodox Translation.** Directory's events are external to this model. They
arrive yellow, get translated into our vocabulary, and `RecipientDirectory` projects from the
translated events. This is exactly what the Translation pattern is for.

**2. It does not motivate the multiple-models extension.** An earlier note called H3 *"the
strongest pull toward the multiple-models idea"* in `event-model-extensions.md` §1 (now
`EventModeling/docs/extensions.md`). That was wrong.
§1 concerns splitting **one** context into layers — protocol, domain, operational. Directory is a
**different context**, which orthodox Event Modeling already handles. H3 needs no extension.

**3. Two fields disappear.** Working the boundary removed `relay_permitted` from
`RecipientDirectory` and `is_local` from `RecipientAccepted`. Both were constants once relay left
scope, and a constant is a rule rather than a fact.

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
