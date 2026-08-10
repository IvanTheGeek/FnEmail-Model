# Model altitude — FnEmail's working rules

Status: **working rules, v0.2 — the altitude apparatus is no longer applied.** Companion to
`event-model.md`. What this document still decides: the charter (§2.1), the payload rule (§2.4),
the register ruling behind the `ReversePathDeclared` rename (§3), the model's boundaries (§4.3),
and the open questions (§6). The filename is unchanged so inbound links resolve.

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

> **Split again 2026-08-10 — and the apparatus unadopted.** §2.2's five-gate sequence and §2.3's
> four tiers are **parked in the method repo's extensions register**,
> [`EventModeling/docs/extensions.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/extensions.md),
> where the family already keeps inventions that are deliberately not applied; §3's nine classified
> events are **retired to git**. The scheme was an agent's invention rather than the owner's —
> commit `d0f1913` says so in its own message — and it had begun generating exceptions rather than
> answers. **Not all of it was ours: G1 and G2 are Dymitruk's and G5 is Dilger's**, and those tests
> keep their corpus footing wherever they are used on their own; the invented part is the five-gate
> *sequence*, and G3 and G4 specifically. Replacing one ranked ladder: **two axes** — a **lane**
> axis, business up and technical or infrastructure down, hideable, which is Dymitruk's and
> Dilger's own device; and a **provenance** axis, did the specification force this or did we choose
> it. The tier word was a lossy summary of a citation that should simply be kept — instead of
> *Domain*, the clause; instead of *Product*, the MAY or SHOULD; instead of a judgment call, "no
> normative clause, stated as a bare declarative". **Section numbers are again unchanged**; stubs
> stand in the numbered positions. This is a standing change and not a correction — the scheme was
> accepted and is now unadopted — so it carries no ⚠️ block, and the record is in the commit.
>
> **§2.1's charter is now unanchored.** Its stated justification — *"altitude is not a property of
> a fact; it is a relation between a fact and a charter"* — existed to make the altitude question
> answerable, and that question no longer has an apparatus behind it. The charter itself stands,
> unchanged and still load-bearing: §2.4, §3's register ruling and §4.3 all read from it. Its
> replacement is unruled, so nothing here is rewritten to fit.
>
> Two things are left as they stood. §6 keeps every question as written, Q1–Q4 included, which were
> doubts about this very scheme; and the moved-notices at §2.0 and §5 still name gates in passing,
> because they summarize material that now lives in the method repo.

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

**Parked 2026-08-10 in the method repo's extensions register,
[`EventModeling/docs/extensions.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/extensions.md),
and no longer applied here.** Five gates applied in order — G1 destination, G2 state change,
G3 charter, G4 substitution, G5 interpretation — with a fact that failed an earlier gate never
reaching the later ones. **G1 and G2 are Dymitruk's and G5 is Dilger's**, and each keeps its corpus
footing wherever it is used on its own; what is parked is the invention, meaning the *sequence*
and G3 and G4. Nothing is restated here — git holds the text:
`git log -p -- docs/model-altitude.md`.

### 2.3 The four tiers, defined

**Parked 2026-08-10 in the same entry —
[`EventModeling/docs/extensions.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/extensions.md)
— and no longer applied here.** *Domain*, *Product*, *Protocol* and *Infrastructure* were ours
entirely; the corpus has no such taxonomy. Each tier word summarized a citation lossily, and the
citation is the part worth keeping: the clause, its MUST or its SHOULD, or the absence of any
normative clause. Where other documents still speak in tiers, the vocabulary is flagged where it
stands rather than silently rewritten. Git holds the text.

### 2.4 Payload rule: store the decision, derive the encoding

Ours. Corollary of G3+G4. When a fact and its wire encoding are both available, the payload
carries the **decision**; the encoding is derived at the edge — *unless* the encoding is itself
emitted, permanent output, in which case it is payload by the same argument `event-model.md`
already makes for `received_at`:

> "The `Received:` timestamp must survive replay unchanged… It is business-relevant output, which
> by Dilger's own test makes it payload."

*The derivation line above — "Corollary of G3+G4" — points at gates that are no longer applied
(§2.2), and is left standing rather than rewritten. The rule does not rest on them: what it turns
on is the distinction between a decision and its wire encoding, plus the exception for an encoding
that is itself emitted as permanent output. Both are testable directly, and `received_at` is the
worked case.*

---

## 3. FnEmail's events, classified

**Retired to git 2026-08-09.** The nine event subsections, the classification table that headed
them, and *Pattern across the nine* were the apparatus applied: every verdict in them was a tier
assigned by the gates of §2.2, so they go where the gates and the tiers went. Nothing is restated
here — `git log -p -- docs/model-altitude.md` holds the text. The arguments that outlived the tiers
were already written elsewhere: H5 and `ConnectionAccepted`'s single-field claim into
`event-model.md` (commit 9d967e6), H1's two consumers into
`docs/paths/WORKING-helo-direct-single-recipient-v2.md`, and H8 into `event-model.md`'s register.
§6's model-level questions, which arose from these subsections, stay as written.

One subsection stays, below. *Registers* is a **charter** ruling rather than a tier ruling — it
turns on which of RFC 5321's two registers this model speaks — and it never used the gates or the
tiers.

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
