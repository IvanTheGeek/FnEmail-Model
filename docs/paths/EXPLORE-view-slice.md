# Exploring — the View Slice form

**Open. Nothing here is adopted.** The Command Slice form was reworked on 2026-08-07 and the View
Slice was deliberately left behind, because bringing it along would have prejudged a question the
corpus does not settle. This document is the starting point: every View Slice as it stands today,
unchanged, so the rework has something to argue against.

Command Slices now read: wire, command **with its actor-supplied fields**, event, and a `Given`
carrying the minimal dependency. See [`STEP-FORM.md`](STEP-FORM.md).

---

## The three View Slices, exactly as they are now

All three are exercised by [`helo-direct-single-recipient.md`](helo-direct-single-recipient.md).

⚠️ **`SessionTranscript` is deliberately excluded.** It is a fourth View Slice in the model that no
path walks, so it has no instantiated values. Consciously set aside for now rather than overlooked.

### `SessionState` — walked, step 3

| 🟩 V · Step 3 | `SessionState` |
|:--|:--|
| Consumed by | ⬜ `MailFrom` · `RcptTo` · `BeginData` |
| | 🟩 **SessionState**&#10;<br>&nbsp;&nbsp;`identified`: true&#10;<br>&nbsp;&nbsp;`transaction_open`: false |
| Sources | 🟧 **ConnectionAccepted** · **ClientIdentified** |

### `RecipientDirectory` — walked, step 5 · the translation boundary

| 🟩 V · Step 5 | `RecipientDirectory` &nbsp;*(translation boundary — H3)* |
|:--|:--|
| Consumed by | ⬜ `RcptTo` |
| | 🟩 **RecipientDirectory**&#10;<br>&nbsp;&nbsp;`is_local`: true |
| Sources | 🟨 translated from the **Directory context** — external, deferred |

### `TransactionState` — walked, step 7

| 🟩 V · Step 7 | `TransactionState` |
|:--|:--|
| Consumed by | ⬜ `RcptTo` · `BeginData` · `SubmitContent` |
| | 🟩 **TransactionState**&#10;<br>&nbsp;&nbsp;`open`: true&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipient_count`: 1 |
| Sources | 🟧 **MailTransactionStarted** · **RecipientAccepted** |

---

## What is already known to be wrong with this form

Recorded so the rework does not have to rediscover it.

**It has no `Given`.** Command Slices now carry the minimal dependency explicitly. A View Slice's
`Sources` row is *arguably* the same thing under another name — it lists the events the read model
folds — but `Sources` and `Given` are not obviously the same claim, and the difference has never
been stated.

**It reads bottom to top while its neighbors read top to bottom.** The current rule says so
outright. Ten stacked steps that change direction three times is a real cost that was accepted when
View Slices were the only exception; the Command Slice rework has now moved the baseline.

**The rows may already be a Given/When/Then in disguise.** Mapping them onto the corpus vocabulary:

| current row | plausibly is |
|:--|:--|
| `Sources` | **Given** — the events |
| `Consumed by` | **When** — the query |
| the read model | **Then** — the result |

If that holds, the form is GWT written in non-corpus words in non-corpus order — and relabeling
would be a rename rather than a redesign.

---

## ⚠️ The corpus disagrees with itself here — this is the whole reason for a separate pass

Detail and citations in `research/GWT-FINDINGS.md`. In brief:

**Dymitruk specified a View as Given/Then with no `When` for six years**, from 2019 through YOW! 2023
and into a 2025 talk, giving a different reason each time — no command, no action, *"you can't reject
state it is what it is"*.

**Then he began adding a `When`, in three podcast episodes** — 2025-03-28, 2025-11-26, 2026-03-29 —
where the `When` is a **query with a key**, not event filtering. His stated motive is a uniform test
signature. It has **not** propagated to his talks or to eventmodeling.org, and episode 43 puts it as
*"the standard I'd like to push forward"*, which is an aspiration.

**Dilger's own artifacts contradict each other**: his 2026 blog writes a view scenario with a query
in the `When`, while his tooling still emits `"when": []` and generates `.when([])`.

**And his book draws it two ways in one chapter** — Fig. 13.5 omits the `When` row entirely,
Fig. 13.6 shows it present and empty.

So *"does a View Slice have a When"* has no settled answer to copy. **Whatever this project picks is
a choice, and it needs to be labeled as one.**

⚠️ Note also that `../event-model.md` **already ships** `WHEN Query(session)` on a state view — a
form whose only support is those three podcast episodes, and which names no key, which is precisely
the *"implicit requirement"* episode 24 complains about.

---

## Questions the rework has to answer

1. **Does a View Slice get a `When`?** And if so, is it a query with a key — `Query(session_id=…)`
   rather than `Query(session)`?
2. **Is `Sources` just `Given` renamed?** If yes, the fix is a relabel and the row order.
3. **Which direction does it read?** Keeping bottom-to-top preserves the existing documents;
   flipping it makes every step in a path read the same way.
4. **What is a View's dependency on an external context?** `RecipientDirectory` has no event of ours
   at all. On a Command Slice that now shows as a 🟨 `Given`; a View has nowhere to put it except
   `Sources`, where it currently sits.
5. **Does the `Where` row belong?** See the attempt below — it is the one row with no counterpart on
   a Command Slice.

---

# An attempt — the View Slice on the Command Slice's ordering

Asked 2026-08-07. **One step, tried rather than argued about.**

## The realization that makes it work

Take `Helo` as it stands. Its wire row carries **both** halves of the exchange:

| 🟦 C · Step 2 | `Helo` — *as it is today* |
|:--|:--|
| MTA Client | ⬛ `HELO` bar.com&#10;<br>`250` foo.com |
| | 🟦 **Helo**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Event | 🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Given | 🟧 **ConnectionAccepted** |

**Those two lines are not the same kind of thing.** `HELO bar.com` is the client's *input* — it is
what the command carries. `250 foo.com` is the server's *output* — and **a command does not return
data**. Something has to have been read to produce that reply.

So the reply belongs to a **View Slice**, and the wire row of a View is the **rendered output** —
the SMTP equivalent of a screen. The client is still the actor; it is simply receiving rather than
sending.

## The attempt

Rows aligned by **kind** with a Command Slice, so a reader scanning ten stacked steps meets the same
row in the same place:

| | Command Slice | View Slice |
|:--|:--|:--|
| 1 | ⬛ wire **in** — what the actor sent | ⬛ wire **out** — what the actor sees |
| 2 | 🟦 the command *(blank label — the slice itself)* | 🟩 the read model *(blank label — the slice itself)* |
| 3 | 🟧 the event emitted | — *a view emits nothing* |
| 4 | `Given` — the events depended on | `Given` — the events folded |
| 5 | — | `Where` — the query that reaches the row |

**Split into two steps:**

| 🟦 C · Step 2 | `Helo` |
|:--|:--|
| MTA Client | ⬛ `HELO` bar.com |
| | 🟦 **Helo**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Event | 🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Given | 🟧 **ConnectionAccepted** |

| 🟩 V · Step 3 | `SessionState` |
|:--|:--|
| MTA Client | ⬛ `250` foo.com |
| | 🟩 **SessionState**&#10;<br>&nbsp;&nbsp;`identified`: true&#10;<br>&nbsp;&nbsp;`transaction_open`: false |
| Given | 🟧 **ConnectionAccepted** |
| | 🟧 **ClientIdentified** |
| Where | `session_id`: 01J8Z… |

**Read it bottom-up** and it is a sentence: *for this session, fold these two events, producing this
state, which renders as this reply.* Read top-down and it is what a reader encounters in order: the
reply, what produced it, and why.

## What the attempt exposed

**The `Where` row is doing real work, and it reaches for something the paths never show.** The only
thing that scopes `SessionState` to *this* conversation is `session_id`, which the model carries as
**ambient metadata** — `../event-model.md` → *Metadata*, where it is defined as *the connection* and
explicitly *"never repeated in the payload"*.

So the `Where` is the **first place in any path document where ambient metadata becomes visible**.
Every payload field shown so far is business data; a query key is not. Three consequences:

- **It cannot be answered from the rows above it.** `Given` lists events, and no event's payload
  contains `session_id`. The value comes from the envelope.
- **It has no real value in this walk.** Rule 8 wants `01J8Z…`-style concreteness and the paths have
  never instantiated one, because nothing until now needed it. Shown above as a placeholder, which
  **breaks rule 8** and is flagged rather than hidden.
- **It is the first row that reaches outside its own slice for a value.** That may be the strongest
  argument that a `Where` is genuinely a fifth kind of row rather than a variant of `Given`.

⚠️ **`Consumed by` has disappeared and that is a loss.** The old form named the slices that read this
view — `MailFrom`, `RcptTo`, `BeginData`. The new form names only the wire it renders to. Those are
different facts and the second does not imply the first: `SessionState` renders a `250` reply *and*
is read by three later commands. **Unresolved** — a sixth row, or folded into `Where`, or accepted as
lost.

⚠️ **This step numbering assumes a split.** Making the reply its own step turns ten steps into
roughly seventeen, since most commands draw a reply. That is a large change to every path and is
**not** implied by adopting the row ordering — the two decisions are separable, and only the ordering
was asked for here.
