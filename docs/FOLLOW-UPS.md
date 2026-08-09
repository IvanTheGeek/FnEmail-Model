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
re-verifying every item against the current tree. Each row leaves this section when worked,
tracked at its source thereafter — this is a waiting room, not a ledger, and it has nearly
emptied: the fifteen mechanical items were worked by the 2026-08-08 reconciliation commits, the
dataset cascade closed under *Don't invent a new test — the view's own definition already
decides it*, the step 7 question closed when the consulted box dissolved (2026-08-09, walk
renumbered), and the cross-repo step-number reword was applied to the three-fates doc. One row
remains:

- **Consumer or key?** `RecipientResolved.forward_path` in `RcptTo`'s `Given` reaches no fold
  and no render — it only selects *which* record the seeded event concerns. The `session_id`
  shape, inside the `Given` grammar itself; decides whether the data degree's definition
  ("folded into state or rendered") needs a third clause.

**Discharged before import** (verified, listed so the transcript is never re-mined): the
payload-check gap (became *Completeness closes in three checks*); the walk's
zero-method-citations claim (the SMTP-layer reduction and the step 6 note added them); the three
predicted accounting knock-ons of the collapse (did not materialize — traversal-with-empty-
dataset still counts; re-verify the accounting once at prefix drop).
