# Exploring — the View Slice form

**Concluded by the v2 rulings, 2026-08-08 — kept as the reasoning record.** The closing questions
were answered on the walk: a view is the dataset provided to the actor, view steps gained `Given`
rows and read top-down like every other step, the required-first preamble carries what no walked
event supplies, and the `Where` row died on paths — the removal was the finding. What stays live
here: the **slice-level** question (does a View Slice's *composed* GWT get a `When`?) and the
corpus contradiction survey below, which is recorded nowhere else. Originally: the Command Slice
form was reworked on 2026-08-07 and the View Slice was deliberately left behind, because bringing
it along would have prejudged a question the corpus does not settle; this document was the
starting point — every View Slice as it stood, unchanged, so the rework had something to argue
against.

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

Detail and citations in `event-modeling-research:research/GWT-FINDINGS.md`. In brief:

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
| 5 | — | `Where` — the query predicate that reaches the row |

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
| Given | 🟧 **ConnectionAccepted**&#10;<br>🟧 **ClientIdentified** |
| Where | `session_id` = 01J8Z… |

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

> ## ⚠️ Corrected 2026-08-07 — two of those three bullets are wrong
>
> **First, the exposure above is the process working, not a defect it uncovered.** The `Where` row
> forced the correlation key to be named, the model turned out to already carry it as ambient
> metadata, and no path had ever instantiated one because nothing had needed to. That is a walk doing
> its job — rule 8 finding the field nobody had instantiated.
>
> **"The rows above it" is the wrong frame.** It borrows Dymitruk's *"the given is the previous
> rows"*, but in his model *rows* means the **progressing timeline** — events that occurred in
> earlier slices, anywhere in the history — not the physical rows of one table. A `Given` is
> satisfied by anything that has already happened, however far back.
>
> **"The first row that reaches outside its own slice" is backwards.** **Reaching outside the slice
> is the default.** A `Given` is *by definition* events from previous slices; that is the entire
> point of it. Every `Given` in every step already does this, so it distinguishes nothing.
>
> **And it *can* be answered from the `Given`.** `session_id` is ambient on **every** event, so the
> events already listed carry it. It is not reaching somewhere unreachable — it reads the envelope
> rather than the payload.
>
> **What a `Where` actually adds is a different *kind* of statement, not a different source.**
> `Given` **enumerates** — these events happened. `Where` **selects** — of all the times they
> happened, this session's. One lists, the other filters. That is a better argument for it being a
> fifth kind of row than anything in the bullets above, and it survives the corrections.

⚠️ **`Consumed by` has disappeared and that is a loss.** The old form named the slices that read this
view — `MailFrom`, `RcptTo`, `BeginData`. The new form names only the wire it renders to. Those are
different facts and the second does not imply the first: `SessionState` renders a `250` reply *and*
is read by three later commands. **Unresolved** — a sixth row, or folded into `Where`, or accepted as
lost.

⚠️ **This step numbering assumes a split.** Making the reply its own step turns ten steps into
roughly seventeen, since most commands draw a reply. That is a large change to every path and is
**not** implied by adopting the row ordering — the two decisions are separable, and only the ordering
was asked for here.

---

# The cadence, and the slices hiding between a command and its reply

Raised 2026-08-07, and it answers the `Consumed by` loss above rather than leaving it open.

## Command and View alternate, and the alternation is load-bearing

**C · V · C · V** is the rhythm of a model, not a coincidence of this example. A command changes
state; a view makes the changed state readable; the next command acts on what was read. A step that
breaks the alternation is usually a step doing two jobs.

⚠️ **Verified in Dymitruk's own file**,
`event-modeling-research:research/archive/open-spaces-comocamp/eventmodel.drawio`:
the actor swimlanes are **Joe the Organizer**, **Adam the Participant**, **Alice the misfit** and
**Eugene the Sysadmin**. The sysadmin lane is what makes the cadence affordable — it gives a View a
place to live when **nothing is shown to the external actor**. The V slot is kept; the audience
changes.

## Which means there are slices between a command and its reply

`250 foo.com` is not automatic. Between receiving `HELO bar.com` and rendering a reply, the server
may consult policy — is `bar.com` a domain we accept mail from? — and answer with an error code
instead. **That check is a read model**, and it belongs in the operator's lane rather than the
client's, because no MTA ever sees it.

| 🟩 V | `DomainPolicy` — *sketch, not in the model* |
|:--|:--|
| Operator | 🟤 *nothing rendered — internal* |
| | 🟩 **DomainPolicy**&#10;<br>&nbsp;&nbsp;`blocked`: false |
| Given | 🟧 **DomainBlocked**&#10;<br>&nbsp;&nbsp;`domain`: spam.example |
| Where | `claimed_domain` = bar.com |

**This project already does exactly this and did not notice.** `RecipientDirectory` is a View
consumed by `RcptTo` and rendered to nobody — it is the same shape as the sketch above, and it is
why `RcptTo` can answer `550` instead of `250`. The pattern is in the model; only the *reason* for it
was missing.

## Which kills `Consumed by` — dependencies only ever point backward

The previous attempt lost `Consumed by` and I called that a loss, then proposed restoring it as a
sixth row. **Both were wrong.** It should not be there at all.

**`MailFrom`, `RcptTo` and `BeginData` already look back through their own `Given` rows.** That is
where a dependency is declared, and it is declared **by the slice that needs it**. A `Consumed by`
row states the same relationship from the other end, which buys nothing and costs three things:

- **It duplicates.** The fact already exists, once per consumer, in each consumer's `Given`.
- **It goes stale silently.** A new path introduces a new consumer, and nothing forces the view's
  list to be updated — the consumer's `Given` is correct while the view's row quietly is not.
- **It is derived data written by hand.** Under *paths are the source, slices are derived*, the set
  of consumers is obtained by **inverting every `Given` across every walk**. A view cannot know its
  own consumers until the last path is walked, so authoring the list is authoring a conclusion.

**Dependencies point backward, never forward.** A slice declares what it needs; nothing declares
what needs it.

### And that is the mechanism, not a tidiness argument

⚠️ **This is the point of the whole apparatus.** If a command needs a value and no prior event
carries it, then **the model is missing a data collection point somewhere in the past** — and the
`Given` is what makes that visible, at the step where the value is wanted rather than in a review
later.

Working backward from the need is how the hole gets located: which earlier slice *should* have
captured it, and did nobody ask for it there, or is the slice missing entirely? A `Consumed by` row
answers none of that, because it looks the wrong way down the timeline.

This is also why the `Given` cannot be dropped as redundant with the walk. Its value is not that it
restates what happened — it is that **an unsatisfiable `Given` is a finding**. That is rule 8 and the
completeness check meeting in one row.

### So a Consulted view has no top row at all

| Kind | Top row | Example |
|:--|:--|:--|
| **Rendered** | ⬛ the wire it draws | `SessionState` → `250` foo.com |
| **Consulted** | **none** — its readers declare it, not it | `RecipientDirectory`, read by `RcptTo` |

| 🟩 V · Step 3 | `SessionState` |
|:--|:--|
| MTA Client | ⬛ `250` foo.com |
| | 🟩 **SessionState**&#10;<br>&nbsp;&nbsp;`identified`: true&#10;<br>&nbsp;&nbsp;`transaction_open`: false |
| Given | 🟧 **ConnectionAccepted**&#10;<br>🟧 **ClientIdentified** |
| Where | `session_id` = 01J8Z… |

| 🟩 V | `RecipientDirectory` — *consulted, renders nothing* |
|:--|:--|
| | 🟩 **RecipientDirectory**&#10;<br>&nbsp;&nbsp;`is_local`: true |
| Given | 🟨 translated from the **Directory context** — external, deferred |
| Where | `forward_path` = \<Jones@foo.com> |

**Three rows when nothing is rendered, four when something is.** No 🟤 in a wire row, which retires
the stretch of that marker flagged earlier — the row is simply absent rather than present and empty.

## Where the error reply goes

If policy rejects `bar.com`, the reply is not `250`. That is a **different rendering of the same
step**, which is where the corpus puts variation: an error is a `Then`, exclusive with the event, and
never both. It does **not** become a branch in the model — it becomes another scenario, or a later
workflow, per *"branching is not shown"*.

⚠️ **This is unbuilt.** `DomainPolicy` is a sketch. FnEmail's charter is `HELO`-only inbound with no
policy layer, and inventing one to make the cadence look tidy would be modeling ahead of need. It is
recorded because it explains a shape already present in `RecipientDirectory`, not because it should
be added.
