# Explore — declaration events versus status-log events

**Open, with the step 5 package adopted.** On 2026-08-08 Ivan adopted the rename and the
steps 6/9 collapse; the walk carries both. The adopting commits are titled
*ReversePathDeclared names the step 5 event* and
*Position replaces state at steps 6 and 9*. On 2026-08-09 the step 6 view was named
`ReversePathAllowed`, splitting the slice, and the consulted box at `RecipientDirectory`
dissolved, renumbering the walk (see *Second pass* and *The consulted box dissolves*). Still
open: the `RcptTo` symmetry, the refusal-shape fork, the third-category question, the
command-naming rule. (Closed later on 2026-08-09: the `RecipientAccepted`-occasioned view became
`RecipientConfirmed`, and H1 closed entirely with `DataRequested` — see *The lens applied to
H1*.) The dataset cascade the collapse spawned closed 2026-08-09 — ruled: "Don't invent a new
test — the view's own definition already decides it"; the walk's steps 10, 13 and 15 notes
carry the grounds. Started 2026-08-08 on branch
`explore/declaration-vs-status`, during the step-by-step walk-through of
[`WORKING-helo-direct-single-recipient-v2.md`](WORKING-helo-direct-single-recipient-v2.md). Under
rule 13 the walk lags this file for anything this exploration governs, and nothing else is
reconciled to either until both settle.

This is the working log. It records the options, the evidence for and against each, what adoption
would cost, and what has already failed — including the arguments this file's own author tried and
had to withdraw. It is read for its trail (rule 14 classes it as exploration material), so
superseded reasoning stays in place with a ⚠️ block rather than being cleaned away.

---

## What triggered it

The walk-through reached step 6. The question queued there by commit `cf3227d` was narrow: does
`TransactionState`'s dataset survive the test that killed `accepting` and `protocol`? Working that
question surfaced a larger one that the deferral had not anticipated.

Ivan's observation, 2026-08-08:

> the event "MailTransactionStarted" is actually wrong now that I look at it, that might be a side
> effect that modeling the infrastructure processing of receiving mail might need or something, but
> for this path and applying event modeling naming conventions, ReversePathDeclared or something of
> the like. the step 6 is responding to that exact event — for step 9 while the rendered output IS
> the same, it is in response to a different event (where currently it is responding to the field
> values of a same event that is not really an event — it is a status log)

Two claims, and they are separable. **First**, that `MailTransactionStarted` names a receiver state
transition rather than a thing that happened, which makes it a status log entry wearing an event's
name. **Second**, that steps 6 and 9 are distinguished by *which event occasioned them*, not by the
field values they currently carry — so the field values were doing work that structure already does.

---

## The tell: name and payload disagree

The discriminator that does not depend on anyone's taste. Every event in the walk, with the payload
it actually carries:

| Event | Payload | Name describes | Agree? |
|:--|:--|:--|:--|
| **ConnectionAccepted** | `peer_address`, `local_address` | an act of ours, with the facts it established | yes |
| **ClientIdentified** | `claimed_domain` | the client identifying itself; the payload is the identity | yes |
| **MailTransactionStarted** | `reverse_path` | a transaction beginning; the payload is a declared address | **no** |
| **RecipientAccepted** | `forward_path` | our acceptance; the payload is what we accepted | yes |
| **DataPhaseEntered** | *no payload* | a phase transition; there is no payload at all | **no** |
| **MessageAccepted** | `queue_id`, `reverse_path`, `recipients`, `content_ref`, `actual_octets`, `received_at` | our acceptance, with everything it fixed | yes |
| **SessionClosed** | `cause` | closure, with why | yes |

Two events fail, and they fail in opposite directions: one carries a declaration's argument under a
state-transition name, the other carries nothing at all. **Those two are exactly the ones the
project has independently found problems with.** `DataPhaseEntered` is H1 — *does it earn its
place?* — open since before the v2 walk existed, and named in the method repo twice
(`EventModeling/docs/altitude.md:70`, `docs/extensions.md:22` and `:245`) as the question the SMTP
work keeps landing on.

That convergence is the strongest evidence here: an independent test, applied mechanically,
picks out the two elements that were already sore.

---

## What the RFC says, checked rather than recalled

⚠️ **An earlier draft of this section claimed a discriminator that does not exist, and the claim is
recorded rather than deleted.** The proposed test was: the RFC describes the two suspect commands in
terms of the receiver's buffers, and the sound one in terms of intent. It is false. **All three
commands have buffer sentences** — §4.1.1.3 says RCPT *"appends its forward-path argument to the
forward-path buffer"* (rfc5321.txt:1936-1937), which is the same register as MAIL's and DATA's. The
test was built after reading two of the three sections and checking the third is what killed it.
Rule 3, on our own reasoning rather than on a remembered citation.

⚠️ **A second error, caught before it reached this file.** The sentence *"tells the SMTP-receiver
that a new mail transaction is starting"* was about to be attributed to §4.1.1.2. It is **§3.3**,
rfc5321.txt:1029-1031. §4.1.1.2 does not begin until line 1879. Both sections describe MAIL, which
is exactly why the misattribution was plausible.

What survives the checking is more useful than the discriminator would have been. **The RFC
describes every command in both registers, so it does not choose for us.**

| Command | Intent register | Buffer / state register |
|:--|:--|:--|
| `MAIL` | §4.1.1.2:1881 — *"used to initiate a mail transaction"* | §3.3:1029 — *"a new mail transaction is starting and to reset all its state tables and buffers"*&#10;<br>§4.1.1.2:1895 — *"clears the reverse-path buffer, the forward-path buffer, and the mail data buffer"* |
| `RCPT` | §4.1.1.3:1918 — *"used to identify an individual recipient of the mail data"* | §4.1.1.3:1936 — *"appends its forward-path argument to the forward-path buffer"* |
| `DATA` | §4.1.1.4 — the 354, then the lines that follow treated as mail data | §4.1.1.4:1997 — *"causes the mail data to be appended to the mail data buffer"* |

The RFC even has a complete alternative model at the buffer altitude: §4.1.1 at :1773 states the
machine as *"a reverse-path buffer, a forward-path buffer, and a mail data buffer"*. **That is a
coherent model of SMTP and it is not this one.** Ivan's framing — that `MailTransactionStarted` may
be what an infrastructure-processing model needs — is precisely this register, and the RFC supports
building at that altitude. Choosing between registers is a `model-altitude.md` question, not a
citation question, which means rule 3 cannot settle it and the altitude charter has to.

**One further fact, and it bears on the step 6 / step 9 question.** §3.3 permits the `250` after
`MAIL FROM` to be provisional: *"the server MAY reasonably accept the reverse-path (with a 250
reply) and then report problems after the forward-paths are received and examined"* (:1043-1045),
and *"Normally, failures produce 550 or 553 replies"* (:1045). So both commands can be refused, and
the two `250`s do not mean the same thing — step 6's is *recorded, provisionally*, step 9's is
*this recipient is good*.

---

## The candidate rule, stated so it can be argued with

> **An event names what happened. If its name describes the receiver's state after the fact rather
> than the fact, it is a status log entry and belongs to whatever model owns receiver state — not
> to a protocol path.**

And its corollary, which is what makes it testable rather than a matter of taste:

> **The payload is the check. An event whose payload is some other kind of thing's argument is
> misnamed.**

⚠️ **The rule is not yet tested against the events it passes.** Four events pass above, but passing
was not adversarially attacked — `ConnectionAccepted` and `MessageAccepted` both name *our*
decisions rather than the peer's action, and no one has asked whether that is a third category. Do
that before adopting.

---

## Options

### Option 1 — rename both status events

`MailTransactionStarted` becomes `ReversePathDeclared`; `DataPhaseEntered` becomes something naming
what the client asked for.

**Buys.** Name and payload agree everywhere. The two known sore spots close together, and H1 stops
being *does this event earn its place* — an unanswerable question about a status entry — and becomes
*is this the right name*, which the rule answers.

**Costs.** `DataPhaseEntered` has no declared argument to be named from: `DATA` carries none. So
whatever replaces it names a *request* rather than a *declaration*, which is a third shape the rule
has not yet been stated to cover. Candidate names have not been generated; that is the next piece of
work and it should not be rushed, because H1 has been open a long time and a hasty name would close
it for the wrong reason.

### Option 2 — rename `MailTransactionStarted` only

**Buys.** The clean case lands without waiting on the hard one. Steps 6 and 9 unblock, and the
walk-through resumes.

**Costs.** Leaves the rule half-applied, with `DataPhaseEntered` standing as a known counterexample
to a rule the project has just adopted. That is the exact shape rule 4 exists to prevent going
unrecorded — acceptable only with a ⚠️ block saying the second case is deferred and why.

### Option 3 — reject the rule; keep status events

**Buys.** No blast radius. The RFC's buffer register is real, and a protocol server genuinely does
have receiver state; a model that refuses to name it may be pushing a real thing off the page.

**Costs.** `TransactionState`'s dataset stays incoherent — an event named for a state, folded into a
view named for the same state, rendering a reply that shows neither. And it leaves unanswered why
the two failing events are the two the project already had trouble with.

⚠️ **Not yet argued properly.** Option 3 is stated here so the case against the change is on record
before the change is made, per the practice `EXPLORE-rewalk-cadence.md` used under *Against adopting
this*. It has not been given the strongest form someone who believed it would give it.

---

## The step 5 name — candidates, generated and screened

The piece of work the options above left ungenerated, done 2026-08-08 against the RFC text rather
than from taste: every verb below was grepped before it was argued (rule 3). The corollary makes
the first cut mechanical — the payload is `reverse_path`, so the name is about the reverse-path,
or the replacement fails the same test that condemned the incumbent.

### Killed, and by what

| Candidate | Killed by |
|:--|:--|
| `SenderIdentified` · `SenderDeclared` · `OriginatorIdentified` | **The null path.** `Reverse-path = Path / "<>"` (§4.1.2), and `MAIL FROM:<>` is legal and load-bearing — §4.1.1.2:1893 sends the null case to §3.6, worked at §3.6.3:1522 — so an event with *sender* in its name cannot fire on the bounce-suppression walk; [`../smtp-path-vs-mailbox.md`](../smtp-path-vs-mailbox.md) stands on exactly this fact. *Identify* is also spent: it is RCPT's intent verb (§4.1.1.3:1918), and `ClientIdentified` already holds it |
| `TransactionOpened` and kin | The status register under challenge — §3.3:1029 is where the incumbent's own name came from |
| `EnvelopeOpened` | Real RFC vocabulary — §2.3.1:614 has the envelope consist of an originator address, recipient addresses, and extension material — but it is the same status register one altitude up, and the payload is one path, not an envelope |
| `ReversePathReceived` | Restates the wire; the command row directly above it already records what crossed |
| `ReversePathBuffered` · `ReversePathRecorded` | §4.1.1.2:1895's buffer register — the RFC's other model, already placed at the other altitude above |
| `ReversePathClaimed` | Set aside on a distinction worth keeping: a *claim* is checkable against the world — `claimed_domain`'s register, and the reason the alignment question exists at all. A path is not that kind of thing: `<>` cannot be true or false. Blurring the two registers costs more than the echo buys |

### The three that survive

| Name | Provenance | Against |
|:--|:--|:--|
| `ReversePathDeclared` | Not core RFC 5321 — the sole *declar* hit in the spec is a reference title (:4533). But that title is RFC 1870, *Message Size Declaration*: the ecosystem's own word for a client-supplied, unverified value. And the register has precedent on this exact event — `declared_size` sat on `MailTransactionStarted` from v0.1 (a927002) until the HELO scoping dropped it (4f6379a) | The verb is extension vocabulary, not the base spec's |
| `ReversePathSpecified` | The strongest core grounding: §2.3.1:625 calls MAIL the *"reverse-path (originator) address specification"* command, and §3.3:1035 refuses *"the mailbox specification"* | Registerless — *specify* says nothing about verification status, and this walk has been using register to carry epistemics |
| `ReversePathAccepted` | Direct: *"accept the reverse-path (with a 250 reply)"* (§3.3:1043); *"If accepted, the SMTP server returns a '250 OK' reply"* (:1034) | Three arguments, below |

### What selects among them

**The two `250`s, already found above — and only one option keeps them.** *What the RFC says*
establishes that step 6's reply is *recorded, provisionally* and step 9's is *this recipient is
good*. The page shows the same thing structurally: step 8's acceptance has an adjudication behind
it — step 7 consults `RecipientDirectory` — and step 5 consults nothing. `ReversePathDeclared`
against `RecipientAccepted` carries that difference in the names themselves.
`ReversePathAccepted` erases it, naming the unadjudicated reply identically to the adjudicated
one.

**`Accepted` swells the unexamined category.** The rule's own ⚠️ flags `ConnectionAccepted` and
`MessageAccepted` — events naming our decision rather than the peer's action — as a possible third
shape nobody has attacked. Adopting `ReversePathAccepted` makes it four of seven before the
question is asked.

**And an acceptance cannot fire on a refusal.** A declaration happened whether or not we honor
it, so the same event serves the 550/553 branch and keeps what the client supplied on the page.
Under `Accepted`, the rejection walk records the offered path nowhere — or mints a second event to
hold it.

**The step 8 coupling, argued here, not ruled.** The open item below says consistency pushes
`RecipientAccepted` toward `ForwardPathDeclared`. The two-`250`s fact says that push is
resistible: the asymmetry is not an inconsistency but the finding itself — MAIL's answer is
provisional by the RFC's own words, RCPT's is adjudicated on this very page, and the tell wants
differently shaped names for differently shaped facts. This file's argument, not a ruling; the
bullet stands.

---

## What steps 6 and 9 become

Independent of which option lands, Ivan's second claim stands on its own and is worth recording
separately, because it is about structure rather than naming.

Both `Given` blocks collapse to a single event, existence only:

| Step | `Given` today | Under the claim |
|:--|:--|:--|
| 6 | `MailTransactionStarted` with `reverse_path`: \<Smith@bar.com> | the declaration event alone, existence |
| 9 | `MailTransactionStarted` with `reverse_path`: \<Smith@bar.com>&#10;<br>`RecipientAccepted` | `RecipientAccepted` alone, existence |

Step 9 currently reads a **field value** off the event Ivan calls a status log, and that citation
exists only to feed `TransactionState.reverse_path`. When the field goes, the citation goes with it
— which is the third instance of the move commit `cf3227d` made at step 4, where `ConnectionAccepted`
left the `Given` *"having only ever been there for their fold"*.

**And `recipient_count` was encoding position as data.** It distinguished step 6 from step 9 by
value, 0 against 1, when the occasion distinguishes them structurally. That is the `session_id`
finding again — a value doing work that position already does — and it is the second time this walk
has found one.

So: **one slice, two scenarios, distinguished by `Given`.** Same rendered output, empty dataset both
times. What the current form got wrong was not that the two steps are the same, but that it made
differing field values carry a distinction the `Given` already carried.

**The empty dataset is a fact about this path, not about the slice.** §3.3 gives both occasions a
failure branch (550, 553), so once the rejection walk runs, whatever selects the reply code seats a
field at the slice layer. Empty here; not empty forever.

### The view's name — screened 2026-08-09, left open

`TransactionState` describes state the view no longer carries. The candidates were screened by
the same discipline as the step 5 event's, with one new device the pass added: **the
discrimination test** — three views in this walk render `250`s (`SessionReady`, this one,
`MessageQueued`), so a name that just describes "the 250" names none of them.

| Candidate | Killed by |
|:--|:--|
| `TransactionStatus` · `TransactionProgress` · `TransactionOpen` | the incumbent's own defect — a status register describing a dataset that is gone |
| `ActionCompleted` · `ActionOkay` | §4.2.2's own words, the strongest provenance — and the discrimination test: every `250` is "action okay, completed" |
| `CommandAccepted` · `CommandAcknowledged` | discrimination again (every reply acknowledges a command), and *accepted* overstates step 6's provisional `250` |
| `EnvelopeAccepted` | overstates twice — step 6 is provisional, and the envelope is incomplete until the last `RCPT` |

Two survive. **`TransactionReady`** completes the ruled sibling progression — *service ready,
then session ready* (step 4's note), then transaction ready — discriminates correctly (the only
`250`s that leave the transaction open for more envelope-building), reads naturally on the second
traversal, and is coherent with the empty dataset. Two honest costs: *Ready* is state vocabulary
— but the status-log rule above is about **events**, and `ServiceReady`/`SessionReady` already
settled *Ready* as the view register for what the reply tells the actor; and the rejection branch
would have `TransactionReady` render a `550` — a property `ServiceReady` already has on the `554`
branch, so precedented. **`EnvelopeAcknowledged`** discriminates by what is acknowledged — the
two occasions are exactly the two envelope-building commands, §2.3.1's own vocabulary — at the
cost of breaking the Ready progression and minting a register no other element uses.

Ivan deferred the choice 2026-08-09: candidates recorded, none adopted, the incumbent stands —
until the second pass, below, the same day.

#### Second pass — the view responds to its occasion, and the pick

Ivan's argument, quoted from the working conversation:

> i would argue that THIS View is a required response to the "ReversePathDeclared" so while the
> rendered wireline reply is fixed, the View name should be something that is in response to the
> event trigger of "ReversePathDeclared"

and, on the expanded model:

> later when building out our server there could be any number of processing slices that takes
> ReversePathDeclared and the value of reverse_path: \<Smith@bar.com> could be processed and
> decided that this server is NOT going to accept the mail from that path (address) and hence
> might NOT be based on ReversePathDeclared event but rather MailFromReversePathRefused

Checked against the walk, the principle turned out to be the unwritten rule six of the seven
rendered views already follow — each names the server's response to its occasioning event:
`ServiceReady` ← `ConnectionAccepted`, `SessionReady` ← `ClientIdentified` (step 4's note calls
it "the occasion"), `DataPrompt` ← `DataPhaseEntered`, `MessageQueued` ← `MessageAccepted`,
`SessionClosing` ← `SessionClosed`. `TransactionState` was the odd one out, named for state
instead of response.

The name itself was trimmed from `MailFromReversePathAllowed`: the command prefix is redundant —
the reverse path arrives by exactly one command, and verbs name commands. *Allowed* beats the
earlier screen's *Accepted* on the wrinkle that killed *Accepted* twice: §3.3's `250` is
provisional, and *allowed to proceed* claims permission, not final judgment — the
report-problems-later door stays open in the name. **Adopted for step 6: `ReversePathAllowed`.**

The consequence, accepted with it: **the slice splits.** Step 9's view responds to
`RecipientAccepted` and needs its own name — open, riding the step 8 symmetry
(`ForwardPathAllowed`, if the mirror lands). The walk's accounting becomes 16 steps over 16;
"traversed twice" dissolved with the split.

Recorded open, not settled, from the same pass:

- **The refusal-shape fork.** The model is mixed on how refusals live: `ServiceReady` handles
  its `554` as scenario selection — one view, two codes, the step 2 ruling — while `RcptTo` has
  an event pair (`RecipientAccepted` / `RecipientRejected{forward_path, reply_code, reason}`).
  Ivan's expanded model puts MAIL on the RCPT side: `ReversePathRefused` as a decision event
  with its own responding view. The consumer test decides per slice, with real data, on the
  rejection walk. Pressure toward the event side already exists: the `MailFrom` contract's
  refusal arm is reply-without-event — the shape the `AcceptConnection` correction removed.
- **Two kinds of view, and SMTP's fusion.** Ivan's second observation: one kind feeds the actor
  in the same slice (the rendered replies), one feeds the actor in the *next* command slice.
  Kind two is the method's canonical read-model position — event → read model → next command.
  In SMTP, §4.3.1's lockstep fuses the kinds: the reply *is* the remote actor's screen for the
  next command, which is what the model's old command-screen drawings drew. The unfused case —
  kind two with an internal actor — is `RecipientDirectory`, the open consulted-view question
  at step 7, arrived at from a new direction.

#### The consulted box dissolves — ruled 2026-08-09

The two-kinds fusion answered the step 7 question a day after posing it. `RcptTo`'s actor is
the MTA client, whose kind-two need is already served by the fused reply view; what would read
`RecipientDirectory` is the handler — and the three-fates discipline, checked verbatim, says
*"The handler still never reads a view"*: a command handler's fold gets no box, and when
outside data is genuinely needed *"a processor fetches it and puts it on the command as a
parameter"*. The box performed no fold (`is_local`: true was a verbatim pass-through of the
seeded event), had no reader (no `Given` can cite a view), and was invisible to all three
completeness checks. **Dissolved.** The walk renumbered — steps 8–16 became 7–15 — and the
translation boundary survives in the seeded `RecipientResolved`, exactly where `RcptTo`'s
`Given` already cited it. In the expanded model the consultation returns as a processing slice,
the same shape as the `ReversePathRefused` policy processing above. "Consulted" is now a
category with no walked instance; the method repo's sanction of it is qualified accordingly.

*(Step numbers in this log's earlier sections predate the renumbering and are kept as written —
they were true at the time; rule 14 reads this file for its trail.)*

**And the cadence is descriptive, not doctrine.** Ivan, on where the box came from:

> I think this came from trying to enforce the CVCVCV cadence and while that is the general
> idea it is not doctrine that it must match that. I think Adam's saying that the event model
> is like a sine wave and i think it is really more of a fm frequency wave or a digital block
> wave even and there could be VV+ or CC+ sequences

The record supports him twice over: the cadence exploration's own renderer sweep found that
*"all three agree that long adjacent same-type runs are present in both directions"*, and the
cadence ruling (commit 89ac5fd) is precisely what legitimized `RecipientDirectory` — "slices
hide between a command and its reply" was the argument that put the box on the page. This
refines that ruling's cadence half: alternation is the carrier idea, the wave is FM or square
rather than sine, and VV+ and CC+ runs are legal. Flows to the method repo with the cadence
verdict when that flow-candidate ripens (`FOLLOW-UPS.md`).

#### The two kinds of events — Ivan's observation, 2026-08-09

Working the step 7 evaluation surfaced a classification the walk's own names had been obeying
unnamed: events split into **captures** of the peer's act (`ReversePathDeclared`,
`ClientIdentified` — the peer's vocabulary: *Declared, Identified, Requested*) and **decisions**
of ours (`ConnectionAccepted`, `RecipientAccepted`, `MessageAccepted` — adjudication vocabulary:
*Accepted, Rejected, Resolved*), with the killed status register (`MailTransactionStarted`) as
the third kind that names neither. Ivan, quoted:

> i think it also is the signal that we have missing slices - we gather the data and record that
> gathered it and THEN that data would be processed in some way and produce a decision event.
> Adam's reservation and assignment for a concert seat that needs 2 or more slices and 2 events

and, on what "hidden" means:

> it is hidden as I have never heard Adam nor Martin bring out how an event might be classified
> like this, just that its an event which is a fact or something that happened. so that its a
> capture or a decision has not really been made explicit that I know of. might be interesting
> to see if the status kind event comes up but again, that will not be obvious in the material -
> it would need to come from analysis i am sure

The pipeline reading: every command-shaped interaction latently contains
capture → process → decide → confirm. The walk's envelope steps are that pipeline **losslessly
compressed from opposite ends** — MAIL keeps the capture (the decision, consumed by nothing,
lives in the view), RCPT keeps the decision (the capture rides inside its payload) — each step
keeping exactly the half that has consumers. Slices decompress when a consumer arrives, which
in the expanded model is the processor: policy at MAIL, directory at RCPT, scanning at DATA.
Dymitruk's reservation/assignment is the corpus's own fully decompressed case — capture event,
todo-list processor, decision event, both persisted because both are consumed.

Falsifiable predictions, sent to a corpus review (results below): the corpus defines an event
only as "a fact / something that happened," never classifying; capture and decision kinds appear
implicitly in its examples; the status kind appears in no example as a concept and is findable
only by analysis. Descriptive, not doctrine — a lens the tell and the consumer test already
enforce, same footing as the FM-wave cadence refinement. Method-generic; flows to
`EventModeling` if the review supports it.

#### The corpus review — 2026-08-09, five sweeps, rule 7 citations only

**The hypothesis holds in its precise form: the axis is real, enacted everywhere, named
nowhere.** Per prediction:

- **P1 — never made explicit: supported, with one sharpening.** Both authors define events only
  as fact-that-happened (Dilger, book 0033: *"events are simple facts and not technical"*;
  Dymitruk, machine transcript: *"an event is a really good way of saying fact"*), and the
  absence of taxonomy is by stated design — Dymitruk: *"use regular English and common sense and
  not really add any specific terms"* (machine transcript, OYtEwBunk2A). But the corpus DOES
  classify events explicitly on three *other* axes — internal vs external (ownership), delta vs
  domain events (payload granularity; nebulit iTuw-FXnsyY), and summary events (stream
  lifecycle) — all orthogonal to capture/decision. Ivan's axis is the hidden one, exactly as
  claimed.
- **P2 — both kinds implicit in the examples: strongly supported.** The jewel is Dilger's
  bank-account video (nebulit NcRQdlQMkgE, machine transcript): its whole thesis is that a
  single *money transferred* event is *"too simplistic"* because it merges the request with the
  settlement — *money transfer requested* (capture) → open-transfers todo → processor →
  settlement (decision). The capture/decision split demanded outright, never named. Dymitruk's
  side: *"the only way you mark success is by storing an event"* (RoomBooked — adjudication
  semantics in his own words), and his Open Spaces model runs UniqueIDRequested → todo →
  ConfIDProvided. Sharpest of all: the kit's own naming corrections rewrite the status name
  *PaymentPending* into exactly one capture (*PaymentInitiated*) plus one decision
  (*PaymentAuthorized*) — the taxonomy operating underground in their own tooling, unremarked
  (`event-modeling-research:research/METHOD-REFERENCE-DETAIL.md:780`).
- **P3 — status events absent from models: supported in the strong form, with an altitude
  refinement.** Zero free-floating state-transition events in any worked model (corpus-wide
  grep). But the status shape is deliberately *sanctioned* at two other altitudes: integration
  (Dilger recommends a full-state *updated* event for cross-service publication) and stream
  management (*summary events* like Register Closed, "closing the books" — H4's own territory).
  So the status kind is not a disease but an **altitude marker** — which is precisely what this
  file concluded about `MailTransactionStarted` and §3.3's buffer register before the taxonomy
  existed.
- **Bonus, unpredicted: the taxonomy resolves a recorded corpus contradiction.** The project's
  own research files carry *store-first vs validate-first* as "genuinely unresolved"
  (`event-modeling-research:research/TALKS-FINDINGS-CORPUS.md` §3) — Dymitruk saying both
  *"just store the damn thing as fast as you can"* and validate-before-store. Under the
  taxonomy it is not a contradiction: store-first is capture discipline, validate-first is
  decision discipline. A hidden distinction that dissolves a known contradiction is carving at
  a real joint.
- **A candidate third kind surfaced.** The examples contain events that are neither peer-acts
  nor adjudications: *NotificationSent*, *CartPublished*, the todo-closers — faithful records
  of a side effect the system itself performed. They read as **captures where the actor is the
  system** — the pipeline's execution stage recorded. FnEmail will meet these at delivery
  (right of the responsibility boundary). Noted, not taxonomized.
- Prior art check: the project's earlier corpus analysis reached decision-kind reasoning twice
  (rejection events; one field) and never generalized it. The distinction is new.

Method-generic and now corpus-supported — ripe to flow to `EventModeling` on Ivan's review,
alongside the cadence verdict.

#### The lens applied to H1 — test run, 2026-08-09

`DataPhaseEntered` is the lens's first live test, and the screen closes cleanly. The RFC's own
structure decides the kind question: §4.3.2 lists DATA's reply as `I: 354` — the core protocol's
**only intermediate reply** — so nothing is adjudicated at DATA-time; the decision arrives at
the `250` closing the content (`MessageAccepted`). A decision-named event here would misread the
wire. The peer's act is plain: the client requested to transmit content.

| Candidate | Killed by |
|:--|:--|
| `DataPhaseEntered` (incumbent) | the status register — names the receiver's phase transition; the exploration's own tell table flagged it ("a phase transition; there is no payload at all") |
| `TransferInitiated` · `MailDataInitiated` | §3.3's verb ("initiates transfer") but the machine's process state — the status register wearing an RFC word |
| `DataAccepted` and all decision names | `I:` — nothing is decided; the adjudication is `MessageAccepted`'s |
| `ContentOffered` · `TransmissionRequested` | manufactured verbs with no RFC grounding |

The survivor: **`DataRequested`.** Provenance is the protocol's own reply vocabulary — §4.2.2's
success text reads *"Requested mail action okay, completed"*, framing every command as a
**request** — and the capture register (*Declared, Identified, Requested*) is exactly where a
bare-verb command's event belongs: the request has no argument, the wire row corroborates the
empty payload, and the tell passes where the incumbent failed it. Both consumers survive
untouched (`DataPrompt` folds the request's existence to render the `354`; `SubmitContent`
declares it), and `DataPrompt` becomes a clean response-to-occasion pair: request captured,
prompt issued.

The DATA pipeline is also the lens's best exhibit on the page — the walk's most decompressed
sequence: capture (`BeginData` → the request), prompt (`DataPrompt`), capture (`SubmitContent` →
the content), decide (`MessageAccepted`), confirm (`MessageQueued`). Five slices where the
envelope steps compress to two.

Screened and recommended — then adopted the same day ("land it"): `DataRequested` is the event,
H1 is closed entirely (the earn-its-place half had closed on the walk's consumers), the model's
🔴 marker is cleared, and the method repo's H1 citations updated. The lens's first live test
succeeded on its first application.

#### The second acknowledgment named — 2026-08-09

`RecipientConfirmed`, Ivan's pick from the step 7 evaluation: the reply's job is confirming the
adjudication `RecipientAccepted` already made — a report of a decision, so *confirmed*, not a
second *accepted*, and recipient vocabulary rather than path vocabulary because RCPT's object is
the adjudicated mailbox where MAIL's is the declared route (the path-vs-mailbox distinction
doing naming work). `ForwardPathAllowed` died with the vocabulary argument; whether the *event*
ever moves remains the open `RcptTo` symmetry, and this name rides its occasion either way.
`TransactionState` retires as a page name.

---

## Blast radius

Measured 2026-08-08 with `grep -rc --include='*.md'`. Counts are raw occurrences, not distinct
statements — a step table that names an event three times counts three.

| Name | This repo | Method repo | Research repo (files) |
|:--|:--|:--|:--|
| `MailTransactionStarted` | **63** across 10 files | 0 | 1 |
| `DataPhaseEntered` | **38** across 9 files | 3 (`altitude.md`, `extensions.md` ×2) | 2 |
| `TransactionState` | **38** across 11 files | 3 (`state-view-todo-list-decision-model.md`) | 1 |
| `RecipientAccepted` | 66 across 10 files | 0 | — |
| `MailFrom` | 26 across 11 files | 0 | — |

`MailTransactionStarted`, by file:

| File | Hits | Class under rule 14 |
|:--|:--|:--|
| `paths/EXPLORE-gwt-form.md` | 24 | exploration — keeps the old name as trail |
| `event-model.md` | 9 | normative — reconciles when the prefix drops |
| `paths/WORKING-helo-direct-single-recipient-v2.md` | 8 | the walk — changes now |
| `paths/EXPLORE-rewalk-cadence.md` | 5 | exploration — trail |
| `paths/helo-direct-single-recipient.md` | 4 | walked path — trail |
| `paths/helo-multi-recipient.md` | 3 | walked path — trail |
| `paths/helo-single-recipient.md` | 3 | walked path — trail |
| `model-altitude.md` | 3 | normative — and this is where the altitude ruling belongs |
| `diagrams/README.md` | 2 | normative |
| `paths/EXPLORE-view-slice.md` | 1 | exploration — trail |

**The real number is 8, not 63.** Rule 13 holds the walk apart until its prefix drops; rule 14 keeps
exploration records and superseded walks as trail rather than sweeping them. So adoption changes the
working walk only, and the rest becomes a reconciliation pass scheduled by the prefix drop — the
same pass `event-model.md`'s banner already lists seven other items for.

⚠️ **The research repo hits are not ours to change** and are counted only so the number is not a
surprise later (rule 7 — cite, never reproduce; nothing was read out of that repo for this count
beyond file counts).

---

## If adopted — what needs updating

Ordered by when, not by size.

**Now, on this branch:**

1. The walk's eight `MailTransactionStarted` occurrences — steps 5 and 6 `Event`/`Given` rows,
   steps 8, 9 and 10 `Given` rows, the forward completeness table, the backward completeness
   table. Done (d19f91d).
2. Steps 6 and 9 `Given` blocks collapse to one event, existence only. Done (cd06274).
3. `TransactionState`'s dataset and name — the step-6 ruling this exploration was spawned from.
   Dataset: done (cd06274). Name: still open — the acknowledgment view's name, on this header's
   still-open list; not settled here.
4. The backward completeness rows for both `250 OK` lines: they currently claim field origins for a
   line with no varying value, and must take the honest form line 315 already uses for the `354`.
   Done (cd06274).
5. The forward completeness table: `MailTransactionStarted` → the new name, existence only at every
   site. **This orphans `reverse_path`,** which invalidates the walk's claim that
   `ConnectionAccepted.local_address` is the single unconsumed field. Do not patch around it — the
   orphan is a real finding, and the field's actual consumer is step 12's emitted
   `MessageAccepted.reverse_path`, which no `Given` declares. Done (d19f91d); the orphan was then
   discharged by the payload check becoming completeness's third (27627f9), which seated step 12's
   `Given` — the arc the v2 walk's forward table records.
6. `docs/FOLLOW-UPS.md` — the three-fates step-6 mismatch is discharged by the same edit and its
   entry updated or removed. Done (cd06274) — the entry was removed; its cross-repo successor is
   now bidirectional with the step-6 note.

**When this branch lands:**

7. `README.md` — *The working stance* says *"The three `EXPLORE-` files"*. There are four. Left
   untouched while the branch is open, so the statement stays true on `main`.

**At the prefix drop:**

8. `event-model.md` (9 + 4 hits), `model-altitude.md` (3 + 1), `diagrams/README.md` (2 + 3) — and
   `model-altitude.md` is where the altitude ruling that justifies the rename has to live, since
   the choice of register is an altitude decision.
9. The method repo's `state-view-todo-list-decision-model.md`, which cites `TransactionState` three
   times including the parked step-6 question, and whose sentence *"The model defined it as a view
   answering Open? Reverse path? How many recipients? for `RcptTo`, `BeginData` and
   `SubmitContent`"* is independently unsupported — only `BeginData` names it as a precondition
   (`event-model.md:271`).

**Never:**

10. The three superseded walked paths and the three `EXPLORE-` records. They keep the old name.
   Rule 14: exploration and walk material is read for its trail.

---

## Open, and deliberately not decided here

- **The name for `DataPhaseEntered`.** `DATA` declares nothing, so the rule's declaration shape does
  not fit. H1 rides on this and should not be closed cheaply.
- **The symmetry at step 8.** If `MAIL FROM`'s event names the client's declaration, consistency
  pushes `RecipientAccepted` toward `ForwardPathDeclared`, with acceptance carried by the reply.
  §3.3 confirms both commands can be refused, so the asymmetry cannot be defended on the ground that
  only RCPT is adjudicated. But the 550 branch has to attach somewhere, and today it attaches to an
  event named for the accept path.
- **The acknowledgment view's name.** `TransactionState` describes state it will no longer carry.
  Naming waits on the one-slice-or-two question, which the *steps 6 and 9* section above answers as
  one — but that answer is this file's, not a ruling.
- **Where the transaction-start fact goes.** §3.3's *"reset all its state tables and buffers"* is
  real protocol behavior — a second `MAIL FROM` resets. This path never exercises it, and `Reset` is
  already listed among the slices no walk has touched. It belongs with the D.2 walk.
- **Whether `ConnectionAccepted` and `MessageAccepted` are a third category** — events naming our
  decision rather than the peer's action. The rule passes them; nobody has attacked that.
- **Command naming.** Separate thread, recorded here because it surfaced in the same session: the
  seven commands follow an unwritten rule — *name for the wire verb where one exists, for the intent
  where none does* — which predicts six of seven, with `BeginData` the principled exception because
  `DATA` spans two steps. Nothing to change; the rule wants writing down.
