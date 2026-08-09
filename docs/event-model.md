# FnEmail — Event Model (RFC 5321, inbound, HELO)

Status: **draft v0.4 — reconciled to the v2 walk, 2026-08-08**
Scope: **inbound SMTP, `HELO` only.** No ESMTP, no relay, no outbound.

Method reference: the method repo's
[`EventModeling/docs/`](https://github.com/IvanTheGeek/EventModeling/tree/main/docs); the research
repo's `event-modeling-research:research/METHOD-REFERENCE.md` remains as source material.
Corrections from v0.1: `event-modeling-research:research/CORRECTIONS-v0.1.md`.
Altitude rules — what belongs in this model at all: `model-altitude.md`.
Diagrams (markdown tables, render on GitHub and the Claude apps): `diagrams/`.
Extensions are parked in the method repo — `EventModeling/docs/extensions.md`, moved there
2026-08-08 from this repo — and **not** applied here — this document is orthodox Dymitruk/Dilger
by intent, so extensions can be measured against it.

> ⚠️ **This document is derived.** Paths are the source and slices are derived — a working
> hypothesis, not settled: the method repo's `layering.md` carries the status
> ([`DECISIONS.md`](DECISIONS.md)) — and this file is the union of what the walked paths have
> contributed, reconciled in passes. Reconciled 2026-08-08 to
> [`paths/WORKING-helo-direct-single-recipient-v2.md`](paths/WORKING-helo-direct-single-recipient-v2.md)
> on the owner's waiver of the rule-13 hold, the `WORKING-` prefix not yet dropped. Still open
> after that pass, flagged where each bites below:
>
> - The six path-defined views are absorbed by name only; full contracts wait on the open
>   dataset-cascade and step-8 symmetry rulings
>   ([`paths/EXPLORE-declaration-vs-status.md`](paths/EXPLORE-declaration-vs-status.md)).
> - The entry mechanism for `ServiceConfigured` — two candidate shapes recorded, neither chosen
>   (`paths/EXPLORE-rewalk-cadence.md`, *two shapes, neither chosen*; commit 6ed62cf).
> - The metadata ruling — the walk passed the correlation-scheme question upward; see the ⚠️
>   under *Metadata*.
> - `SessionState`'s fate — pending the consulted-view question; see the slice's ⚠️.
> - The scenario-form questions — the view-`When` form and its missing key
>   ([`paths/EXPLORE-view-slice.md`](paths/EXPLORE-view-slice.md)).
> - H8's resolution — registered under *Hotspots*, blocked on the dataset cascade.

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

Twelve slices survived the scoping pass, and the v2 walk adds six path-defined View Slices —
eighteen slices in all. Every reply is 3-digit only — no enhanced status codes (RFC 3463 is also
an extension).

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

---

## Conventions

**Slice types.** Every slice is a **Command Slice** (one command) or a **View Slice** (one read
model). Never both — a feature needing each is two slices.

Event **orange** · Command **blue** · Read Model **green** · Screen **white** · hotspot **red** ·
external **yellow** (Translation) · wire **black**.
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
| **Transaction** | `ReversePathDeclared`, `RecipientAccepted`, `RecipientRejected`, `DataPhaseEntered`, `MessageAccepted`, `MessageRejected`, `TransactionAborted` |
| **Directory** | ✅ **A separate context**, not a swimlane here — see *H3 resolved* |

---

## The responsibility boundary

RFC 5321 §2.1: *"once the server has issued a success response at the end of the mail data, a
formal handoff of responsibility for the message occurs: the protocol requires that a server MUST
accept responsibility for either delivering the message or properly reporting the failure to do
so"*. §6.1 fixes the moment: the receiver-SMTP accepts *"by sending a "250 OK" message in
response to DATA"*.

`MessageAccepted` records the acceptance decision; obligation transfers when the success reply is
issued. Left of that reply, abandoning costs nothing. Right of it is delivery — out of scope.
Whether the `250` is thereby constitutive — the `MessageQueued` view's `queue_id` copy — is part
of the open dataset cascade the walk leaves standing for a ruling (its notes, and
`FOLLOW-UPS.md`); flagged here, not settled.

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
       OR  reply 554 (the refused session still runs to QUIT; see the correction below)
```

⚠️ **Corrected 2026-08-08.** This contract read *reply 554, no event*, and that is contradicted
by RFC 5321 §3.1: *"A server taking this approach MUST still wait for the client to send a QUIT
(see Section 4.1.1.10) before closing the connection and SHOULD respond to any intervening
commands with "503 bad sequence of commands"."* A refused greeting still has a session, the
session runs to `QUIT`, so `ConnectionAccepted` must exist on the refused path too — found by the
v2 walk's backward pass (its *Hotspots*, *the refused greeting still has a session*). How the
refusal decision itself is recorded is undecided design awaiting the rejection-path walk; that
shape is flagged, not invented here.

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
Post   ClientIdentified{claimed_domain}
       OR  reply 501 (bad syntax) / 500 (unrecognized)
```
`protocol` **removed.** In a `HELO`-only charter it cannot vary — a constant is a rule, not a
fact — and the `WITH` clause's value joined the renderer's constants (commit 2673b1b).

### `SessionState` — V

```
Sources   ConnectionAccepted, ClientIdentified, SessionReset
Answers   Has the client identified itself? Is a transaction open?
Post      SessionState{identified: bool, transaction_open: bool}
```
Placed here because `MailFrom`'s `Pre` consumes it (`Reset`'s does too, below) — not next to its
sources.

⚠️ **Survival flagged, not ruled.** No `Given` in the v2 walk ever consults this view, and its
rendering role went to the path-defined `SessionReady{server_domain}` (commit cf3227d) — which
weakens even the consumers named above. Whether the slice survives, and how any command `Pre`
consults a view at all, is the open consulted-view question; nothing here is deleted until it
rules.

### `MailFrom` — C

```
Pre    SessionState.identified = true
Post   ReversePathDeclared{reverse_path}
       OR  reply 503 (no HELO) / 550 (sender rejected)
```
The event names the client's declaration, not our state change — candidates screened in
[`paths/EXPLORE-declaration-vs-status.md`](paths/EXPLORE-declaration-vs-status.md), which stays
the citation for what that exploration still leaves open.

### `RecipientDirectory` — V  *(translation boundary — H3 resolved)*

```
Sources   RecipientResolved, translated from the Directory context (deferred)
Answers   Is this address a local mailbox?
Post      RecipientDirectory{is_local: bool}
```
**No longer a hole.** H3 asked whether Directory has its own charter. It does — see *H3 resolved*
below — so this is an ordinary **Translation** boundary: the Directory context's events arrive as
external (yellow), are translated into our vocabulary, and this read model projects from the
translated event — the walk seeds it as `RecipientResolved{is_local, forward_path}` — rather than
from foreign ones directly. The event's name is walked fact; how a consulted view is read by a
command's `Pre` stays open (the consulted-view question).

The translation slice itself is deferred with the Directory model. What is *not* deferred is that
the boundary is now typed and orthodox rather than unexplained.

`relay_permitted` **removed.** Relay is out of scope, so it is constant-false — a rule, not a fact.
Under G1 nothing varies and nothing consumes the variation.

### `RcptTo` — C

```
Pre    ReversePathDeclared exists for this session
       RecipientDirectory available
Post   RecipientAccepted{forward_path}
       OR  RecipientRejected{forward_path, reply_code, reason}
```
Walked once per recipient. The same slice, not N slices.

`is_local` **removed** from `RecipientAccepted`. In this scope an accepted recipient is necessarily
local — relay is deferred — so the field is constant-true. A constant is a rule, not a fact.

### `TransactionState` — V

```
Sources   ReversePathDeclared, RecipientAccepted, TransactionAborted
Answers   Did the occasioning event happen?
Post      renders the 250 from the cited event's existence; no dataset
```
The dataset is empty by ruling, not by omission: `open` and `reverse_path` restated the `Given`,
and `recipient_count` encoded position as data (commit cd06274). Empty here is not empty forever
— §3.3 gives both occasions a 550/553 branch, and the rejection walk seats whatever selects the
reply code at the slice layer.

⚠️ **The view's own name is open**: `TransactionState` describes state it no longer carries
([`paths/EXPLORE-declaration-vs-status.md`](paths/EXPLORE-declaration-vs-status.md), *Open, and
deliberately not decided here*). Flagged, not renamed.

### `BeginData` — C 🔴 **H1**

```
Pre    ReversePathDeclared exists
       at least one RecipientAccepted exists for this transaction
Post   DataPhaseEntered
       OR  reply 503 (no recipients)
```
The *command* is sound — it makes a real decision. The *event* is used: the walk gives it two
consumers — `DataPrompt` folds it to render the `354`, and `SubmitContent` declares it as
`Given`. What H1 still holds open is the event's name — see *Hotspots*. The event-existence form
of this `Pre` is safe under the `TransactionState` collapse (commit cd06274); how a `Pre` may
cite a *view* at all is the open consulted-view question, not exercised here.

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

### Path-defined views — absorbed by name; contracts pending

The v2 walk defines six View Slices the scoping pass predates. They are carried here by name,
occasioning event, and rendered output only; full contracts wait on the open dataset-cascade and
step-8 symmetry rulings
([`paths/EXPLORE-declaration-vs-status.md`](paths/EXPLORE-declaration-vs-status.md)).

| View | Occasioned by | Renders |
|:--|:--|:--|
| `ServiceReady` | `ConnectionAccepted` | the `220` greeting |
| `SessionReady` | `ClientIdentified` | the `250` after `HELO` |
| `DataPrompt` | `DataPhaseEntered` | the `354` prompt |
| `MessageTrace` | `MessageAccepted` | the `Received:` line, into the stored message |
| `MessageQueued` | `MessageAccepted` | the `250` after the final dot |
| `SessionClosing` | `SessionClosed` | the `221` closing line |

⚠️ **The open dataset cascade bites four values, flagged rather than settled**:
`DataPrompt.awaiting_content` and `MessageQueued.accepted` — each a boolean nothing renders, over
which the constant test hangs — `MessageQueued`'s `queue_id` copy, and `SessionClosing.cause`,
the deciding case because it is the one non-constant among them. The cascade is tracked in the
walk's notes and `FOLLOW-UPS.md`.

**Two seeded events.** The walk's preamble declares `ServiceConfigured{server_domain,
greeting_text}` and `RecipientResolved{is_local, forward_path}` — path-local placeholders whose
provenance the path rules out of scope. `RecipientResolved` is the Directory boundary's
translated event (see `RecipientDirectory` above).

⚠️ **The entry mechanism for `ServiceConfigured` is undecided** — *two shapes, neither chosen*
(`paths/EXPLORE-rewalk-cadence.md`; commit 6ed62cf): Shape A, configuration is ours — an
operator-lane Command Slice and the model's true origin; Shape B, configuration is someone
else's — a translation from a Configuration context, the `RecipientDirectory` precedent. Neither
is adopted here.

---

## Events

| Event | Fields |
|---|---|
| `ConnectionAccepted` | `peer_address`, `local_address` ⚠️ |
| `ClientIdentified` | `claimed_domain` |
| `SessionReset` | — |
| `SessionClosed` | `cause` (quit \| timeout \| abort \| shutdown) |
| `ReversePathDeclared` | `reverse_path` |
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

⚠️ **The walk contradicts this table, and the ruling is passed upward, not made here.** On the v2
path `session_id` and `correlation_id` both died (commit 0e31109; the walk's *The layering*
block): `correlation_id` duplicates `session_id` in scope, `queue_id` supersedes it beyond, and
no key survives on a path. That leaves three correlation schemes standing here against this
section's own one-mechanism warning. The model-level metadata ruling is open — entangled with
H4's stream/envelope sub-rulings — and the rows stay until it lands.

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

Either give it a consumer or delete it. Two candidate consumers are on record (the walk's
*Hotspots*, H6), and neither is decided:

- **The multi-homed `BY` arm.** On a server listening on several addresses with different
  hostnames, `Received:`'s `BY` clause depends on *which* address was connected to — `BY` would
  source from `local_address` rather than configuration, at the `MessageTrace` view.
- **The nameless-server greeting.** The `220` identity slot admits an address literal (§4.2), so
  a server with no name would render its greeting from `local_address`.

Which arm wins, if either, stays open — the field stays flagged rather than removed.

### No `ServiceGreetingSent`

The `220` banner is not an event. Nothing consumes it: the transcript renders it `derived:` from
`ConnectionAccepted` plus config, and `Received:` takes `BY` from config. Origin but no
destination — it does not earn a place. Deleted by the forward completeness pass, which is the
check doing real work rather than ratifying what was drawn.

---

## Scenarios

**Sequencing — `RCPT` before `MAIL`** (§4.1.1)
```
Slice   RcptTo                            [Command Slice / error]
GIVEN   ConnectionAccepted, ClientIdentified
WHEN    RcptTo(<bob@example.org>)
THEN    error 503 Bad sequence of commands
```

**`RSET` clears the transaction, keeps the session** (§4.1.1.5)
```
Slice   Reset                                     [Command Slice]
GIVEN   ClientIdentified(mail.acme.com),
        ReversePathDeclared(<alice@acme.com>),
        RecipientAccepted(<bob@fn.email>)
WHEN    Reset
THEN    TransactionAborted(cause=rset)                   → 250 OK
```
```
Slice   SessionState                                 [View Slice]
GIVEN   ClientIdentified(mail.acme.com), SessionReset
WHEN    Query(session)
THEN    SessionState{identified: true, transaction_open: false}
```
⚠️ **This scenario is flagged, not normalized.** Its `WHEN Query(session)` form is supported by
only three podcast episodes, it names no key, and the view it queries is the weakened
`SessionState` — flagged in [`paths/EXPLORE-view-slice.md`](paths/EXPLORE-view-slice.md) before
the v2 walk existed. It stands as written until the view-`When` form and consumer-or-key
questions settle.

**Responsibility transfer**
```
Slice   SubmitContent                             [Command Slice]
GIVEN   ReversePathDeclared(<alice@acme.com>),
        RecipientAccepted(<bob@fn.email>),
        RecipientAccepted(<carol@fn.email>),
        DataPhaseEntered
WHEN    SubmitContent(octets, dot-unstuffed)
THEN    MessageAccepted(queue_id, content_ref, actual_octets)
                                       → 250 OK queued as <queue_id>
```

**Relay refused**
```
Slice   RcptTo                                    [Command Slice]
GIVEN   ClientIdentified(unknown.example),
        ReversePathDeclared(<x@elsewhere>),
        RecipientDirectory{is_local: false}
WHEN    RcptTo(<victim@third-party.org>)
THEN    RecipientRejected(reply_code=550, reason="relay not permitted")
```
Modeled as an event, not a bare error — a refused relay attempt is a fact worth keeping.
Contrast the sequencing slip above, which produces none. **That line is H2.**

---

## Completeness check

Completeness closes in **three checks**: backward, payload, forward.

**Backward — `Received:` (§4.4).** Every rendered value needs an origin.

| Clause | Origin |
|---|---|
| `FROM` domain | `ClientIdentified.claimed_domain` |
| `FROM` address literal | `ConnectionAccepted.peer_address` |
| `BY` | config (`derived:`) |
| `WITH SMTP` | renderer constant — cannot vary in a `HELO`-only charter |
| `ID` | `MessageAccepted.queue_id` |
| `FOR` | `RecipientAccepted.forward_path` *(only when exactly one)* |
| timestamp | `MessageAccepted.received_at` — **payload, not metadata** (see *Why `received_at` is payload, not metadata* above) |

**Payload — every emitted event value needs an origin.** The check the other two cannot make:
backward stops at rendered outputs and forward counts consumers, so neither asks where an
*emitted* event's payload values come from. An origin is a command field, a `Given` fold, a
derivation from either, a mint at the emitting step, or a named boundary fact. The v2 walk's
instantiated tables (*Payloads — every emitted value to its origin*) are the worked example.

**Forward — the transcript**

> If the session transcript cannot be reconstructed from the event stream alone, the model is
> incomplete.

Covers the whole session rather than one field set, and exercises the direction that exposes
events nothing consumes. `DataPhaseEntered` lived by exactly this direction — see H1.

---

## Hotspots

**H1 — ✅ answered on existence; open on the name.** The earning question is closed by the v2
walk (its *✅ H1 — verified answered by this walk* block): the event has two consumers on that
page — `DataPrompt` folds it to render the `354`, and `SubmitContent` declares it as `Given`.
The destination gate of `model-altitude.md` §3 decided it, with consumers rather than argument.
What remains open is the event's **name**: `DATA` declares nothing, so the declaration-naming
rule has no purchase, and no candidate has been generated
([`paths/EXPLORE-declaration-vs-status.md`](paths/EXPLORE-declaration-vs-status.md), *Open, and
deliberately not decided here*). The name is not chosen here.

**H2 — Are protocol errors events?** `RecipientRejected` is; a `503` is not. The line drawn is
"policy decisions are facts, protocol slips are not" — defensible, unverified. The corpus answer
is a **size test, not a kind test**: a large sad-path divergence becomes its own workflow to the
right with a cross-reference back, a small one stays a red scenario on the slice
(`event-modeling-research:research/TALKS-FINDINGS.md`). Close kin, also open:
**store-first or validate-first** — the corpus contradicts itself, and this model's
validate-first stance (`ClientIdentified` on success; `501`/`500` produce no event) was assumed
rather than decided (`event-modeling-research:research/TALKS-FINDINGS-CORPUS.md` §3).

**H3 — RESOLVED. Directory is a separate context.** See below.

**H4 — Stream design.** One stream per session, per transaction, or per message? The session
outlives the transaction, which argues they are not one stream. The spoken corpus has nothing on
it — swept 2026-08-06, 982,637 words across both transcript bodies, zero hits — so it is reasoned
out, not looked up. The v2 walk passes four findings and three pending sub-rulings upward; see
its *Hotspots* section, and D.2 (the aborted-transaction scenario) is the designated test for the
transaction question.

**H6 — Is `BY` config or `local_address`?** Decides whether `local_address` is an orphan field or
load-bearing. Two candidate arms carry it (the walk's *Hotspots*): the multi-homed `BY` clause at
the `MessageTrace` view, and the nameless-server greeting — the `220` identity slot admits an
address literal (§4.2). The v2 walk renders the config arm and exercises neither candidate;
which arm wins, if either, stays open.

**H7 — Does FnEmail ever refuse mail entirely?** RFC 7504 `521` applies only to a host that never
accepts mail — a null-MX sentinel or a domain configured to accept none. If FnEmail supports that
mode, `AcceptConnection` gains a `521` branch and the model needs a configuration input saying so.
If not, `521` never appears and this can be closed. A product question.

Note the shape of the rule: RFC 7504 hard-lines *when* `521` is used, then leaves the aftermath to
the operator — after `521` the server **MAY** keep replying `521` or **MAY** just close the
connection. A normative frame with an operator choice inside it.

**H8 — Does `SessionClosed` earn its place?** Registered from `model-altitude.md` Q6, where it
was proposed. The walk gives `SessionClosed.cause` a consumer — `SessionClosing` folds it to
render the `221` — but `SessionClosing.cause` is the deciding case of the open dataset cascade:
if it dies there, `cause` becomes a second flagged-unconsumed value. Resolution is blocked on
that cascade; decide jointly with `Quit`'s postcondition.

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
arrive yellow, get translated into our vocabulary — the walk seeds the translated event as
`RecipientResolved{is_local, forward_path}` — and `RecipientDirectory` projects from it. This is
exactly what the Translation pattern is for.

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
