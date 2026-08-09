# Model altitude — FnEmail's working rules

Status: **working rules, v0.1.** Companion to `event-model.md`. Resolves the framing behind H1,
H2 and H5.

> **Split 2026-08-06.** This document's first half — §0 and §1 on what the corpus says about
> altitude, and §2.0b–§2.0d on vocabulary and the DDD question — moved to the private
> `event-modeling-research` repo, at `research/model-altitude-theory.md`. FnEmail uses Event
> Modeling but is not about it. **Section numbers here are unchanged**, so the sequence starts at
> §2; references below to §0 and §1.1 point into that document *(a §1.4 reference left with §4's
> generic half in the 2026-08-08 move below)*.
>
> What stayed is everything that decides what goes in *this* model: role collapse, the charter,
> the gate sequence, the four tiers, the payload rule, and the classified events.

> **Split again 2026-08-08.** The remaining generic sections — §2.0 role collapse, §4–§4.2 when
> to split, §5 the two-altitudes worked example — moved to the public method repo, as
> `EventModeling/docs/altitude.md`. Ivan ruled that generic method material belongs there now
> that the repo exists; this overrides the 2026-08-06 split's choice to keep them here, whose
> premise — no method repo to hold them — no longer held (recorded per rules 4 and 9). Section
> numbers are again unchanged; stubs below say where each section went. What stays is what
> decides *this* model: the charter (§2.1), the gates (§2.2), the tiers (§2.3), the payload rule
> (§2.4), the classified events (§3), the applied boundaries (§4.3), and the open questions (§6).
> The gates and tiers are themselves flow candidates once a second modeled system tests them.

---

## 2. Our working rules

**Everything in §2 is ours.** Where a rule has corpus footing it is cited; where it has none, that
is stated.

### 2.0 The problem the corpus did not anticipate: role collapse

**Moved 2026-08-08 to `EventModeling/docs/altitude.md` → *Role collapse*.** The short form: every
corpus oracle assumes the expert and the modeler are different people; when every role sits in
one head that signal vanishes, and written gates exist to *simulate the separation that is
missing* — which is why §2.2's gates are phrased as questions with answers. It also holds the
constraint-oracle/product-expert distinction: the RFC rejects, it never proposes, and every
question the RFC leaves open (H6, H7, the HELO-only scope) has turned out to be a product
question.

### 2.1 Rule zero: write the charter first

**Every model states, in one sentence, what it is a model of.** Altitude is not a property of a
fact; it is a *relation between a fact and a charter*. This follows directly from §1.1 — if there
is no global model of an order, the question "is this protocol or domain" is unanswerable until you
name the context that is asking.

FnEmail's charter, as of v0.3:

> **A conformant inbound RFC 5321 + RFC 7504 server, operated by us, whose obligations begin at
> `MessageAccepted`.**

*(`MessageAccepted` is the model's name for the obligation moment. §2.1 and §6.1 place the
handoff itself at the issuance of the `250` that follows `DATA`; the event is its record.)*

Read that charter twice. It contains "conformant" and it names an RFC. **This is what makes SMTP
mechanics domain rather than protocol here** — not the fact that we happen to implement SMTP.

### 2.2 The gate sequence

Apply in order. A fact that fails an earlier gate does not reach later ones.

| # | Gate | Question | Corpus footing |
|---|---|---|---|
| **G1** | **Destination** | Does anything in this model consume it? | Dymitruk l. 167 — **direct** |
| **G2** | **State change** | Does it change what the system will accept or produce next? | Dymitruk l. 120 — **direct** |
| **G3** | **Charter** | Is it a fact *about* the charter, or about how we happen to satisfy it? | ours |
| **G4** | **Substitution** | Does it survive replacing the implementation? | ours |
| **G5** | **Interpretation** | Could an outsider read it without our rules? | Dilger Ch. 5 — **direct**, but scoped to boundary crossing |

**G1 — Destination.** *All information has to have an origin and a destination.* A fact with no
consumer is not "low altitude"; it is **not in the model**. This gate is unforgiving and it does
most of the work. It already killed `ServiceGreetingSent`. Note the corollary: a fact can be
promoted into the model by *adding* a consumer, which is why altitude arguments must always name
the consumer rather than appeal to taste.

**G2 — State change.** *"only state-changing events are to be specified… "guest viewed calendar for
room availability"… they are not events."* For a protocol server, "state" includes **which commands
are now legal**. This is what saves several SMTP facts that would otherwise look like transcript
noise.

**G3 — Charter.** Distinguishes **domain** from **product**. A fact required by the charter is
domain. A fact arising from a choice we made, which we could have made differently while still
satisfying the charter, is **product**. Test: find the RFC clause. **MUST** → domain.
**MAY/SHOULD**, or no clause at all → product. Both are modeled; the difference is that product
facts move when the product moves, and domain facts do not.

**G4 — Substitution.** Distinguishes **protocol** and **infrastructure** from the two above.
Two forms, and you must apply the right one:

- *Wrong form:* "would this exist if we didn't use SMTP?" For FnEmail this is meaningless — a
  charter naming RFC 5321 has no substitution set.
- *Right form:* **"would this exist under a different conformant implementation of the same
  charter?"** Different socket library, different concurrency model, different process
  architecture, different storage — does the fact survive? If yes, it is not infrastructure. And
  separately: **"does this fact exist only because of the wire's encoding, or does it exist in the
  conversation the wire is encoding?"** A phase transition, a line terminator, a three-digit code
  are encoding. An identity claim, a return path, a refusal are conversation.

  Corpus adjacency, not support: *transport is "an implementation detail"* (Ch. 5, p. 92) is the
  same shape of argument applied to Kafka-vs-CSV.

**G5 — Interpretation.** Only for facts crossing a model boundary. *"A system using these events
needs to know exactly what it means to 'add an item'."* If a consumer would need our rules to make
sense of it, it stays inside. This is the gate that forbids exporting reply codes.

### 2.3 The four tiers, defined

Ours. The corpus has no such taxonomy (§0).

| Tier | Definition | In the model? |
|---|---|---|
| **Domain** | The charter is *about* this fact. Passes G1–G4. Survives any conformant reimplementation. | Yes — first-class event |
| **Product** | A fact produced by a decision we made and could remake while staying conformant. Passes G1, G2, G4; fails G3. | Yes — event, flagged as ours, expected to churn |
| **Protocol** | Exists only as an artifact of the wire encoding of the conversation. Fails G4's second form. | Only if the charter names the protocol *and* G1 passes — otherwise collapsed at translation |
| **Infrastructure** | Exists because of how we run the software. Fails G4's first form. | Never an event. May be metadata. |

**The single most important consequence:** *Protocol* is a tier only relative to a charter. For an
e-commerce model, everything below `MessageAccepted` is protocol and disappears. For FnEmail,
whose charter names the RFC, most of it is domain. **The facts did not move. The charter did.**

### 2.4 Payload rule: store the decision, derive the encoding

Ours. Corollary of G3+G4. When a fact and its wire encoding are both available, the payload
carries the **decision**; the encoding is derived at the edge — *unless* the encoding is itself
emitted, permanent output, in which case it is payload by the same argument `event-model.md`
already makes for `received_at`:

> "The `Received:` timestamp must survive replay unchanged… It is business-relevant output, which
> by Dilger's own test makes it payload."

---

## 3. FnEmail's events, classified

Charter per §2.1. Consumers available in this model: the `Received:` header (§4.4), rendered by
the `MessageTrace` view, the rendered reply views, and the queue handoff at `MessageAccepted`.
⚠️ Two former entries are flagged rather than restated: no `Given` on the v2 walk consults
`SessionState` (its rendering role became `SessionReady{server_domain}`, commit cf3227d), and
`TransactionState`'s dataset was emptied on the walked path (commit cd06274), and the 2026-08-09
naming split it: `ReversePathAllowed` at the `MailFrom` step, `RecipientConfirmed` after
`RcptTo` — the trail is in `docs/paths/EXPLORE-declaration-vs-status.md`.

| Event | Tier | Decided by | Survives implementation swap? |
|---|---|---|---|
| `ConnectionAccepted` | **Product** ⚠️ *was "Domain (fragile)"* | G1 via `peer_address`; **G3 → product**, §4.4's address literal is SHOULD | Yes — one field only |
| `ClientIdentified` | **Domain** | G2, G4 | Yes |
| `ReversePathDeclared` | **Domain** | G2, G4 | Yes |
| `RecipientAccepted` | **Domain** | G2, G4 | Yes |
| `RecipientRejected` | **Domain** + product field + protocol field | G2, G4; §2.4 on `reply_code` | Yes |
| `DataPhaseEntered` | **Product**, not protocol | G1 — two consumers on the v2 walk | No, as drawn |
| `MessageAccepted` | **Domain** — the domain fact | all gates | Yes |
| `TransactionAborted` | **Domain**, protocol-named values | G2, G4 | Yes |
| `SessionClosed` | **Product**, weakest in the model | G1 marginal, G4 fails | No |

### ConnectionAccepted — Domain, on one field

A TCP accept is infrastructure by G4: it survives no reimplementation as a *fact*, only as an
event in the operating system's sense. It is in the model for exactly one reason, and
`event-model.md` already states it:

> "Exists to carry `peer_address`, which the `Received:` header requires and nothing else supplies."

G1 passes because the `Received:` FROM clause consumes the address literal, which is permanent
output on the message. It resolves H5's classification: not "domain fact or infrastructure noise"
as a matter of judgment, but *infrastructure in shape, **admitted by consumer*** — the residue,
whether FnEmail commits to emitting the address literal, folds into H7, and `event-model.md`
keeps H5 open on exactly that. If the `Received:` requirement vanished, the event would go the
way of `ServiceGreetingSent`.

> ⚠️ **Corrected 2026-08-06 — "mandates" was wrong, and the tier moves.** This paragraph read
> *"§4.4 mandates the FROM address literal"*, which would make `peer_address` domain by G3. The
> clause does not say that:
>
> > *"The FROM clause, which **MUST** be supplied in an SMTP environment, **SHOULD** contain both
> > (1) the name of the source host as presented in the EHLO command and (2) an address literal
> > containing the IP address of the source, determined from the TCP connection."*
>
> **The clause is MUST; its contents are SHOULD.** A server supplying a FROM clause with only the
> EHLO name is conformant. G3 is explicit — *MUST → domain, MAY/SHOULD → product* — so
> `ConnectionAccepted` is **Product**, and the table above is corrected.
>
> **This is why it read as fragile.** It was not fragile domain; it was product misfiled as domain,
> and product facts are expected to churn. The discomfort the "(fragile)" annotation recorded was
> the misclassification itself.
>
> Note also where the RFC names TCP. It says *transmission channel* throughout and abstracts the
> transport — except in this clause, *"determined from the **TCP connection**."* The one place the
> specification names the transport is the one place this model admits a transport fact. The full
> argument now lives in `event-model.md` under H5, which is what this paragraph asked for.

The v2 walk adds an existence footing: a `554`-refused greeting still has a session — §3.1 makes
waiting for the client's `QUIT` a MUST — so `ConnectionAccepted` must exist on the refused path
too, contradicting the model's recorded contract *reply 554, no event* (the rule 4 correction is
pending in `event-model.md`). `peer_address` stays the only field-level consumer, so the tier
argument is untouched.

`local_address` (H6) still fails G1 on the walked path — the v2 walk found two candidate
consumers, the multi-homed `Received:` BY arm and the nameless-server `220` greeting, since the
greeting's identity slot admits an address literal (§4.2) — and exercises neither. Which arm, if
either, wins stays open. Correctly flagged rather than kept on faith.

### ClientIdentified — Domain

Named for the state change, not the verb. That naming is doing real work: `HELO` is protocol, but
*"the peer has claimed an identity"* is the conversation the protocol encodes, and it survives G4 —
any submission protocol has this moment. G2 passes hard: it gates every subsequent write — later
commands cite `ClientIdentified` directly in their `Given`s, the fold that `SessionState.identified`
once named having fallen with the booleans-nothing-renders ruling (commit cf3227d).
`claimed_domain` has a permanent destination (`Received: FROM`).

⚠️ **The `protocol: "SMTP"` field fell, exactly as this section predicted.** The paragraph that
stood here kept the field solely because `Received: WITH` required it as output, and warned: *a
constant-valued field is normally a smell; note explicitly why this one stays, or a later reader
will delete it.* The later reader did, for exactly that reason — in a `HELO`-only charter the
value cannot vary, so `WITH`'s value joined the renderer's constants and the field left the
command and the event (commit 2673b1b, recorded at the walk's `Helo` step). If the charter ever
admits `EHLO`, the field re-enters as a fact originated by the verb that actually arrived.

### ReversePathDeclared — Domain

`reverse_path` is where delivery failure reports go. That is a business fact in any mail system
whatsoever and survives G4 outright. Named for the client's declaration — the thing that happened —
rather than for the wire verb (`MAIL FROM`) or for the receiver's state transition, which was the
old name's register: `MailTransactionStarted` spoke §3.3's own vocabulary (see *Registers* below).
The candidate screening lives in `docs/paths/EXPLORE-declaration-vs-status.md`; the rename was
adopted in commit d19f91d.

### RecipientAccepted — Domain

Accepting a recipient is a decision with a consequence: the address enters the delivery set that
`MessageAccepted` will carry, and responsibility attaches to it. `is_local` is arguably product
(our routing policy), but it is sourced from the Directory context (H3, resolved — a separate
context joined by Translation; see `event-model.md`) and is a fact about the address space, so:
domain.

### RecipientRejected — Domain event carrying one product field and one protocol field

The rejection is domain: it is a policy decision, and the model's H2 line ("policy decisions are
facts, protocol slips are not") is exactly G2 plus G4 applied consistently. Corpus adjacency: the
kit's own instruction to keep what the business acts on and drop the raw signal.

But the payload mixes tiers, and §2.4 exposes something the model has not yet said:

- `reason` — product. Ours, changeable, no RFC clause.
- `reply_code` — **protocol encoding of two different things at once.** `550` vs `553` is a
  presentation choice (product). `5xx` vs `4xx` is **permanent vs transient**, which tells the
  sender whether to retry. Retryability is a domain fact wearing protocol clothes, and right now
  it exists in our model **only** as a digit.

**Proposal (ours, not in the model):** add `disposition: permanent | transient` as payload, and
keep `reply_code` as the emitted-output exception. Then G5 also becomes satisfiable — an outside
consumer can read the disposition without knowing SMTP.

### DataPhaseEntered — H1, answered by the walk

G4's second form fails — a phase transition is an artifact of SMTP's line-oriented
command/response encoding, not of the conversation — so the event is **product**, not protocol.
G2 passes narrowly (it does change which input is legal). G1, the live question this section used
to carry, is answered by the v2 walk: *"On this page it has two consumers: `DataPrompt` folds it
to render the `354`, and `SubmitContent` declares it as `Given`."*
(`docs/paths/WORKING-helo-direct-single-recipient-v2.md`, its ✅ H1 block.) Formal closure in
`event-model.md` is pending model work — and whether a `Given` citation counts as a consumer, the
consumer-or-key question, is genuinely open, so the count is the walk's, on the walk's terms.

What H1 leaves open is the event's **name**. `DataPhaseEntered` is this document's own
wire-named-event case, and `DATA` declares nothing, so the declaration reading that produced
`ReversePathDeclared` has no purchase here. Tracked in
`docs/paths/EXPLORE-declaration-vs-status.md`; no name is proposed here.

### MessageAccepted — Domain, and the model's integration event

Every gate passes. This records the responsibility transfer — §2.1 and §6.1 place it at the
issuance of the `250` that follows `DATA`, and `MessageAccepted` is its record — and its
payload is already Dilger-shaped: `queue_id`, `content_ref`, `recipients[]`, `actual_octets`,
`received_at` — **pre-calculated data sufficient for a downstream system, requiring none of our
rules to interpret.** That is G5 satisfied, and it is `Cart Submitted` by another name:

> "It's much better to provide a "Cart Submitted" event that already contains the pre-calculated
> data necessary for an external system to process it."

**`MessageAccepted` is the one event that appears in every other business's model too.** §5.

### TransactionAborted — Domain, with protocol leakage in the values

"The sender abandoned the transaction" survives G4. But `cause: rset | quit | disconnect` names two
SMTP verbs in payload values. By the kit's no-leakage rule — *translate the business meaning, not
just map fields* — these should be domain-named: `client_reset | client_quit | connection_lost`.
Minor, real, cheap to fix, and it prevents the vocabulary from hardening.

### SessionClosed — Product, and the weakest event in the model

G4 fails: closing a connection is infrastructure. G1 is marginal — on the v2 walk its consumer is
the `SessionClosing` view, rendering the `221` from `ServiceConfigured.server_domain` and
`SessionClosed.cause`, a named view rather than a generic transcript end-marker — and
`TransactionAborted` already carries the abandonment fact. `cause: timeout | shutdown` is
operational telemetry, not domain; whether `cause` survives at all sits inside the open
dataset-cascade question — the render-failure test for a non-constant field — flagged in the v2
walk and not settled here.

It survives as **product**, on the same footing as `DataPhaseEntered`: the operator wants sessions
to be countable and terminable in the transcript. But it has not been subjected to the forward
check — of the three-check completeness pass, *Completeness closes in three checks* (commit
27627f9) — that deleted `ServiceGreetingSent`, and by symmetry it should be.

**Proposed new hotspot, H8 — Does `SessionClosed` earn its place?** Note the coupling: deleting it
leaves `Quit` (`Quit`) with no post-condition event, which is legal under H2's rule but odd for a
non-error command. Decide the two together. (`event-model.md`'s banner records H8 as proposed but
not yet registered there.)

### Pattern across the nine

Every event that passes G4 is **named for the conversation, not the wire** — `ClientIdentified`,
not `HeloReceived`; `ReversePathDeclared`, not `MailFromAccepted`. Every event on trial
(`DataPhaseEntered`) or weak (`SessionClosed`) is named for the wire. **Naming predicts tier**, and
this is a cheap review heuristic: an event named after a verb or a phase is a protocol event until
it proves otherwise.

### Registers — the altitude ruling behind the rename

RFC 5321 states every transaction command in two registers: the client's intent — `MAIL` is
*"used to initiate a mail transaction"* (§4.1.1.2) — and the receiver's buffer/state machinery —
the same command *"tells the SMTP-receiver that a new mail transaction is starting and to reset
all its state tables and buffers"* (§3.3). §4.1.1 even carries a complete alternative model at
the buffer altitude: *"there is a reverse-path buffer, a forward-path buffer, and a mail data
buffer"* — a coherent model of SMTP, and not this one. The RFC does not choose between registers,
so the choice is a charter decision. This charter's conversation altitude selects the intent
register, and that selection is what renamed `MailTransactionStarted` — §3.3's own vocabulary —
to `ReversePathDeclared`. The screening is `docs/paths/EXPLORE-declaration-vs-status.md`, adopted
in commit d19f91d; only the rename is adopted — the exploration's generalized naming rule is not
settled here.

---

## 4. When to split into a separate model

**§4's generic half — the corpus default (don't; reach for a lane first), the three-part split
test, and the interface asymmetry (one collapsed event out, Translation in) — moved 2026-08-08 to
`EventModeling/docs/altitude.md` → *When to split into a separate model*.** What remains here is
its application:

### 4.3 Applied — FnEmail's three boundaries

| Boundary | Split? | Interface |
|---|---|---|
| **Directory / provisioning** (H3, resolved) | **Yes** — different room (ops/provisioning), different language (mailbox ≠ forward path) | Inbound. `RecipientDirectory` is the Translation boundary onto the separate Directory context; the upstream model publishes mailbox and relay-authorization facts, we translate them into the read model. `event-model.md` was right that the hole *"names a slice that must exist upstream"* — the resolution confirmed exactly this split. |
| **Outbound delivery / relay** | **Yes** — the roles invert (already documented: *"they command, we reply"* becomes *"we command, they reply"*, and the automation lives there) | Outbound. `MessageAccepted` is the integration event. Responsibility transfers at the `250`'s issuance, which `MessageAccepted` records (§2.1, §6.1). |
| **Any consuming business** (e-commerce, notifications) | **Yes**, trivially | Outbound. `MessageAccepted` again — collapsed to one event on their board. §5. |

Note that all three interfaces are the boundary the RFC itself draws at §2.1. The responsibility
transfer — at the issuance of the `250` that follows `DATA` — the integration event that records
it, and the model boundary coincide. That is a good sign and worth saying out loud in
`event-model.md`.

---

## 5. The same fact at two altitudes — worked

**Moved 2026-08-08 to `EventModeling/docs/altitude.md` → *The same fact at two altitudes*.** The
short form, kept because §3 cites this section: an order confirmation leaving an e-commerce system
and arriving at FnEmail is one wire instant appearing in two models, both complete, both correct —
**two models that share exactly one event**, `OrderConfirmationSent` on their side and
`MessageAccepted` on ours. Everything below `MessageAccepted` fails G1 *in their model*; the count
of shared events measures the boundary's quality, and one is healthy.

---

## 6. Open questions

**Method-level**

- **Q1. No human oracle.** Every corpus test for what belongs in a model ends at a business expert.
  Ours ends at an RFC. G3 (MUST vs MAY) and G4 (substitution) are our substitutes, and **neither
  has any corpus support**. If they are wrong, most of §3 is wrong. Unverified.
- **Q2. Is "product" a real tier?** It may just be "domain we chose rather than inherited." It
  earns its keep here because it decides `DataPhaseEntered` and `SessionClosed`, but it has no
  corpus basis whatsoever, and a simpler three-tier scheme might do the same work.
- **Q3. G4's substitution set is stipulated, not derived.** We declared that "different socket
  library" is a substitution and "different protocol" is not. That declaration comes from the
  charter, and the charter is ours. A charter reading *"deliver mail for our users"* rather than
  *"conformant RFC 5321 server"* would reclassify most of §3 in one stroke. **Is our charter the
  right one?**
- **Q4. Does one model get one altitude?** We assumed yes and pushed collapse to the boundary,
  following Ch. 5. Ch. 7's Fig. 7.4 shows System A translating raw records *into* domain events
  within its own boundary — arguably two altitudes inside one system. Unresolved in the corpus.

**Model-level, arising from §3**

- **Q5. H1 restated:** does the operator need to distinguish abandonment-before-body from
  abandonment-during-body? Answer decides `DataPhaseEntered` outright.
  ✅ **Answered by the v2 walk** — two consumers on the page; formal closure in `event-model.md`
  pending; the event's name remains open (H1 residue, tracked in
  `docs/paths/EXPLORE-declaration-vs-status.md`).
- **Q6. New — H8:** does `SessionClosed` survive a forward completeness pass? Decide jointly with
  `Quit`'s post-condition.
- **Q7.** Should `RecipientRejected` gain `disposition: permanent | transient`, promoting the
  retryability decision out of the reply code? Same question for `MessageRejected`.
- **Q8.** Should `TransactionAborted.cause` be renamed to domain values
  (`client_reset | client_quit | connection_lost`) to remove verb names from payload?
- **Q9.** H2's line — "policy decisions are facts, protocol slips are not" — is G2+G4 applied
  consistently, but it is still **unverified**. A `503` changes no state and produces no
  consequence; a `550` records a decision. Does that hold for every error branch in all twelve
  slices? Not yet walked.
- **Q10.** H5 is answerable now (`ConnectionAccepted` = infrastructure in shape, domain by
  consumer) but the reasoning should be written into `event-model.md`, because the event's claim to
  exist rests on a single field and a future reader will not reconstruct that.
  ✅ **Done 2026-08-06** — commit 9d967e6 wrote the H5 argument into `event-model.md`, and
  corrected the tier it rests on while doing so. *(The line above ending "claim to exist rests on
  a single field" was lost in the 2026-08-06 repo split, commit 49b6d71, which truncated this file
  mid-sentence; restored 2026-08-08 from commit 0edeb96 — an accident repaired, not a claim
  changed.)*
