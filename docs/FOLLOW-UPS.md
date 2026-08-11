# Cross-repo follow-ups

Work this repository cannot finish in-repo, tracked here because it has no source document to
live at. Everything in-repo tracks at its source instead — see the README → *Where status lives*.
Absorbed from `HANDOFF.md`'s open items when that file dissolved, 2026-08-08.

## EventModeling

- **When the `WORKING-` prefix drops**, three deep links there need the new filename:
  `layering.md`, `path-and-step-form.md`, `state-view-todo-list-decision-model.md` (all link the
  v2 walk). Coordinate with this repo's own rename sweep.
- **Create the departures register** — its AGENTS rule requires labeling departures, but no
  register document exists yet. Two entries wait on it: the minimal-`Given` departure
  (the reasoning moved to the method repo's `path-and-step-form.md` on 2026-08-08, when
  `paths/STEP-FORM.md` was reduced to the SMTP layer), and the author's disagreements with Adam's
  method — mentioned repeatedly, never captured; a disagreement is a different category from an
  extension.
- **Flow candidates when ripe**: the cadence verdict (`paths/EXPLORE-rewalk-cadence.md`); the
  GWT-vs-timeline analysis and the view-`When` corpus contradiction (`paths/EXPLORE-gwt-form.md`,
  `paths/EXPLORE-view-slice.md`); `diagrams/README.md`'s generic halves (slice anatomy, chairs,
  one-event-per-command, no-arrows, the sequence-diagram rejection); `DECISIONS.md`'s held method
  rows; the EXPLORE-/WORKING- prefix convention; the rendering experiments (with `rendering.md`'s
  evidence pointer updated in the same move).
- **The payload check may be ours alone** — Cratis runs the backward and forward completeness
  checks and has no payload equivalent ([`cratis-analysis.md`](cratis-analysis.md) §6). That makes
  the third check of *Completeness closes in three checks* a contribution rather than a
  restatement, and the independent convergence on the other two is evidence worth carrying with it
  when it flows.
- **Q4 — does one model get one altitude?** A question owed to that repo's `altitude.md`, which
  now holds the split test and the lane material it bears on. Arrived 2026-08-10 from
  `model-altitude.md` §6, which routed it here rather than into the parked apparatus: unlike Q1 to
  Q3 it is not a doubt about the invented gates, so parking them left it standing. As §6 put it,
  this project assumed one altitude per model and pushed collapse to the boundary, following Ch. 5,
  while Ch. 7's Fig. 7.4 shows System A translating raw records into domain events inside its own
  boundary — arguably two altitudes in one system. It bears on the **lane axis**, which is the
  authors' own device and adopted rather than parked, so it belongs with live work.

## ModelingTool-Model

- **Cratis Screenplay is the closest public prior art to the tool** — a declarative modeling
  language whose unit is the typed slice, with Given/When/Then in the language, a published EBNF
  grammar, and the commit trail of a language being designed in the open. Read it before resolving
  a construct twice. [`cratis-analysis.md`](cratis-analysis.md) §2.
- **Mermaid's native `eventmodeling` diagram type is an output-format candidate** — v11.15.0+,
  merged upstream 2026-04-07, grammar published separately, and it already carries instance data.
  It did not exist as a considered option when that repo was created, and it is live *there* and
  not here: rule 5's **No Mermaid** turns on the Android app's missing diagram engine, which a tool
  emitting to other renderers does not face. `cratis-analysis.md` §5.
- **`Stage` is the standing alternative to generation** — an interpreter over the model as the
  deployable, so the model and the app are one artifact. Bears on what the future `FnEmail` code
  repo is for; recorded so that choosing generation becomes a stated position rather than an
  unexamined default. `cratis-analysis.md` §3.

## event-modeling-research

- `UPSTREAM-DEFECTS.md` cites the deleted `EXPERIMENT-block-labels.md` under the pre-rename repo
  name — repoint to the commit-pinned permalink (in commit c64f483's message).
- `PR-HANDOFF-eventmodelers-kit.md` is a verified, ready, **never-submitted** upstream fix —
  re-verify before submitting; it may already be fixed upstream.
- **D3**: is `MD_STR` missing from Mermaid's `block` grammar deliberately or by omission? Read
  `block.jison` against the flowchart grammar.
- `app.eventmodelers.ai` — the platform the archived skills are written against — remains
  unexplored; the 2018 Medium article still has no URL.
- Connection quirks worth keeping: `medium.com` returns 403 to curl (Cloudflare, not an egress
  block); the `eventmodelers.ai` apex fails TLS — use `www.`.

## Parked questions with no source document yet

- **Vocabulary residue** — the DDD-avoidance premise was refuted 2026-08-06 and *context* keeps
  its name; what remains open is only whether an EM-native name would be *clearer* on its own
  merits. `event-modeling-research:research/model-altitude-theory.md` §2.0d.
- **Retryability in the payload** — should `RecipientRejected` gain
  `disposition: permanent | transient`, promoting the retryability decision out of the reply code?
  Same question for `MessageRejected`. Arrived here 2026-08-10 as `model-altitude.md` §6's Q7,
  when that document became a signpost and its open-questions section stopped being a home; it is
  live and unanswered, and it lands in `event-model.md` as a hotspot on those two events if it is
  taken up rather than settled (AGENTS.md rule 12). Git holds §6 as written.
- **Domain values for `TransactionAborted.cause`** — should it take
  `client_reset | client_quit | connection_lost` rather than SMTP verb names, keeping verb names
  out of payload? Same arrival, `model-altitude.md` §6's Q8, and the same two dispositions.

## Docs-review findings, 2026-08-08 — imported from transcript, verified against the tree

A full review of the v2 walk ran in a parallel session before the `ReversePathDeclared` package
landed; its findings lived only in that session's transcript. Imported 2026-08-08 after
re-verifying every item against the current tree (13 parallel checks; three of the review's
claims had been discharged by intervening commits, one had its fix direction reversed). Each row
leaves this section when worked, tracked at its source thereafter — this is a waiting room, not
a ledger.

**Mechanical — no ruling needed, fix when convenient:**

- **The responsibility boundary is placed one step early.** The walk says responsibility begins
  at `MessageAccepted` and the `250` merely informs; §2.1 makes the handoff occur at reply
  *issuance* ("once the server has issued a success response") and §6.1 says "by sending a
  '250 OK' message". Failure case: crash between event and reply — the walk's placement owns a
  message the client will legitimately retry. Bears on the step 14 `queue_id` question below.
- **`event-model.md` presents a spliced paraphrase as a verbatim §2.1 quote** ("…accepting
  responsibility for delivering or reporting the failure to do so") — not §2.1's text; it grafts
  §6.1's frame onto §2.1's tail and drops the moment-fixing clause. Rules 2 and 3.
- **`actual_octets`: 194 is wrong.** The displayed step 12 message is 126 octets (CRLF, dot line
  excluded); 194 is unreachable under any counting. Fix the walk's three occurrences; the
  superseded sibling walks keep their wrong counts as trail (412, 287 — also wrong).
- **"Unchanged from the original except `BY`" is false** — `WITH` also changed source (from
  `ClientIdentified.protocol` to renderer constant).
- **`bar.com` sits in a code span in the Scene alignment sentence** — the file's one instance
  value in monospace.
- **The `Given`-block/Required-first parallel is overstated in both repos.** Required first's
  label row carries text (*before this walk's timeline*) and puts one row per event — the very
  form the ruling forbids. What genuinely matches: the empty left label and the sub-header
  reading. Soften in the walk preamble and in the method repo's `path-and-step-form.md`.
- **§4.3.1 is quoted as a flat MUST** ("the client MUST wait for every reply") — the greeting
  reply is SHOULD, and the command-reply MUST is conditioned on no negotiated extensions. H4's
  evidence item 3; the serialization argument survives restated.
- **The `Opt-info` ABNF quote is truncated mid-rule** — `[Additional-Registered-Clauses]`
  dropped with no elision mark.
- **The walk's "H5 is resolved" points the wrong way.** `event-model.md` deliberately keeps H5
  open (its residue is H7's decision); the walk's flat claim is the inaccurate side. Reword the
  walk, not the model.
- **H6's two passages disagree** — step 13's note says `local_address` has its "only candidate
  consumer"; the Hotspots section correctly says two. The note is the stale side.
- **The layering block still says "ruled by Ivan"** — the method repo retracted that to working
  hypothesis, and the retraction swept only that repo. Owed: a ⚠️ block at the walk's layering
  preamble and a `DECISIONS.md` register fix (the stale do-not-re-litigate row would wrongly
  invoke rule 9 against live work).
- **Method-side circularity, attenuated but standing**: `state-view-todo-list-decision-model.md`
  cites the walk as authority for the renderer/view rule its own sibling
  `path-and-step-form.md` states normatively. Repoint one citation.
- **README says "The working rules — thirteen"; there are fifteen.** One word — or drop the
  count so it cannot drift again. Fix on `main` (file identical on both branches).
- **D.4 sits unaccounted inside an absolute claim.** "No appendix scenario is *both* direct and
  single-recipient" — D.4 is single-recipient, local, sender's-own-domain client, all-2xx/3xx,
  and has no prose, so its topology is *unstated*, not relayed. Amend the claim to name it.
- **A cross-repo step-number citation is outside the sanction.** The method repo's three-fates
  doc cites "the walk's step 6" by number from outside the walk — now bidirectional with the
  step-6 note. Step numbers are positional facts *within one walk*; a renumber (live risk while
  the step 7 question below is open) silently strands it. Coordinated two-repo reword.

**Ruling-gated — the walk-through owner's, in order of leverage:**

- **The dataset cascade, three sites remaining, only two on the walk's own pending list.**
  `DataPrompt.awaiting_content` (step 11) and `MessageQueued.accepted` (step 14) are
  constant-true booleans arguably dead under the existing constant precedent.
  `SessionClosing.cause` (step 16) is the deciding case: the first *non-constant* field that
  renders nothing — killing it makes `SessionClosed.cause` a second flagged-unconsumed value;
  keeping it concedes render-failure alone does not kill. `MessageQueued.queue_id` is fully
  originated and consumed at steps 12/13, making step 14's copy destination-free — but §2.1's
  constitutive `250` (first item above) is the strongest argument that step 14 is not a mere
  copy. One ruling can settle all four; the backward table's step 14 and 16 rows correct in the
  same motion.
- **What is step 7 for?** `RecipientDirectory` has no top row and no reader — no `Given` can
  cite a view, so a consulted view is invisible to all three completeness checks, and
  "consulted" is a category with exactly one instance that nothing consumes. Load-bearing at
  `event-model.md` (H3), `model-altitude.md`, `diagrams/README.md`; deletion renumbers steps
  8–16. Needs a ruling on how a consulted view is read on a path, not an edit.
  **Input for that ruling, not a ruling:** Cratis answers the same question by making the consult a
  declaration on the *reader* — `reads <ReadModel> by <property>` on the command — and warns when a
  read model nothing produces is read. It matches this repo's *"a consulted view has no top row —
  its readers declare it"*, reached independently. [`cratis-analysis.md`](cratis-analysis.md) §2.2.
- **Consumer or key?** `RecipientResolved.forward_path` in step 8's `Given` reaches no fold and
  no render — it only selects *which* record the seeded event concerns. The `session_id` shape,
  inside the `Given` grammar itself; decides whether the data degree's definition ("folded into
  state or rendered") needs a third clause.

**Discharged before import** (verified, listed so the transcript is never re-mined): the
payload-check gap (became *Completeness closes in three checks*); the walk's
zero-method-citations claim (the SMTP-layer reduction and the step 6 note added them); the three
predicted accounting knock-ons of the collapse (did not materialize — traversal-with-empty-
dataset still counts; re-verify the accounting once at prefix drop).
