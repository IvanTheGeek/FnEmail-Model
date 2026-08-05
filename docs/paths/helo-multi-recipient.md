# Path — HELO, multi-recipient, one rejection

The first worked path through `../event-model.md` v0.3, with concrete example data.

Modelled on the RFC 5321 Appendix D.1 exchange with `HELO` substituted for `EHLO`.
⚠️ The RFC data is reproduced from knowledge of the specification and **has not been checked
against the published text** — `rfc-editor.org` is blocked by this environment's egress policy.
Verify before promoting this to a conformance test.

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

### Step 1 · `AcceptConnection` — W

```
Pre     none  ✓
Wire    S: 220 foo.com Simple Mail Transfer Service Ready
Post    ConnectionAccepted{ peer_address: "198.51.100.25",
                            local_address: "192.0.2.10:25" }
```
The `220` is `derived:` from `ConnectionAccepted` plus config — not an event.

### Step 2 · `Helo` — W

```
Pre     ConnectionAccepted exists  ✓  (step 1)
Wire    C: HELO bar.com
        S: 250 foo.com
Post    ClientIdentified{ claimed_domain: "bar.com", protocol: "SMTP" }
```

### Step 3 · `SessionState` — R

```
Sources ConnectionAccepted, ClientIdentified
Wire    (none — read columns are invisible on the wire)
Post    SessionState{ identified: true, transaction_open: false }
```

### Step 4 · `MailFrom` — W

```
Pre     SessionState.identified = true  ✓  (step 3)
Wire    C: MAIL FROM:<Smith@bar.com>
        S: 250 OK
Post    MailTransactionStarted{ reverse_path: "<Smith@bar.com>" }
```

### Step 5 · `RecipientDirectory` — R 🔴

```
Sources ⚠️ NONE — the chain breaks here (H3)
Needs   { is_local: true, relay_permitted: … } for each forward_path
Post    RecipientDirectory{ is_local: true, relay_permitted: false }
```
Every accept path crosses this hole three steps in. Typed so it fails loudly.

### Steps 6a–6c · `RcptTo` — W, walked three times

```
6a  Pre   MailTransactionStarted ✓ · RecipientDirectory ⚠️
    Wire  C: RCPT TO:<Jones@foo.com>          S: 250 OK
    Post  RecipientAccepted{ <Jones@foo.com>, is_local: true }

6b  Wire  C: RCPT TO:<Green@foo.com>          S: 550 No such user here
    Post  RecipientRejected{ <Green@foo.com>, reply_code: 550,
                             reason: "No such user here" }

6c  Wire  C: RCPT TO:<Brown@foo.com>          S: 250 OK
    Post  RecipientAccepted{ <Brown@foo.com>, is_local: true }
```
One column, three steps, three payloads. Dymitruk: *"a workflow step is considered to be repeated
on the event model if it uses the same command or view."*

The rejection is **not** fatal — the session continues.

### Step 7 · `TransactionState` — R

```
Sources MailTransactionStarted, RecipientAccepted ×2
Post    TransactionState{ open: true,
                          reverse_path: "<Smith@bar.com>",
                          recipient_count: 2 }
```
`RecipientRejected` is deliberately not a source — the count is of *accepted* recipients. The
rejection reaches the operator through `SessionTranscript` instead.

### Step 8 · `BeginData` — W 🔴

```
Pre     TransactionState.open ✓ · recipient_count >= 1 ✓ (= 2)
Wire    C: DATA
        S: 354 Start mail input; end with <CRLF>.<CRLF>
Post    DataPhaseEntered
```

### Step 9 · `SubmitContent` — W

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

### Step 10 · `Quit` — W

```
Pre     ConnectionAccepted exists  ✓
Wire    C: QUIT
        S: 221 foo.com Service closing transmission channel
Post    SessionClosed{ cause: "quit" }
```

---

## Accounting

**13 steps over 11 distinct columns.** `Reset` never fires — it needs its own path (D.2).

| Column | Steps |
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

**H3 is not abstract.** In prose it is a footnote; in a path it is a hard stop at step 5, and
every accept path crosses it.

**The RFC's own happy path contains a rejection.** `Green` gets `550` mid-flow and the session
continues. So the "golden path" is not error-free — which complicates the golden-path-plus-unique-
branches promotion rule in `../event-model-extensions.md` §3, because the default already contains
an error branch.

**`FOR` was omitted, correctly, and only by accident of this data.** Two recipients means the
single-recipient branch is never exercised here. One golden path is not coverage.

---

## Next paths

| Path | Exercises |
|---|---|
| **D.2 — aborted transaction** | `Reset`, the only uncovered column |
| **Single recipient** | the `FOR` clause branch |
| **No HELO** | `503` sequencing errors |
| **All recipients rejected** | `BeginData`'s `recipient_count >= 1` precondition failing |
