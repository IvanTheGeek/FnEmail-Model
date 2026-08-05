# Domain, Product, Protocol — deciding model altitude

Status: **working rules, v0.1.** Corpus-grounded where possible, ours where not, and marked either way.
Companion to `event-model.md`. Resolves the framing behind H1, H2 and H5.

---

## 0. Read this first: the corpus has no theory of altitude

Nothing in the primary sources answers the question "how much detail belongs in a model."
This was confirmed by full reads plus grep. In Dilger Ch. 8 (Domain-Driven Design) the terms
**"altitude", "granularity", "level of detail", "abstraction", "zoom", "protocol",
"infrastructure", "subdomain", "core domain", "context map"** appear **zero times**. The same
terms are absent from Ch. 5 (Internal vs External Data), Ch. 7 (Stream Design) and Ch. 44 (DCB).
Dymitruk's canonical article contains no discussion of choosing a level of detail. The
`agent-modeling-kit` skills have no vocabulary for it and, decisively, **no mechanism for a second
model at all** — every skill resolves one `BOARD_ID` and appends to one
`.trogonai/interviews/[project-name]/EVENTMODELING.md`.

Three further absences worth stating plainly, because they are the exact shape of our question:

1. **No domain / product / protocol / infrastructure taxonomy exists in the corpus.** The only
   axes offered anywhere are *internal vs external* (Dilger Ch. 5) and *business vs technical*
   (Dilger Ch. 7, undefined). Dilger says Event Modeling means "we do not focus on technical
   concepts" but never defines "technical concept," so it cannot adjudicate whether an SMTP reply
   code is technical.
2. **No criterion for when something deserves its own model.** Dilger discusses when to split a
   *system* (answer: don't) and how to find *aggregates* (answer: name the cluster). Ch. 7's
   decomposition unit is the swimlane and never rises above it; Ch. 44's is the command handler's
   decision model and never rises above that. Ch. 7 explicitly defers the question: *"In the next
   chapter, we will look a bit deeper into exactly this when we widen our horizon and look beyond
   the technical measures."*
3. **Dilger disclaims the DDD chapter as guidance.** *"This is neither a comprehensive description
   of DDD in general, nor will we touch all necessary areas of it."* … *"While I am not an expert
   in DDD… This definition is one perspective among many."*

**Consequence for this document:** §2 onward is largely **ours**. It is marked. Anyone attributing
it to Dymitruk or Dilger is misciting them.

Note also §2.0, which corrects a claim in an earlier draft of this document about who the business
expert is. The methodological gap is real but it is **role collapse**, not an absent person.

---

## 1. What the corpus does say, and how much weight it bears

Five statements bear on altitude. Ranked by how much work they can actually do.

### 1.1 There is no context-free correct altitude — strongest statement in the corpus

> "It would be futile and silly to even try to define a global model for an order. It simply won't
> work. The concept of an order always has to be defined within a clear context so everybody knows
> what is being discussed at the moment."
> — Dilger, Ch. 8 (`0152-domain-driven-design.txt`, ll. 175–178)

> "we have made the task of trying to fit all our concerns into one model an unnecessary constraint"
> — Dymitruk, `what-is-event-modeling/index.md` l. 27

This settles the premise of the question rather than answering it. *"What altitude is email
really at"* is malformed in the same way *"what is the global model of an order"* is malformed.
The one-command/one-event collapse and FnEmail's twelve slices are not competing answers; they are
different contexts. Neither is a coarsening of the other.

### 1.2 Origin and destination — the corpus's one usable, mechanical test

> "At this time the event model should have every field accounted for. **All information has to
> have an origin and a destination.** Events must facilitate this transition and hold the necessary
> fields to do so. This rigor is what is required to get the most benefits of the technique."
> — Dymitruk, l. 167

This is the sharpest tool available and it is already load-bearing in `event-model.md`: it is what
deleted `ServiceGreetingSent` ("origin but no destination"), what put `DataPhaseEntered` on trial,
and what flagged `local_address`. It is not *about* altitude — but it prunes altitude in practice,
because most protocol detail has no destination.

### 1.3 The translation example — the phenomenon, never generalised into a rule

> "we may get events from guests' GPS coordinates… **We would not want to use longitude and
> latitude pairs as events to specify preconditions in our system. We would rather have events
> that mean something to us like "Guest left hotel", "Guest returned to hotel room".**"
> — Dymitruk, l. 95

Dilger works the same example from the other side: technical records (userId, lat, long) on a Kafka
topic, translated by System A into `User entered store`, while System C consumes the raw records and
never lifts them at all (Ch. 7, pp. 119–121, Fig. 7.4 *From Record to Domain Event*).

The kit makes it a prohibition:

> "**No Leakage**: Don't expose external IDs/data structures in your event model" …
> "`title` | `<DomainEventName>` (translated name, **not the external event name**)" …
> "We ignore: latitude, longitude (we just care that guest left)"
> — `eventmodeling-translating-external-events/SKILL.md`, ll. 310, 466, 294–299

**This is our exact question, worked once, by example, in one direction only.** Lat/long is to a
hotel what a reply code is to a retailer. But no source states the rule, and none discusses the
case where the lat/long *is the product* — i.e. FnEmail. The kit even names the collapse case
("Need aggregation/multiple events"), classifies it as "complex", and then declines to give a
criterion for it.

### 1.4 Three variants of "ask a human"

- **Naming test (scoped to aggregates):** *"In discussions with your business experts, you should
  try to find out what the real name of the "cluster" is. What term do they use in their meetings
  for this? If you can´t find a good name for it, most probably it´s not (yet) detailed enough to
  become an aggregate."* (Ch. 8, ll. 254–263)
- **Isolated-narrative test (scoped to stream boundaries):** *"hide all swimlanes but one and just
  read the events from left to right in isolation to someone from the business side who cannot see
  the model. The events should form a compelling narrative and a consistent story. If not, the
  business person will tell you immediately."* (Ch. 7, p. 123)
- **Puzzled-faces test (scoped to snapshots):** *"Talking to business about snapshots will almost
  always result in puzzled faces and question marks. Whereas mentioning that the cash register has
  been closed for the day will be understandable by the business people involved."* (Ch. 7, pp. 125–126)

All three route the decision to **who is in the room**. None was stated about altitude; extending
them to altitude is **our inference**. Note the fatal complication for FnEmail in §2.0.

### 1.5 The two-tier default: full detail inside, one collapsed event outside

> "It's much better to provide a "Cart Submitted" event that already contains the pre-calculated
> data necessary for an external system to process it. From the perspective of the cart-system,
> this is an integration event and serves as a contract with external systems."
> — Dilger, Ch. 5, p. 91

The test for what may cross:

> "A system using these events needs to know exactly what it means to "add an item." What does it
> mean for prices? … **This is all domain knowledge of the cart domain. It should not be
> distributed across the system.**" — Ch. 5, pp. 90–91

The book states the N-inside/1-outside collapse as the **normal correct arrangement, not a
compromise**. Every system has two altitudes by construction. Also from Ch. 5: transport is not
architecture — *"You can use some stable middleware like Apache Kafka or RabbitMQ, but it could
also be a CSV file uploaded to a network share. From the architectural perspective, it's basically
all the same."*

### 1.6 Boundary count is organisational

> "**Team Structure & Ownership** (Impact: Determines how many swimlanes/systems to create)"
> — `eventmodeling-applying-conways-law/SKILL.md`, l. 24

> "We need to do this to allow the system to exist as a set of autonomous parts that separate teams
> can own." — Dymitruk, l. 159

And the standing default when unsure:

> "If you need to make a decision early, most of the time it´s the best conscicous decision to not
> split but keep everything in one system until you know more." — Ch. 8, ll. 150–152 *(sic)*

---

## 2. Our working rules

**Everything in §2 is ours.** Where a rule has corpus footing it is cited; where it has none, that
is stated.

### 2.0 The problem the corpus did not anticipate: role collapse

Every corpus test for what belongs in a model terminates in a human oracle — the business expert
who *"will tell you immediately"*, who has a term for it in their meetings, who makes a puzzled
face at snapshots.

**An earlier draft of this document said FnEmail has no such person, and that our business expert
is RFC 5321, a document that cannot be puzzled. That was wrong on both counts.**

FnEmail has a business expert. Its author is building the mail system he wants and releasing it
for others to use freely, and in doing so holds every role: domain expert, product expert,
designer, implementer, maintainer.

The real problem is subtler and worth stating precisely.

**The corpus tests work because the expert is a different person from the modeller.** The
puzzled-faces test carries information *because* the business person does not know the mechanics —
their puzzlement is a signal generated by ignorance the modeller does not share. *"The business
person will tell you immediately"* works because they are hearing the narrative cold.

When every role sits in one head, that signal vanishes. Not for lack of expertise — **because you
cannot surprise yourself.** Nobody makes a puzzled face at their own model.

So the gap is not a missing person. It is **role collapse**: the roles exist, they are simply not
separable, and the oracle mechanism depends on their separation.

This changes what the written gates are for. They are **not proxies for an absent expert** — they
are a way to *simulate the separation that is missing*. A rule committed to while wearing one hat
binds the other hat later. That is precisely why writing them down beats holding them in mind, and
why §2.2's gates are phrased as questions with answers rather than as judgement calls.

#### Role collapse is a phase, not a property

It ends with adoption. Once others use FnEmail, their requests, feedback and bug reports supply
outside analysis, and the roles separate again without anyone being hired.

That makes the written gates **scaffolding for a phase** rather than permanent infrastructure —
but it does not make them temporary, because adopter feedback is a *different* oracle from a
workshop expert in three ways:

| | Workshop expert | Adopter feedback |
|---|---|---|
| **When** | before building | after building |
| **Selection** | whoever is in the room | only those who adopted *and* hit a problem — the "why I didn't adopt" signal never arrives |
| **What they see** | the model | the software |

The third is the sharp one. A user can report that `550` fired when it should not have. A user
cannot report that `DataPhaseEntered` is an event that should not exist, because they will never
see the model.

**Two things convert product feedback into model-level signal, and both are already planned:**

1. **`event-model-extensions.md` §8 — paths as a diagnostic artifact.** If a support case arrives
   as an event log and a catalog of known paths exists to diff it against, a bug report becomes
   *"here is a walk through your model that you did not predict."* That is the bridge from
   after-the-fact feedback to design-time critique.
2. **Publishing the model.** Model-level feedback is only possible if the model is readable by
   someone who is not its author. The hotspots are already phrased as questions to a future reader
   — H1, H2, H6 and H7 are open invitations. An adopter *can* be puzzled by H1, but only if H1 is
   somewhere they can find it.

So the gates matter most now, and the hotspots matter most later.

#### The RFC is a constraint oracle, not a product expert

A second correction from the same conversation, and it resolves a pattern already visible in the
model's hotspots.

| Source | Can answer | Cannot answer |
|---|---|---|
| **RFC 5321 + RFC 7504** | *Must* we? Is this conformant? | *Should* we build it? For whom? |
| **The author** | both — and is the only source for the second |

The RFC is **requirements plus a conformance constraint**. It rejects; it never proposes. It is
not a product expert and cannot be consulted as one.

Every question so far where the RFC permits both answers has turned out to be a product question:

- **H6** — multi-homed with per-address hostnames? RFC silent.
- **H7** — ever refuse mail entirely (`521`)? RFC silent.
- **HELO-only scope** — RFC permits either. Ours.

This is also the two-source structure behind `event-model-extensions.md` §4: normative rules come
from the constraint oracle, operator rules come from the product expert.

### 2.0b Vocabulary — provisional

Terms in play, unresolved: *context, altitude, detail, level, layer, tier, concern*. A first cut at
which are independent, offered to be argued with rather than adopted:

| Term | What it appears to be | Independent axis? |
|---|---|---|
| **Context** | what the model is a model of — the charter | **primary; everything else is relative to it** |
| Altitude / level / detail / granularity | how much detail | **no** — a consequence of naming the context (§1.1) |
| **Layer** | one model sitting beneath another — protocol under domain | yes |
| **Concern** | cross-cutting aspect: security, GDPR, operations, metadata | yes |
| Tier | classification of a *fact* within one model (§2.3) | derived bookkeeping |

If that holds, there are **three** genuinely independent things — context, layer, concern — and
altitude is not a dial anyone sets. It falls out of the charter. This mirrors the reduction of four
patterns to two slice types: several named things collapsing into fewer real ones.

Re-sorting the parked extensions along those axes:

| Extension | Axis |
|---|---|
| §1 protocol / domain / operational models | **layer** |
| §2 GUI as its own model | **context** — different actors, different charter |
| §5 business vs technical errors | **layer** |
| Dilger's "missing chapters": metadata, security, GDPR, UI, organization | **concern** |

Held loosely. The terms are still being worked out.

### 2.0c Testing the vocabulary — two results

#### Result 1: "domain" is Dilger's word, not Dymitruk's

Term counts across the archived corpus:

| Source | business | domain | behavior | "business behavior" |
|---|---|---|---|---|
| **Dymitruk, canonical article** | 3 | **0** | **0** | 0 |
| Dilger, `agent-modeling-kit` skills | 93 | 148 | 7 | 1 |
| Dilger, *Understanding Eventsourcing* | 209 | 65 | 34 | 2 |

Zero occurrences of "domain" in Dymitruk's foundational article. His three "business" hits are
incidental (*"a timeline of the year in that business"*); none is definitional. The single
"behaviour" is a reference to **Behaviour Driven Development**, not his own usage. "Business
behavior" appears three times in the entire corpus and is not a term of art anywhere.

The domain/business vocabulary is **Dilger's**, brought from DDD.

**But the absence is deliberate, not incidental — and this correction matters.** Dymitruk knows DDD
well; he avoids its vocabulary on purpose, so that Event Modeling stays legible to anyone who walks
up to a model, and so that EM is not conflated with DDD. The stated position is that Event Modeling
*implements* DDD concepts without requiring anyone to have read the blue book.

His written rationale is in the article's *Simplicity* section, and the target is unmistakable:

> *"if an organization chooses to adopt a process called 'X', and **X requires one book and a
> workshop that takes a week** to go through, it nullifies the effectiveness of X, and here's the
> worst part, **no matter how good X is**."*

Followed by the halving cascade — everyone says they read the book, half did, half of those claim
they understood it, half of those actually did.

So the finding stands but an earlier reading of it was wrong: this is **suppression of vocabulary,
not absence of the concept**.

⚠️ **Consequence for this document.** If EM's vocabulary is deliberately DDD-free, then importing
DDD terms into it works against the method's design intent — and "context" is Evans' *bounded
context* wearing a shorter name. Whether §2.0b's axes should be renamed in EM-native terms is an
open question. It is not merely cosmetic: the whole argument for the vocabulary is adoption cost.

⚠️ **Scope of this claim.** Only Dymitruk's *written* work is available — the podcast repo carries
no transcripts and video hosts are blocked by the egress policy. This is "absent from his canonical
written work", **not** "absent from his vocabulary". He may use these terms freely in talks.

**What he uses instead:** *information system*, *the story*, *workflow step*, *user*, and
*"empowering the user and informing the user."* His frame is **information and users**, not
business and domain.

This may be why Event Modeling fits a protocol server at all. A method organised around a business
domain would make SMTP an awkward guest. A method organised around how information changes over
time from a user's perspective does not care that the user is an MTA.

#### Result 2: layer and context do not separate as peer axes

The proposed test was **directional dependency** — a layer relationship means one model's events
feed another's, whereas two contexts merely coexist. It fails:

| Pair | Directional? | Intuition |
|---|---|---|
| FnEmail protocol -> FnEmail domain | yes | layers |
| FnEmail -> an e-commerce shop that sends mail | **also yes** | separate contexts |

The shop depends on an MTA to send. Same directionality, opposite intuition. Directionality is not
the distinguishing property.

**What distinguishes them is charter and ownership.** The shop and FnEmail have different charters
and different owners. The protocol/domain split has one charter and one owner.

So **layer is not a peer of context — it is a subdivision within one**:

> Subdivide within a context and you have **layers**. Subdivide until the pieces have separate
> charters and owners and you have separate **contexts**.

#### Revised axes

Two, not three:

| Axis | Definition |
|---|---|
| **Context** | What the model is a model of. One charter, one owner. Internally layerable. |
| **Concern** | Cross-cutting aspect — security, GDPR, operations, metadata. Orthogonal, because the same concern applies across contexts. |

Everything else is derived: *altitude / level / detail / granularity* fall out of naming the
context (§1.1); *tier* is per-fact bookkeeping (§2.3); *layer* is context, subdivided.

#### This unifies an existing mechanism

Conway's Law swimlanes are the layering mechanism, and they **become** contexts when ownership
separates far enough. Dilger's internal-vs-external distinction (Ch. 5) is the same boundary seen
from the data side — external data is another context's data.

> **swimlane -> layer -> context** is one continuum of separation, not three concepts.

That also predicts where the boundary sits in practice: the point at which a swimlane acquires its
own charter is the point at which it should become its own model.



### 2.1 Rule zero: write the charter first

**Every model states, in one sentence, what it is a model of.** Altitude is not a property of a
fact; it is a *relation between a fact and a charter*. This follows directly from §1.1 — if there
is no global model of an order, the question "is this protocol or domain" is unanswerable until you
name the context that is asking.

FnEmail's charter, as of v0.3:

> **A conformant inbound RFC 5321 + RFC 7504 server, operated by us, whose obligations begin at
> `MessageAccepted`.**

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
**MAY/SHOULD**, or no clause at all → product. Both are modelled; the difference is that product
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
| **Protocol** | Exists only as an artefact of the wire encoding of the conversation. Fails G4's second form. | Only if the charter names the protocol *and* G1 passes — otherwise collapsed at translation |
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

Charter per §2.1. Consumers available in this model: the `Received:` header (§4.4), the session
transcript, `SessionState` / `TransactionState`, and the queue handoff at `MessageAccepted`.

| Event | Tier | Decided by | Survives implementation swap? |
|---|---|---|---|
| `ConnectionAccepted` | **Domain** (fragile) | G1 via `peer_address` | Yes — one field only |
| `ClientIdentified` | **Domain** | G2, G4 | Yes |
| `MailTransactionStarted` | **Domain** | G2, G4 | Yes |
| `RecipientAccepted` | **Domain** | G2, G4 | Yes |
| `RecipientRejected` | **Domain** + product field + protocol field | G2, G4; §2.4 on `reply_code` | Yes |
| `DataPhaseEntered` | **Product**, not protocol — if kept | G1 (contested) | No, as drawn |
| `MessageAccepted` | **Domain** — the domain fact | all gates | Yes |
| `TransactionAborted` | **Domain**, protocol-named values | G2, G4 | Yes |
| `SessionClosed` | **Product**, weakest in the model | G1 marginal, G4 fails | No |

### ConnectionAccepted — Domain, on one field

A TCP accept is infrastructure by G4: it survives no reimplementation as a *fact*, only as an
event in the operating system's sense. It is in the model for exactly one reason, and
`event-model.md` already states it:

> "Exists to carry `peer_address`, which the `Received:` header requires and nothing else supplies."

G1 passes because §4.4 mandates the `FROM` address literal, which is permanent output on the
message. **This is the whole argument, and it should be written that way in the model.** It
resolves H5: not "domain fact or infrastructure noise" as a matter of judgement, but *infrastructure
in shape, domain by consumer*. If the `Received:` requirement vanished, the event would go the way
of `ServiceGreetingSent`.

`local_address` (H6) fails G1 today — infrastructure until multi-homing gives it a destination.
Correctly flagged rather than kept on faith.

### ClientIdentified — Domain

Named for the state change, not the verb. That naming is doing real work: `HELO` is protocol, but
*"the peer has claimed an identity"* is the conversation the protocol encodes, and it survives G4 —
any submission protocol has this moment. G2 passes hard: it gates every subsequent write via
`SessionState.identified`. `claimed_domain` has a permanent destination (`Received: FROM`).

The `protocol: "SMTP"` field is **protocol tier**, constant-valued in HELO-only scope. It earns its
place solely because `Received: WITH` requires it as output — §2.4's exception, same as
`received_at`. A constant-valued field is normally a smell; note explicitly *why* this one stays,
or a later reader will delete it.

### MailTransactionStarted — Domain

`reverse_path` is where delivery failure reports go. That is a business fact in any mail system
whatsoever and survives G4 outright. Named for the transaction rather than for `MAIL FROM` — same
good pattern as `ClientIdentified`.

### RecipientAccepted — Domain

Accepting a recipient is a decision with a consequence: the address enters the delivery set that
`MessageAccepted` will carry, and responsibility attaches to it. `is_local` is arguably product
(our routing policy), but it is sourced from the Directory hole (H3) and is a fact about the
address space, so: domain.

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

### DataPhaseEntered — the H1 resolution

As currently drawn: empty payload, one candidate consumer (the transcript rendering `354`).
G4's second form fails — a phase transition is an artefact of SMTP's line-oriented
command/response encoding, not of the conversation. G2 passes narrowly (it does change which input
is legal). G1 is the live question, and `event-model.md` already frames it correctly: *"its only
candidate consumer is the transcript rendering `354`."*

**But the tier is wrong, and that changes the decision.** `DataPhaseEntered` is not protocol —
it is **product**, because there is exactly one thing it records that nothing else does:

> A session that entered DATA and then died mid-body emits **no** `MessageAccepted` and **no**
> `MessageRejected`. Without `DataPhaseEntered`, the log cannot distinguish "abandoned after
> `RCPT`" from "abandoned after transferring 4 MB of body."

That distinction is an **operator** question — bandwidth consumed, abuse detection, whether a peer
is repeatedly failing mid-transfer. So H1 resolves to a criterion instead of taste:

> **Keep `DataPhaseEntered` iff the operator needs to distinguish abandonment-before-body from
> abandonment-during-body. If yes, it is a product event and should be documented as one — its
> consumer is the transcript, and its reason for existing is the abandonment case, not the `354`.
> If no, it fails G1 and should be deleted exactly as `ServiceGreetingSent` was.**

Note that G1 as currently applied looks only at the *success* path, which is why the answer has
seemed unclear. Applying the completeness check to the abandonment path is what settles it.

### MessageAccepted — Domain, and the model's integration event

Every gate passes. This is where responsibility transfers (§2.1, quoted in the model), and its
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

G4 fails: closing a connection is infrastructure. G1 is marginal — its only consumer is the
transcript, as an end-marker, and `TransactionAborted` already carries the abandonment fact.
`cause: timeout | shutdown` is operational telemetry, not domain.

It survives as **product**, on the same footing as `DataPhaseEntered`: the operator wants sessions
to be countable and terminable in the transcript. But it has not been subjected to the forward
completeness pass that deleted `ServiceGreetingSent`, and by symmetry it should be.

**Proposed new hotspot, H8 — Does `SessionClosed` earn its place?** Note the coupling: deleting it
leaves slice 11 (`Quit`) with no post-condition event, which is legal under H2's rule but odd for a
non-error command. Decide the two together.

### Pattern across the nine

Every event that passes G4 is **named for the conversation, not the wire** — `ClientIdentified`,
not `HeloReceived`; `MailTransactionStarted`, not `MailFromAccepted`. Every event on trial
(`DataPhaseEntered`) or weak (`SessionClosed`) is named for the wire. **Naming predicts tier**, and
this is a cheap review heuristic: an event named after a verb or a phase is a protocol event until
it proves otherwise.

---

## 4. When to split into a separate model

The corpus's default is emphatic: **don't.**

> "most of the time it´s the best conscicous decision to not split but keep everything in one system
> until you know more" — Dilger, Ch. 8

Splitting is not the corpus's normal tool for scale. **Swimlanes are.** The kit shows four
separately-owned, independently-deployed systems sharing **one timeline and one diagram**, and the
kit has no construct for a second model at all. Dilger's own decomposition unit is the swimlane
(*"Swimlanes define stream boundaries"*), and Dymitruk's is likewise the swimlane, motivated by
Conway's Law. **Reach for a lane before reaching for a model.**

### 4.1 Our test for a genuine split

A second model is warranted when **all three** hold:

1. **Different room.** A different set of people would be in the workshop, and the reading of the
   timeline that satisfies one group is gibberish to the other (§1.4's isolated-narrative test,
   run for both audiences).
2. **Different language.** A word means two different things across the line — Dilger's `Order` in
   Order Context vs Payment Context, where the attributes of one *"simply have no meaning"* in the
   other. If no word collides, you probably have a lane, not a model.
3. **Different rate of change.** *"Domain Events… will change over time, hopefully… The external
   view of our data is a completely different story. It's like a stable summary."* If both halves
   churn together, they are one model.

**Explicit non-criteria**, on Dilger's authority: being a separate deployment (*"That's a big
mistake, in my opinion… It's simply not defined"*), stream size (*"neither forbidden nor a
problem"*), and — post-DCB — needing a shared invariant, since DCB *"allows you to temporarily
construct strong consistency boundaries on demand"* and thereby breaks the old forcing function
from "must be consistent together" to "must be modelled together." (Ch. 44 is presented as
immature and contested; Greg Young's criticism is cited in it. Treat accordingly.)

### 4.2 The interface, and its asymmetry

The two directions are **not** mirror images. This is stated in both primary sources and should be
followed literally.

**Outbound — we publish one collapsed event.** One integration event, carrying pre-calculated data,
versioned, schema'd, and interpretable without our rules (G5). `MessageAccepted` already is this.
It costs a permanent versioning tax — *"External Events can and should be versioned"* — which is
itself a brake on exporting detail.

**Inbound — we translate through the Translation pattern.** Never consume a foreign vocabulary
directly:

> "Note that we keep to the convention of connecting events from swim lane 1 to state view, state
> view to processor, processor to command, command to event in swim lane 2." — Dymitruk, l. 95

The foreign event lands as a **view**, a processor reads it, and **we** issue a command producing
**our** event, under our name. The kit forbids the shortcut: no raw external IDs, no external
schema, the board carries *"the translated name, not the external event name."*

### 4.3 Applied — FnEmail's three boundaries

| Boundary | Split? | Interface |
|---|---|---|
| **Directory / provisioning** (H3) | **Yes** — different room (ops/provisioning), different language (mailbox ≠ forward path) | Inbound. `RecipientDirectory` is a typed hole; the upstream model publishes mailbox and relay-authorisation facts, we translate them into the read model. `event-model.md` is right that the hole *"names a slice that must exist upstream"* — this is the split, already correctly identified. |
| **Outbound delivery / relay** | **Yes** — the roles invert (already documented: *"they command, we reply"* becomes *"we command, they reply"*, and the automation lives there) | Outbound. `MessageAccepted` is the integration event. Responsibility transfers with it, per §2.1. |
| **Any consuming business** (e-commerce, notifications) | **Yes**, trivially | Outbound. `MessageAccepted` again — collapsed to one event on their board. §5. |

Note that all three interfaces are the boundary the RFC itself draws at §2.1. The responsibility
transfer, the integration event, and the model boundary coincide. That is a good sign and worth
saying out loud in `event-model.md`.

---

## 5. The same fact at two altitudes — worked

An order confirmation email leaves an e-commerce system and arrives at FnEmail. One instant on the
wire. Two models. Both complete. Both correct.

### The e-commerce model

```
Command   SendOrderConfirmation { order_id, customer_email }
Event     OrderConfirmationSent { order_id, sent_at }
Event     OrderConfirmationFailed { order_id, reason }      ← if they even have this
```

Why nothing below this line exists in their model, gate by gate:

- **G1 — no destination.** Nothing in an e-commerce model reads a `250`. No view renders it, no
  command is gated on it, no field of any screen or report traces back to it. By Dymitruk's rule it
  cannot be in the model. *This is sufficient on its own.*
- **G5 — they cannot interpret it.** Knowing what `550` means, when `4xx` implies retry, why a
  `421` is not a rejection — *"This is all domain knowledge of the cart domain. It should not be
  distributed across the system."* Substitute "mail" for "cart."
- **Kit rule — forbidden outright.** Reply codes are the external system's schema. *"No Leakage."*
  They may not go on the board.
- **The one survivor is a reference field.** If they need reconciliation, they keep our `queue_id`
  as `mailGatewayRef` — an attribute, never identity, never structure. Exactly the kit's
  `paymentGatewayRef: ch_123` shape.

Even `OrderConfirmationFailed` is questionable in their model: SMTP acceptance is not delivery, so
what they actually learn at that moment is *"handed off"*, and real failure arrives later as a
bounce — a different event from a different source. **Modelling their email as one event is not a
simplification. It is the accurate account of what their business knows.**

### FnEmail's model

Twelve slices, nine events, `Received:` reconstructable from the stream, transcript reconstructable
from the stream. The same instant appears as `MessageAccepted`.

### Why both are correct

**They are not two altitudes of one model. They are two models that share exactly one event.**

The e-commerce model's `OrderConfirmationSent` and FnEmail's `MessageAccepted` are the two sides of
one boundary — the internal/external pair Dilger describes, where full detail lives inside and a
stable summary crosses. This is *"the normal, correct arrangement"* in Ch. 5, not a compromise.
Asking which is the real model of email is asking for the global model of an order: *"futile and
silly."*

Two consequences worth keeping:

1. **The count of shared events measures the boundary's quality.** One is healthy. If the
   e-commerce model ever needed to branch on a `421`, the boundary would be wrong — either they
   have absorbed our domain knowledge (G5 violated) or we have failed to give them a fact they
   legitimately need (in which case add it to `MessageAccepted`'s payload as a *decision*, per
   §2.4 — never as a code).
2. **The collapse is not lossy from their side.** Nothing was thrown away that they could have
   used. Everything below `MessageAccepted` fails G1 *in their model*. Detail belongs where the
   rules that interpret it live.

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
- **Q6. New — H8:** does `SessionClosed` survive a forward completeness pass? Decide jointly with
  slice 11's post-condition.
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