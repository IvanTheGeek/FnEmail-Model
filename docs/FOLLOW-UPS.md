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
  evidence pointer updated in the same move); the gate sequence and the four tiers
  (`model-altitude.md` §2.2–§2.3), once a second modeled system tests them.

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
- **Consumer or key?** `RecipientResolved.forward_path` in step 8's `Given` reaches no fold and
  no render — it only selects *which* record the seeded event concerns. The `session_id` shape,
  inside the `Given` grammar itself; decides whether the data degree's definition ("folded into
  state or rendered") needs a third clause.

**Discharged before import** (verified, listed so the transcript is never re-mined): the
payload-check gap (became *Completeness closes in three checks*); the walk's
zero-method-citations claim (the SMTP-layer reduction and the step 6 note added them); the three
predicted accounting knock-ons of the collapse (did not materialize — traversal-with-empty-
dataset still counts; re-verify the accounting once at prefix drop).
