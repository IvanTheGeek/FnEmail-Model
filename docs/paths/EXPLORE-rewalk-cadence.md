# Re-walk — the direct path under command/view cadence

**Open. Nothing here is adopted.** A parallel walk of
[`helo-direct-single-recipient.md`](helo-direct-single-recipient.md) with everything settled on
2026-08-07 applied at once, so the two can be read side by side.

What changed, and why a re-walk rather than an edit: **a server reply is a rendered read model, not
part of the command that provoked it.** Splitting each reply out of its command's wire row turns
ten steps into fifteen and puts four replies in front of read models that **do not exist in the
model**. That is not a formatting difference, so it was walked again rather than patched.

| Applied | From |
|:--|:--|
| Commands carry the fields they take from the actor | `STEP-FORM.md` |
| `Given` is minimal, one row, Event-row structure | settled 2026-08-07 |
| 🟤 marks a `Given` with nothing, and only a `Given` | settled 2026-08-07 |
| A rendered view's top row is the wire it draws | this document |
| A consulted view has **no** top row — its readers declare it | `EXPLORE-view-slice.md` |
| `Where` is a query predicate, written with `=` | `EXPLORE-view-slice.md` |
| No `Consumed by` anywhere — dependencies point backward only | `EXPLORE-view-slice.md` |

---

## The dialogue being walked

Unchanged from the original path. RFC 5321 D.1 reduced to one recipient, `HELO` for `EHLO`.

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

**Seven server replies.** The original walk absorbed all seven into command steps. This one gives
each its own view.

---

## The walk

### 🟦 C · Step 1 · `AcceptConnection`

⚠️ **Known wrong, deliberately not fixed here.** Flagged on 2026-08-07 and deferred. Carried
forward unchanged so this document differs from the original in *one* dimension only.

| 🟦 C · Step 1 | `AcceptConnection` |
|:--|:--|
| MTA Client | ⬛ *TCP connection accepted — no SMTP bytes yet* |
| | 🟦 **AcceptConnection**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |
| Event | 🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |
| Given | 🟤 |

### 🟩 V · Step 2 · `ServiceReady` &nbsp;🟥 **not in the model**

| 🟩 V · Step 2 | `ServiceReady` &nbsp;🟥 |
|:--|:--|
| MTA Client | ⬛ `220` foo.com Simple Mail Transfer Service Ready |
| | 🟩 **ServiceReady**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`accepting`: true |
| Given | 🟧 **ConnectionAccepted** |
| Where | `session_id` = 01J8Z… |

⚠️ **No such read model exists.** `server_domain` is configuration, which is **H6**. The greeting was
previously just text on a command's wire row; giving it a view forces the question of what state
produced it.

### 🟦 C · Step 3 · `Helo`

| 🟦 C · Step 3 | `Helo` |
|:--|:--|
| MTA Client | ⬛ `HELO` bar.com |
| | 🟦 **Helo**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Event | 🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Given | 🟧 **ConnectionAccepted** |

### 🟩 V · Step 4 · `SessionState`

| 🟩 V · Step 4 | `SessionState` |
|:--|:--|
| MTA Client | ⬛ `250` foo.com |
| | 🟩 **SessionState**&#10;<br>&nbsp;&nbsp;`identified`: true&#10;<br>&nbsp;&nbsp;`transaction_open`: false |
| Given | 🟧 **ConnectionAccepted**&#10;<br>🟧 **ClientIdentified** |
| Where | `session_id` = 01J8Z… |

⚠️ **The reply renders `foo.com` and this view does not carry it.** Same gap as step 2.

### 🟦 C · Step 5 · `MailFrom`

| 🟦 C · Step 5 | `MailFrom` |
|:--|:--|
| MTA Client | ⬛ `MAIL FROM:`\<Smith@bar.com> |
| | 🟦 **MailFrom**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com> |
| Event | 🟧 **MailTransactionStarted**&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com> |
| Given | 🟧 **ClientIdentified** |

### 🟩 V · Step 6 · `TransactionState`

| 🟩 V · Step 6 | `TransactionState` |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **TransactionState**&#10;<br>&nbsp;&nbsp;`open`: true&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipient_count`: 0 |
| Given | 🟧 **MailTransactionStarted** |
| Where | `session_id` = 01J8Z… |

### 🟩 V · Step 7 · `RecipientDirectory` &nbsp;*(consulted — renders nothing)*

**No top row.** Nothing is drawn to any actor; `RcptTo` declares this dependency in its own `Given`.

| 🟩 V · Step 7 | `RecipientDirectory` &nbsp;*(translation boundary — H3)* |
|:--|:--|
| | 🟩 **RecipientDirectory**&#10;<br>&nbsp;&nbsp;`is_local`: true |
| Given | 🟨 translated from the **Directory context** — external, deferred |
| Where | `forward_path` = \<Jones@foo.com> |

### 🟦 C · Step 8 · `RcptTo`

| 🟦 C · Step 8 | `RcptTo` |
|:--|:--|
| MTA Client | ⬛ `RCPT TO:`\<Jones@foo.com> |
| | 🟦 **RcptTo**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Event | 🟧 **RecipientAccepted**&#10;<br>&nbsp;&nbsp;`forward_path`: \<Jones@foo.com> |
| Given | 🟧 **MailTransactionStarted**&#10;<br>🟨 **Directory translation** — *no event of ours* |

### 🟩 V · Step 9 · `TransactionState` &nbsp;*(second traversal)*

| 🟩 V · Step 9 | `TransactionState` |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **TransactionState**&#10;<br>&nbsp;&nbsp;`open`: true&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipient_count`: 1 |
| Given | 🟧 **MailTransactionStarted**&#10;🟧 **RecipientAccepted** |
| Where | `session_id` = 01J8Z… |

### 🟦 C · Step 10 · `BeginData` &nbsp;🟥 **H1**

| 🟦 C · Step 10 | `BeginData` &nbsp;🟥 **H1** |
|:--|:--|
| MTA Client | ⬛ `DATA` |
| | 🟦 **BeginData** |
| Event | 🟧 **DataPhaseEntered**&#10;<br>&nbsp;&nbsp;*no payload* |
| Given | 🟧 **MailTransactionStarted**&#10;<br>🟧 **RecipientAccepted** |

### 🟩 V · Step 11 · `DataPrompt` &nbsp;🟥 **not in the model — but see H1 below**

| 🟩 V · Step 11 | `DataPrompt` &nbsp;🟥 |
|:--|:--|
| MTA Client | ⬛ `354` Start mail input; end with `<CRLF>.<CRLF>` |
| | 🟩 **DataPrompt**&#10;<br>&nbsp;&nbsp;`awaiting_content`: true |
| Given | 🟧 **DataPhaseEntered** |
| Where | `session_id` = 01J8Z… |

### 🟦 C · Step 12 · `SubmitContent`

| 🟦 C · Step 12 | `SubmitContent` |
|:--|:--|
| MTA Client | ⬛ `Date:` Tue, 19 May 1998 09:14:02 -0700&#10;<br>`From:` Smith \<Smith@bar.com>&#10;<br>`To:` Jones@foo.com&#10;<br>`Subject:` Tuesday&#10;<br>(blank)&#10;<br>Blah blah blah...&#10;<br>`.` |
| | 🟦 **SubmitContent**&#10;<br>&nbsp;&nbsp;`content`: 194 octets, dot-unstuffed |
| Event | 🟧 **MessageAccepted**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipients`: [\<Jones@foo.com>]&#10;<br>&nbsp;&nbsp;`content_ref`: blob:sha256:9c1e…&#10;<br>&nbsp;&nbsp;`actual_octets`: 194&#10;<br>&nbsp;&nbsp;`received_at`: 1998-05-19T09:14:07-07:00 |
| Given | 🟧 **DataPhaseEntered** |

### 🟩 V · Step 13 · `MessageQueued` &nbsp;🟥 **not in the model**

| 🟩 V · Step 13 | `MessageQueued` &nbsp;🟥 |
|:--|:--|
| MTA Client | ⬛ `250` OK |
| | 🟩 **MessageQueued**&#10;<br>&nbsp;&nbsp;`queue_id`: f2C8D14&#10;<br>&nbsp;&nbsp;`accepted`: true |
| Given | 🟧 **MessageAccepted** |
| Where | `session_id` = 01J8Z… |

**The responsibility boundary is here.** Left of it, abandoning costs nothing; at `MessageAccepted`
we have accepted responsibility for delivering or reporting failure — RFC 5321 §2.1. **The `250` is
the moment the client learns that**, which the original walk did not make a step of its own.

### 🟦 C · Step 14 · `Quit`

| 🟦 C · Step 14 | `Quit` |
|:--|:--|
| MTA Client | ⬛ `QUIT` |
| | 🟦 **Quit** |
| Event | 🟧 **SessionClosed**&#10;<br>&nbsp;&nbsp;`cause`: quit |
| Given | 🟧 **ConnectionAccepted** |

### 🟩 V · Step 15 · `SessionClosing` &nbsp;🟥 **not in the model**

| 🟩 V · Step 15 | `SessionClosing` &nbsp;🟥 |
|:--|:--|
| MTA Client | ⬛ `221` foo.com Service closing transmission channel |
| | 🟩 **SessionClosing**&#10;<br>&nbsp;&nbsp;`server_domain`: foo.com&#10;<br>&nbsp;&nbsp;`cause`: quit |
| Given | 🟧 **SessionClosed** |
| Where | `session_id` = 01J8Z… |

---

## What the re-walk exposed

### 1. The model has four read models and the protocol renders seven replies

| Reply | Rendered by | In the model? |
|:--|:--|:--|
| `220` greeting | `ServiceReady` | ❌ |
| `250` after `HELO` | `SessionState` | ✅ |
| `250` after `MAIL FROM` | `TransactionState` | ✅ |
| `250` after `RCPT TO` | `TransactionState` | ✅ |
| `354` after `DATA` | `DataPrompt` | ❌ |
| `250` after content | `MessageQueued` | ❌ |
| `221` after `QUIT` | `SessionClosing` | ❌ |

**Four of seven replies have no read model behind them.** They were invisible while each reply sat
as text on a command's wire row. This is the completeness check running in the direction it was
always meant to run: *what state produced this output?*

⚠️ **Naming these four is not proposing them.** Some may collapse — `ServiceReady` and
`SessionClosing` both render `server_domain` and may be one lifecycle view; `DataPrompt` may be
`TransactionState` in another state. **Do not add four slices on the strength of this table.**

### 2. H1 may be resolved by the cadence

`DataPhaseEntered` carries no payload and the open question is whether it earns its place. Its only
candidate consumer was *"the transcript rendering `354`"*.

**Under the cadence it has a direct consumer**: step 11's view, whose `Given` is exactly
`DataPhaseEntered` and whose sole job is to render the `354`. An event that a view folds is an event
that is used.

⚠️ **Candidate, not a resolution.** It depends on accepting that a `354` prompt is a rendered read
model rather than an artifact of the command. If that is accepted, H1 dissolves; if it is not, H1
stands untouched. Rule 9 — recorded here, not moved in `event-model.md`.

### 3. The server's own domain has no home

`foo.com` renders in three replies — `220`, `250` after `HELO`, and `221`. **No read model in the
model carries it.** It is configuration, which is **H6**, and the re-walk shows H6 is not a detail
of the `Received:` header but a value the protocol emits at the very first and very last thing the
client ever sees.

### 4. ⚠️ The cadence is not doctrine — the corpus prescribes *dependency*, not *sequence*

Asked for verification 2026-08-07 rather than taken on my assertion. **Nothing in the corpus requires
command and view to alternate.** What it prescribes is narrower and different.

**Slices split by type, and that rule is explicit.** The slicing kit, line 33:

> *"A slice never mixes a COMMAND and a READMODEL — the platform models these as two distinct slice
> types (`state-change` and `state-view`). If a 'feature' needs both a command and a read model …
> that's **two slices**, not one."*

**Two of the four patterns are *already* a view followed by a command.** Automation and Translation
are both composed **read + write**, and Dymitruk specifies an automation as **two** specifications:

> *"The specification for this is always done in 2 parts: The Given-When-Then for creating the
> to-do list and the Given-When-Then for executing the command."*

So **V → C is not merely allowed, it is a named composition.** Steps 7 and 8 of this walk —
`RecipientDirectory` then `RcptTo` — are **one Translation**, which is *"an automation whose input
events belong to someone else's system"*. Step 7 is not a stray view; it is the read half of a
pattern that has a name.

**And slices relate by dependency, not by position.** The kit's step 4 is *"Note Dependencies Between
Slices"*, illustrated as *"`OrderDetailView` depends on `PlaceOrder`'s `OrderPlaced` event"*. That is
a **graph**, not a sequence — and it is exactly what a `Given` row expresses.

**So the V · V at steps 6 and 7 violates nothing.** It is the end of a State View meeting the start
of a Translation: two patterns adjacent, which the corpus neither forbids nor discusses.

⚠️ **Scope of this check.** Searched the method reference and the slicing kit — the two places an
ordering rule would live. **Not an exhaustive sweep**, so this is *no rule found where a rule would
be*, not *proven absent*.

⚠️ **And the alternation is an observation of this protocol, not of the method.** RFC 5321 §4.3.1
calls SMTP an *"alternating dialogue"*. That is why this walk alternates. **The regularity comes from
SMTP, not from Event Modeling** — which is the opposite of what I implied when I first wrote the
cadence up as though the method demanded it.

### 4b. Cadence is per lane, not global

Steps 6, 7 and 8 run **V · V · C**, which breaks a strict alternation. They do not break the rhythm,
because the two views are in **different lanes**: step 6 renders to the MTA Client, step 7 is
consulted internally and renders to nobody.

That is what Dymitruk's sysadmin swimlane is for — verified in his own file, whose actor lanes are
Joe the Organizer, Adam the Participant, Alice the misfit and Eugene the Sysadmin. **Read per lane,
the alternation holds.** A flat step list cannot show that, and this document does not solve it.

### 5. Fifteen steps, and the accounting changes

| | Original | Re-walk |
|:--|:--|:--|
| Steps | 10 | **15** |
| Command steps | 7 | 7 |
| View steps | 3 | **8** |
| Distinct slices touched | 10 | **11**, four of which do not exist |
| Replies shown as wire text inside a command | 7 | **0** |

The command steps are unchanged in number and nearly unchanged in content. **The entire growth is
views** — which is the point: the original walk had been hiding read models inside wire rows.

---

## Against adopting this

Written before any decision, so the case is on record rather than assembled afterward.

**It invents four slices to make a rhythm work.** The strongest objection. `ServiceReady`,
`DataPrompt`, `MessageQueued` and `SessionClosing` exist here because the form demanded something to
render each reply, not because anything in the charter needs them. That is modeling ahead of need,
and rule 10's whole point is that orthodoxy stays measurable only if extensions are not folded in
quietly.

> ### ⚠️ Withdrawn 2026-08-07 — the RFC says these are required
>
> **That objection is wrong, and the RFC overturns it in the strongest available terms.** Raised by
> Ivan with a role-reversal argument: flip the roles, and as an SMTP *client* we **cannot issue the
> next command until we have received and read the reply**. The reply is not decoration around a
> command; it is a value the next step depends on.
>
> **RFC 5321 §4.3.1, *Sequencing Overview*** — read rather than recalled:
>
> > *"The communication between the sender and receiver is an **alternating dialogue**, controlled by
> > the sender. As such, the sender issues a command and the receiver responds with a reply. Unless
> > other arrangements are negotiated through service extensions, the sender **MUST wait for this
> > response before sending further commands**."*
>
> Three things fall out, and the first is the one that matters most.
>
> **The specification calls SMTP an *alternating dialogue*.** The command/view cadence was not
> imported from Event Modeling and imposed on this protocol — **it is the protocol's own
> description of itself.** The section is titled *Sequencing of Commands and Replies*. A form that
> alternates command and view is not a rhythm being imposed; it is the shape of the thing.
>
> **The reply is a MUST, not a courtesy.** A client that does not wait is non-conforming. So a
> read model behind each reply is not a slice invented to fill a slot — it is the state that a
> **mandatory** synchronization point publishes. Under the completeness check, an output the
> protocol requires must have an origin, and four of ours had none.
>
> **And the escape clause does not apply to us.** *"Unless other arrangements are negotiated through
> service extensions"* is `PIPELINING`, RFC 2920, referenced at line 443. FnEmail's charter is
> `HELO`-only with **no ESMTP**, so nothing can ever be negotiated and **the MUST holds
> unconditionally within this scope**.
>
> ⚠️ **One genuine asymmetry survives, and it weakens exactly one of the four.** The greeting is
> a **SHOULD**: *"The sender SHOULD wait for this greeting message before sending any commands."*
> Every other reply is a MUST. So `ServiceReady` rests on weaker ground than `DataPrompt`,
> `MessageQueued` and `SessionClosing` — and this project has been caught once already treating a
> SHOULD as a MUST, which reclassified a whole event when corrected.
>
> **What remains of the objection:** not that the four are invented, but that naming them here is
> still premature. *That* they must exist is now established. *What they are called, and whether
> some collapse into one lifecycle view, is not.*

**Fifteen steps is a worse read than ten.** A walk is for showing someone the model for the first
time. Half the steps now say *and then the server replied*, which is the least surprising thing in
SMTP.

**The gain may be one-off.** Points 1 and 3 above are findings, and findings can be recorded in
prose without restructuring every path. Once written down, the re-walk has delivered most of its
value and the fifteen-step form has to justify itself on readability alone.

**Against all that:** the four missing read models were **invisible** for the entire life of the
original document, and no amount of re-reading it surfaced them. It took changing the form to see
them. That is worth something even if the form is then discarded.
