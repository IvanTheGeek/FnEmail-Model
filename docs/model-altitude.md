# Signpost — where `model-altitude.md`'s contents went

Status: **signpost. This document decides nothing.** It is kept because documents in this repo and
in the method repo cite it, and several of those are trail material — walked paths, `EXPLORE-` and
`EXPERIMENT-` files — which AGENTS.md rule 14 keeps as trail rather than edits. **The filename and
every section number are unchanged**, so a citation to §2.2, §3 or §4.3 still lands on a line that
says where that section went. Nothing here may be cited as authority.

The file was *Model altitude — FnEmail's working rules*, opened 2026-08-05. It held a theory of
altitude, a charter rule, a five-gate sequence for what belongs in a model, a four-tier taxonomy
for what does, a classification of nine events, a payload rule, the model's boundaries, and ten
open questions. Four moves emptied it — 2026-08-06 to the private research repo, 2026-08-08 to the
method repo, 2026-08-09 to git, and 2026-08-10 the unadoption of the apparatus and the ruling that
there is no charter. Each move's reasoning is in the commit that made it, and
`git log -p -- docs/model-altitude.md` walks the removed text in full.

The last of those moves is a **standing change, not a correction**: the gates, the tiers and the
charter were accepted while they stood, and being unadopted is not the same as having been wrong.
So nothing here carries a ⚠️ block, and the record is in the commits.

---

## Where each section is now

| Section | What it held | Where it is now |
|---|---|---|
| **§0, §1** | What the corpus says about altitude — the theory, and the term search behind the absence claim. §1.1 (no global model of an order) and §1.4 are cited from elsewhere | Moved 2026-08-06 to the private research repo, `event-modeling-research:research/model-altitude-theory.md`. A §1.4 citation traveled onward with §4's generic half in the 2026-08-08 move |
| **§2** | The heading over the working rules: everything under it was ours, cited where it had corpus footing | Nothing remains under it |
| **§2.0** | Role collapse — every corpus oracle assumes the expert and the modeler are different people, and written gates existed to simulate the separation that is missing | Moved 2026-08-08 to the method repo, [`EventModeling/docs/altitude.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/altitude.md) → *Role collapse* |
| **§2.0b–§2.0d** | Vocabulary, and the DDD question | Moved 2026-08-06 with §0 and §1 to `research/model-altitude-theory.md`. The vocabulary residue is tracked in [`FOLLOW-UPS.md`](FOLLOW-UPS.md) |
| **§2.1** | Rule zero — write the charter first — and FnEmail's one-sentence charter | **Deleted 2026-08-10.** See below |
| **§2.2** | The five-gate sequence: G1 destination, G2 state change, G3 charter, G4 substitution, G5 interpretation, applied in order | Parked 2026-08-10 in [`EventModeling/docs/extensions.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/extensions.md) §9, unapplied. **G1 and G2 are Dymitruk's and G5 is Dilger's and are not parked** — they keep their footing as individual tests. Parked are the ordered sequence, G3 and G4 |
| **§2.3** | The four tiers — *Domain*, *Product*, *Protocol*, *Infrastructure* | Parked in the same `extensions.md` §9 entry. The tier word summarized a citation lossily; the citation is what is kept instead |
| **§2.4** | The payload rule — store the decision, derive the encoding, unless the encoding is itself emitted as permanent output | Moved to [`event-model.md`](event-model.md), where `received_at` is already the worked case. Its old derivation line, *corollary of G3+G4*, does not travel: the rule turns on the decision/encoding distinction and its emitted-output exception, both testable without the gates |
| **§3** | FnEmail's nine events, each classified by tier, and *Pattern across the nine* | Retired to git 2026-08-09 with the apparatus that produced the verdicts. The arguments that outlived the tiers had already been written elsewhere — H5 and `ConnectionAccepted`'s single-field claim into `event-model.md` (commit 9d967e6), H1's two consumers into the v2 walk, H8 into `event-model.md`'s hotspot register |
| **§3, *Registers*** | The two-register reading behind the `ReversePathDeclared` rename — RFC 5321 states every transaction command in both an intent register and a buffer/state register and does not choose between them | The ruling stands and is registered in [`DECISIONS.md`](DECISIONS.md); the screening is [`paths/EXPLORE-declaration-vs-status.md`](paths/EXPLORE-declaration-vs-status.md), adopted in commit d19f91d. Only the rename was ever adopted. The text is in git, and is not restated here: its stated grounds were charter-relative, and there is no charter |
| **§4, §4.1, §4.2** | When to split into a separate model — the corpus default, the three-part split test, the interface asymmetry | Moved 2026-08-08 to `EventModeling/docs/altitude.md` → *When to split into a separate model* |
| **§4.3** | Applied — FnEmail's three boundaries (Directory, outbound relay, any consuming business) and their interfaces, all landing on the boundary RFC 5321 itself draws at §2.1 | Moved to [`event-model.md`](event-model.md), where H3's resolution and the `MessageAccepted` handoff already sit |
| **§5** | The same fact at two altitudes, worked — two models sharing exactly one event | Moved 2026-08-08 to `EventModeling/docs/altitude.md` → *The same fact at two altitudes* |
| **§6** | Ten open questions, four method-level and six model-level | Routed one by one below |

---

## §2.1 — there is no charter

§2.1 held a rule and a sentence. The rule was that every model states in one sentence what it is a
model of, because altitude is a relation between a fact and a charter rather than a property of the
fact. The sentence, versioned v0.3, described FnEmail as a conformant inbound RFC 5321 + RFC 7504
server, operated by us, whose obligations begin at `MessageAccepted`.

**On 2026-08-10 the owner ruled that there is no charter.** The sentence is deleted rather than
demoted, and it is not preserved here as though it still stood; git holds both it and rule zero.

Two things replace it, neither of them a charter:

- **The normative set**, in [`event-model.md`](event-model.md) — the specifications conformance is
  claimed against, each with its archived and read status. This is the normative part, and it is
  what the provenance question reads. Widening the set creates a reading queue, not a conformance
  claim.
- **A product statement**, in the [README](../README.md) — plain prose saying what FnEmail is. Not
  normative, not an input to any test, and the owner's to change without a ruling.

Where other documents still read from the charter, they are corrected at their own sites and in
their own commits, not here.

---

## §6 — where the ten questions went

**Q1 to Q3 went with the apparatus they doubted.** All three are quoted in the parked entry,
`EventModeling/docs/extensions.md` §9, under *Why it was unapplied*: Q1 that G3 and G4 have no
corpus support, Q2 that *product* may be no tier at all, Q3 that a different charter would
reclassify most of §3 in one stroke. The entry reads them as the shape of something applied faster
than it was justified, and keeps them as the record of that.

**Q4 — *Does one model get one altitude?* — does not go there.** It is not a doubt about the
invented apparatus, so parking the apparatus leaves it standing: it asks whether Ch. 7's Fig. 7.4,
where System A translates raw records into domain events inside its own boundary, contradicts the
one-altitude-per-model assumption. It bears on the **lane axis**, which is the authors' own device
and is adopted, not parked — so it belongs with live work, not in the archive. It has no source
document in this repo, which is exactly what [`FOLLOW-UPS.md`](FOLLOW-UPS.md) is for: it belongs
there under *EventModeling*, as a question owed to the method repo's `altitude.md`, which now holds
the split test and the lane material it bears on.

The model-level six:

| Question | Where it is now |
|---|---|
| **Q5** — H1 restated: does the operator need to distinguish abandonment before the body from abandonment during it? | ✅ **Answered.** The v2 walk gave the event two consumers on one page. `event-model.md`'s *Hotspots* carries H1 as answered on existence, with the event's **name** the only part still open |
| **Q6** — does `SessionClosed` survive a forward completeness pass? | **Registered as H8** in `event-model.md`'s *Hotspots*, which names this question as its origin. Blocked on the dataset cascade; decide jointly with `Quit`'s postcondition |
| **Q7** — should `RecipientRejected` and `MessageRejected` gain a `disposition` field taking `permanent` or `transient`, promoting retryability out of the reply code? | **Live, and unanswered.** Moved 2026-08-10 to [`FOLLOW-UPS.md`](FOLLOW-UPS.md) → *Parked questions with no source document yet*. It lands in [`event-model.md`](event-model.md) as a hotspot on those two events if it is taken up rather than settled |
| **Q8** — should `TransactionAborted.cause` be renamed to domain values, removing verb names from payload? | **Live, and unanswered.** Moved with Q7, same destination and same two dispositions |
| **Q9** — is *policy decisions are facts, protocol slips are not* consistent across every error branch in all twelve slices? | **Registered as H2** in `event-model.md`'s *Hotspots*, where the line is recorded as defensible and unverified. Not yet walked |
| **Q10** — write the `ConnectionAccepted` reasoning into the model, since its claim to exist rests on a single field | ✅ **Done 2026-08-06.** Commit 9d967e6 wrote the H5 argument into `event-model.md` and corrected the tier it rested on while doing so |
