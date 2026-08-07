# Path — HELO, direct, single recipient, clean

The default illustration. One originating host, one recipient, no errors, no relay hop. If you are
showing someone the model for the first time, show them this.

Derived from **RFC 5321 Appendix D.1** — *A Typical SMTP Transaction Scenario* — reduced to one
recipient, with `HELO` substituted for `EHLO`. Verified against
[`../rfc/rfc5321.txt`](../rfc/rfc5321.txt).

⚠️ **The dialogue is ours, the topology is the RFC's.** D.1 sends to three recipients and rejects
one; no appendix scenario is *both* direct and single-recipient. Removing Green and Brown from D.1
is a reduction, not a quotation — so this walk is RFC-derived in structure and RFC-conformant in
every reply code, but the exchange as written does not appear in the specification. Stated plainly
because the other two paths *are* quotable and this one is not.

Steps follow [`STEP-FORM.md`](STEP-FORM.md).

---

## Scene

| | |
|:--|:--|
| Server (us) | `foo.com` at `192.0.2.10:25` |
| Client | `bar.com` at `203.0.113.20` — **the originating host, contacting us directly** |
| Sender | `<Smith@bar.com>` — **same domain as the client** |
| Recipient | `<Jones@foo.com>` — one, local |
| Time | `Tue, 19 May 1998 09:14:07 -0700` |

**The alignment is the point.** `claimed_domain` and the `reverse_path` domain are both `bar.com`.
[`helo-single-recipient.md`](helo-single-recipient.md) deliberately made them differ; this path is
the other half of that pair. See *What this walk tested*.

---

## The walk

| 🟦 C · Step 1 | `AcceptConnection` |
|:--|:--|
| MTA Client | ⬛ `220` foo.com Simple Mail Transfer Service Ready |
| | 🟦 **AcceptConnection**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |
| Event | 🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |
| Given | 🟤 |

| 🟦 C · Step 2 | `Helo` |
|:--|:--|
| MTA Client | ⬛ `HELO` bar.com&#10;<br>`250` foo.com |
| | 🟦 **Helo**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Event | 🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Given | 🟧 **ConnectionAccepted** |

| 🟩 V · Step 3 | `SessionState` |
|:--|:--|
| Consumed by | ⬜ `MailFrom` · `RcptTo` · `BeginData` |
| | 🟩 **SessionState**&#10;<br>&nbsp;&nbsp;`identified`: true&#10;<br>&nbsp;&nbsp;`transaction_open`: false |
| Sources | 🟧 **ConnectionAccepted** · **ClientIdentified** |

| 🟦 C · Step 4 | `MailFrom` |
|:--|:--|
| MTA Client | ⬛ `MAIL FROM:`\<Smith@bar.com>&#10;<br>`250` OK |
| | 🟦 **MailFrom**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com> |
| Event | 🟧 **MailTransactionStarted**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com> |
| Given | 🟧 **ClientIdentified** |

| 🟩 V · Step 5 | `RecipientDirectory` &nbsp;*(translation boundary — H3)* |
|:--|:--|
| Consumed by | ⬜ `RcptTo` |
| | 🟩 **RecipientDirectory**&#10;<br>&nbsp;&nbsp;`is_local`: true |
| Sources | 🟨 translated from the **Directory context** — external, deferred |

The 🟨 source is what makes this Translation rather than Automation: the events belong to another
context. Deferred, so the value is asserted rather than derived — the same typed hole both other
paths cross.

| 🟦 C · Step 6 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com>&#10;<br>`250` OK |
| | 🟦 **RcptTo**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Given | 🟧 **MailTransactionStarted** |
| | 🟨 **Directory translation** — *no event of ours* |

Traversed **once**. No `RecipientRejected` anywhere in this walk.

| 🟩 V · Step 7 | `TransactionState` |
|:--|:--|
| Consumed by | ⬜ `RcptTo` · `BeginData` · `SubmitContent` |
| | 🟩 **TransactionState**&#10;<br>&nbsp;&nbsp;`open`: true&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipient_count`: 1 |
| Sources | 🟧 **MailTransactionStarted** · **RecipientAccepted** |

| 🟦 C · Step 8 | `BeginData` &nbsp;🟥 **H1** |
|:--|:--|
| MTA Client | ⬛ `DATA`&#10;<br>`354` Start mail input; end with `<CRLF>.<CRLF>` |
| | 🟦 **BeginData**&#10;<br>&nbsp;&nbsp;🟤 *the verb carries no data* |
| Event | 🟥 **DataPhaseEntered**&#10;<br>&nbsp;&nbsp;*no payload — H1: does it earn its place?* |
| Given | 🟧 **MailTransactionStarted** |
| | 🟧 **RecipientAccepted** |

**H1 meets you here.** Its only candidate consumer is the transcript rendering `354`. This walk
does not resolve it — it just shows the event firing with nothing downstream that needs it.

| 🟦 C · Step 9 | `SubmitContent` |
|:--|:--|
| MTA Client | ⬛ `Date:` Tue, 19 May 1998 09:14:02 -0700&#10;<br>`From:` Smith \<Smith@bar.com>&#10;<br>`To:` Jones@foo.com&#10;<br>`Subject:` Tuesday&#10;<br>(blank)&#10;<br>Blah blah blah...&#10;<br>`.`&#10;<br>`250` OK |
| | 🟦 **SubmitContent**&#10;<br>&nbsp;&nbsp;`content`: 194 octets, dot-unstuffed |
| Event | 🟧 **MessageAccepted**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipients`: [\<Jones@foo.com>]&#10;<br>&nbsp;&nbsp;`content_ref`: blob:sha256:9c1e…&#10;<br>&nbsp;&nbsp;`actual_octets`: 194&#10;<br>&nbsp;&nbsp;`received_at`: 1998-05-19T09:14:07-07:00 |
| Given | 🟧 **DataPhaseEntered** |

Left of here, abandoning costs nothing. At `MessageAccepted` we have accepted responsibility for
delivering or reporting failure — RFC 5321 §2.1. **That is the responsibility boundary.**

| 🟦 C · Step 10 | `Quit` |
|:--|:--|
| MTA Client | ⬛ `QUIT`&#10;<br>`221` foo.com Service closing transmission channel |
| | 🟦 **Quit**&#10;<br>&nbsp;&nbsp;🟤 *the verb carries no data* |
| Event | 🟧 **SessionClosed**&#10;<br>&nbsp;&nbsp;`cause`: quit |
| Given | 🟧 **ConnectionAccepted** |

Every reply is 2xx or 3xx. No error branch is taken anywhere.

---

## Accounting

**10 steps over 10 distinct slices** — one step per slice, no repeats. Fewer steps than either
other path, because nothing is traversed twice and nothing is rejected.

| | This path | Direct, multi | Relay, single | Combined |
|:--|:--|:--|:--|:--|
| Steps | **10** | 13 | 11 | — |
| `RcptTo` traversals | 1 | 3 | 1 | — |
| Client is | **originator** | originator | relay | — |
| Domains align | **yes** | yes | **no** | both cases covered |
| Slices touched | 10 of 12 | 11 of 12 | 11 of 12 | **11 of 12** |
| `Reset` | ✗ | ✗ | ✗ | **still uncovered** |

Three paths, and `Reset` remains the only untouched slice. It needs D.2.

⚠️ This path touches **ten** slices, not eleven — `SessionTranscript` is not exercised, because no
operator reads the transcript in this scenario. That is a reduction in coverage, and it is the
price of being the simplest illustration.

---

## Completeness, instantiated

### The `Received:` header we prepend

```
Received: from bar.com ([203.0.113.20])
          by foo.com with SMTP
          id f2C8D14
          for <Jones@foo.com>;
          Tue, 19 May 1998 09:14:07 -0700
```

| Clause | Value | Source |
|:--|:--|:--|
| `FROM` domain | `bar.com` | `ClientIdentified.claimed_domain` |
| address literal | `203.0.113.20` | `ConnectionAccepted.peer_address` |
| `BY` | `foo.com` | config — **H6** |
| `WITH` | `SMTP` | `ClientIdentified.protocol` |
| `ID` | `f2C8D14` | `MessageAccepted.queue_id` |
| **`FOR`** | **`<Jones@foo.com>`** | **`RecipientAccepted.forward_path` — emitted, because exactly one** |
| timestamp | `1998-05-19T09:14:07-07:00` | `MessageAccepted.received_at` |

Every clause resolves to an event field or to config. No orphans introduced.

---

## What this walk tested

**1. Field independence, in the case where it is least obvious.** `claimed_domain` is `bar.com` and
`reverse_path` is `<Smith@bar.com>`. They match — and the model still does not relate them. They
are independent fields on independent events, exactly as when they differed in the relay path.

This is the more useful half of the pair, and not the obvious one. The relay path proved the model
*tolerates* a mismatch. This path is where a future modeler, seeing two identical domains sitting
next to each other, might be tempted to derive one from the other or to add a check that they
agree. **The alignment here is a coincidence of the scenario, not an invariant of the protocol.** A model
that coupled them would pass this walk and fail the relay one.

The RFC does not state this directly — it is **silent** on any relationship between the `HELO`
domain and the reverse-path, and silence is the whole argument. The nearest text is §4.1.4, which
forbids something adjacent:

> *"An SMTP server MAY verify that the domain name argument in the EHLO command actually
> corresponds to the IP address of the client. However, if the verification fails, the server
> **MUST NOT refuse to accept a message on that basis**."*

That is `HELO`-domain versus **client IP**, not versus reverse-path — a different pair. But it
establishes the specification's posture at this layer: an identity claim in `HELO` is **for logging
and tracing**, not for gatekeeping. Deriving or enforcing anything from it runs against that.

**2. The `FOR` clause fires.** One recipient, so it is emitted. Consistent with the relay path; the
rule is now exercised in both topologies.

**3. Happy for every actor.** No rejection, no partial outcome.

| Actor | Direct, multi | Relay, single | This path |
|:--|:--|:--|:--|
| Server | success | success | success |
| Sender | **partial** — 2 of 3 | success | success |
| Recipient | 2 of 3 | success | success |

This is the strongest candidate yet for the golden designation — simplest topology, fewest steps,
every actor succeeds. **Golden remains unassigned**, deliberately; see *Happy is not golden* in
[`helo-multi-recipient.md`](helo-multi-recipient.md).

---

## What it did not test

**Nothing new was found, and that was expected.** Three paths in, the pattern holds: a clean path
confirms and does not discover. Both payload defects, the orphan field and H3's resolution came
from the messy paths.

Specifically not covered:

- **`SessionTranscript`** — no operator in this scene, so the twelfth slice goes untouched. Both
  other paths reach it.
- **`Reset`** — still uncovered by any path. D.2 is the only thing that will close it.
- **Received-header stacking.** D.1 shows the content as `Blah blah blah...` with no headers, and
  whether a prior `Received:` line exists depends on how the message reached `bar.com` — which is
  outside this model's scope. The relay path is where stacking is actually exercised. **Do not read
  this path as evidence that ours is the first trace line**; it is evidence that the model does not
  need to know.
- **Any error branch.** No 4xx, no 5xx, no sequencing error.

---

## Next paths

| Path | Exercises | Source |
|:--|:--|:--|
| **D.2 — aborted transaction** | `Reset`, the last uncovered slice | RFC, quotable |
| **No `HELO`** | `503` sequencing errors | ours |
