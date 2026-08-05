# Path — HELO, multi-recipient, one rejection

The first worked path through `../event-model.md` v0.3, with concrete example data.

Modelled on the RFC 5321 Appendix D.1 exchange with `HELO` substituted for `EHLO`.

✅ **Verified against the RFC text** (2026-08-05). `rfc-editor.org` is blocked by the egress
policy, but the specification is mirrored on GitHub, which is reachable — archived at
[`../research/archive/rfc/rfc5321.txt`](../research/archive/rfc/rfc5321.txt). D.1's dialogue
matches this walk exactly: Jones `250`, Green `550 No such user here`, Brown `250`.

⚠️ **One substitution remains unjustified by the RFC.** D.1 uses `EHLO` and advertises
`8BITMIME`, `SIZE`, `DSN` and `HELP`. Substituting `HELO` is our scoping decision, not
something the RFC demonstrates — see *On HELO* below.

**Status:** worked example. Example data is orthodox — Dymitruk: *"The more realistic the data the
better."* The *step/path* framing anticipates the extension parked in
`../event-model-extensions.md` §3 and is not yet part of the method as its authors describe it.

---

## Scene

| | |
|---|---|
| Server | `foo.com` at `192.0.2.10:25` |
| Client | `bar.com` at `198.51.100.25` |
| Time | `2026-08-05T14:23:01Z` |
| Sender | `<Smith@bar.com>` |
| Recipients | `<Jones@foo.com>` ✓ · `<Green@foo.com>` ✗ · `<Brown@foo.com>` ✓ |

Ambient metadata on every event: `session_id`, `correlation_id` (the session),
`causation_id` (the verb that produced it).

---

## The walk

### Step 1 · `AcceptConnection` — C

```
Pre     none  ✓
Wire    S: 220 foo.com Simple Mail Transfer Service Ready
Post    ConnectionAccepted{ peer_address: "198.51.100.25",
                            local_address: "192.0.2.10:25" }
```
The `220` is `derived:` from `ConnectionAccepted` plus config — not an event.

### Step 2 · `Helo` — C

```
Pre     ConnectionAccepted exists  ✓  (step 1)
Wire    C: HELO bar.com
        S: 250 foo.com
Post    ClientIdentified{ claimed_domain: "bar.com", protocol: "SMTP" }
```

### Step 3 · `SessionState` — V

```
Sources ConnectionAccepted, ClientIdentified
Wire    (none — View Slices are invisible on the wire)
Post    SessionState{ identified: true, transaction_open: false }
```

### Step 4 · `MailFrom` — C

```
Pre     SessionState.identified = true  ✓  (step 3)
Wire    C: MAIL FROM:<Smith@bar.com>
        S: 250 OK
Post    MailTransactionStarted{ reverse_path: "<Smith@bar.com>" }
```

### Step 5 · `RecipientDirectory` — V  *(translation boundary)*

```
Sources translated events from the Directory context (deferred)
Needs   is this address a local mailbox?
Post    RecipientDirectory{ is_local: true }
```
**H3 is now resolved** — Directory is a separate context, so this is an ordinary Translation
boundary rather than a hole. The translation slice is deferred with the Directory model, but the
boundary is typed and orthodox. `relay_permitted` removed: relay is out of scope, so it was
constant-false.

### Steps 6a–6c · `RcptTo` — C, walked three times

```
6a  Pre   MailTransactionStarted ✓ · RecipientDirectory ✓
    Wire  C: RCPT TO:<Jones@foo.com>          S: 250 OK
    Post  RecipientAccepted{ <Jones@foo.com> }

6b  Wire  C: RCPT TO:<Green@foo.com>          S: 550 No such user here
    Post  RecipientRejected{ <Green@foo.com>, reply_code: 550,
                             reason: "No such user here" }

6c  Wire  C: RCPT TO:<Brown@foo.com>          S: 250 OK
    Post  RecipientAccepted{ <Brown@foo.com> }
```
One slice, three steps, three payloads. Dymitruk: *"a workflow step is considered to be repeated
on the event model if it uses the same command or view."*

The rejection is **not** fatal — the session continues.

### Step 7 · `TransactionState` — V

```
Sources MailTransactionStarted, RecipientAccepted ×2
Post    TransactionState{ open: true,
                          reverse_path: "<Smith@bar.com>",
                          recipient_count: 2 }
```
`RecipientRejected` is deliberately not a source — the count is of *accepted* recipients. The
rejection reaches the operator through `SessionTranscript` instead.

### Step 8 · `BeginData` — C 🔴

```
Pre     TransactionState.open ✓ · recipient_count >= 1 ✓ (= 2)
Wire    C: DATA
        S: 354 Start mail input; end with <CRLF>.<CRLF>
Post    DataPhaseEntered
```

### Step 9 · `SubmitContent` — C

```
Pre     DataPhaseEntered ✓  (step 8)
Wire    C: Blah blah blah...
        C: .
        S: 250 OK
Post    MessageAccepted{ queue_id: "q7F3A21",
                         reverse_path: "<Smith@bar.com>",
                         recipients: ["<Jones@foo.com>", "<Brown@foo.com>"],
                         content_ref: "blob:sha256:9f2c…",
                         actual_octets: 412,
                         received_at: "2026-08-05T14:23:01Z" }
```
The responsibility boundary. One command, one event.

### Step 10 · `Quit` — C

```
Pre     ConnectionAccepted exists  ✓
Wire    C: QUIT
        S: 221 foo.com Service closing transmission channel
Post    SessionClosed{ cause: "quit" }
```

---

## Accounting

**13 steps over 11 distinct slices.** `Reset` never fires — it needs its own path (D.2).

| Slice | Steps |
|---|---|
| `RcptTo` | 3 |
| all others walked | 1 each |
| `Reset` | 0 — **uncovered** |

---

## Completeness, instantiated

### Backward — the `Received:` header

```
Received: from bar.com ([198.51.100.25])
          by foo.com with SMTP
          id q7F3A21;
          Wed, 05 Aug 2026 14:23:01 +0000
```

| Clause | Value | Source |
|---|---|---|
| `FROM` domain | `bar.com` | `ClientIdentified.claimed_domain` |
| address literal | `198.51.100.25` | `ConnectionAccepted.peer_address` |
| `BY` | `foo.com` | config — but see H6 |
| `WITH` | `SMTP` | `ClientIdentified.protocol` |
| `ID` | `q7F3A21` | `MessageAccepted.queue_id` |
| `FOR` | *omitted* | two recipients — rule fires correctly |
| timestamp | `2026-08-05T14:23:01Z` | `MessageAccepted.received_at` |

### Forward — the transcript

Every wire line above is reconstructible from the events plus config. Steps 3, 5 and 7 produce no
wire output, which is expected: read models are projections, so if every event is reconstructible
the read models are too. The transcript therefore tests the write half directly and the read half
only by implication.

---

## What this walk found

Both were invisible on paper and appeared the moment real data was substituted. This is the
strongest available argument for example-data-first modelling.

**1. `MessageAccepted` had no timestamp.** The `Received:` mapping pointed at
`MessageAccepted.occurred_at`, which did not exist — only `ConnectionAccepted` carried a
timestamp. Fixed by adding `received_at` as **payload**, on the grounds that it must survive
replay unchanged (see the model's metadata section).

**2. `local_address` is an orphan.** Origin but no destination — `BY` comes from config. Kept and
flagged as **H6**: on a multi-homed server with per-address hostnames, `BY` would source from it.

## What this walk taught

**H3 was not abstract — and walking it is what forced the resolution.** In prose it was a
footnote; in a path it was a hard stop at step 5 that every accept path crossed. That pressure is
what got it resolved: Directory turned out to be a separate context, the boundary is orthodox
Translation, and two constant fields disappeared with it.

**The scenario titled "typical" contains a rejection** — but it is not the only one on offer.
Corrected after checking the RFC:

| Scenario | Recipients | Errors | Clean? |
|---|---|---|---|
| D.1 *A Typical SMTP Transaction* | 3 | `550` on Green | no |
| D.2 *Aborted SMTP Transaction* | 2 | `550` on Green, then `RSET` | no |
| **D.3 step 1** *Relayed Mail — source to relay* | **1** | none | **yes** (relay) |
| **D.3 step 2** *Relayed Mail — relay to destination* | **1** | none | **yes** (local) |
| D.4 *Verifying and Sending* | 1 | none | yes, but uses `VRFY` |

So a clean single-recipient happy path does exist. **D.3 step 2 is the cleanest inbound one** —
the destination host receiving mail for a local mailbox, one `RCPT TO`, no errors. D.1 is better
understood as the *multi-recipient with rejection* case.

⚠️ An earlier draft called D.3 step 2 "the true golden path." That was an overclaim — see
*Happy is not golden* below. Being clean does not make a path golden; golden is assigned, not
observed.

What survives for the promotion rule: the scenario the RFC labels "typical" is not the clean one,
so a designation scheme cannot simply take the source's word for which path is canonical.

**`FOR` was omitted, correctly, and only by accident of this data.** Two recipients means the
single-recipient branch is never exercised here. One path is not coverage — now covered by
[`helo-single-recipient.md`](helo-single-recipient.md), which emits it.

---

## On HELO

**No example dialogue in RFC 5321 uses `HELO`.** Five occurrences of `C: EHLO`, zero of `C: HELO`.

Consequence for `../event-model-extensions.md` §3: the claim that "the RFC ships the paths" holds
for the *shape* of each scenario but not verbatim under our HELO-only scope. Every path needs the
greeting exchange substituted, which means these are RFC-**derived** paths, not RFC-**quoted**
ones. Promoting them to conformance tests requires saying which.

## On the `Received:` header

The RFC contains exactly **one** worked `Received:` example, in D.3 step 2:

```
Received: from bar.com by foo.com ; Thu, 21 May 1998 05:33:29 -0700
```

Minimal — no address literal, no `with`, no `id`, no `for`. Our completeness table builds a much
fuller header from §4.4's grammar. Both are legal; the extra clauses are optional. Worth deciding
deliberately how much we emit rather than inheriting the maximal form by default.

## Happy is not golden

Two different things, and this document previously ran them together.

| | Meaning | Kind |
|---|---|---|
| **Happy path** | reaches its intended successful outcome | **descriptive** — a property of the path; many can qualify |
| **Golden path** | the *designated* canonical one: coverage baseline, viewer default, what documentation leads with | **a designation** — assigned, with consequences |

Being clean does not make a path golden. **No golden path is designated yet**, deliberately: if
golden is the coverage baseline, choosing it determines what counts as an "edge", and choosing
early mislabels every edge that follows.

### And "happy" needs an actor

This walk is not unambiguously happy either:

| Actor | Outcome |
|---|---|
| The server | **complete success** — it did exactly what it should |
| `Smith@bar.com` (sender) | **partial** — two of three recipients |
| `Jones`, `Brown` | success |
| `Green` | failure |

Outcome is relative to an actor, the same way screens are. The honest label for this path is
**happy for the server, partial for the sender**. Whether a path's outcome should be recorded per
actor lane rather than as a single verdict is open — see `../event-model-extensions.md` §3.

## Next paths

| Path | Exercises | Source |
|---|---|---|
| ~~D.3 step 2 — single local recipient~~ | ✅ **walked** — [`helo-single-recipient.md`](helo-single-recipient.md) | RFC, derived |
| **D.2 — aborted transaction** | `Reset`, the only uncovered slice | RFC, derived |
| **No HELO** | `503` sequencing errors | ours |
| **All recipients rejected** | `BeginData`'s `recipient_count >= 1` failing | ours |
