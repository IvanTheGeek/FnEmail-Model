# Session Handoff

Everything a fresh session needs to pick this up. Written 2026-08-05 from a Claude Code container
session; revised 2026-08-06 on a local devbox, when the Event Modeling research was split into a
second repository; restructured 2026-08-08 after the four-repo split. **This file is only the
handoff** — what the project is, setup, reading order, open items, state. The working rules live
in [`AGENTS.md`](../AGENTS.md); the settled decisions in [`DECISIONS.md`](DECISIONS.md).

---

## 1. What this project is

**FnEmail** — an email system being designed with **Event Modeling** (Adam Dymitruk's method)
before any code is written. The repo contains **no code**: it is the model, the worked examples,
and the rules deciding what belongs in the model.

Current scope: **inbound SMTP, `HELO` only** (no ESMTP, no relay, no outbound). Deliberately
narrow, so the method is exercised properly before the surface grows.

The author is building this for himself and intends to release it freely.

### It serves two purposes, and the second one explains the first

**FnEmail is a real product, and simultaneously the first worked example for the modeling tool**
whose requirements the method repo is accumulating — the markdown here is a **prototype of
generated output**, which is why the documents get worked to a degree a single SMTP server would
never justify. See the `EventModeling` repo's README for the tool intent. Two operative
consequences survive in short form: a form that is merely tolerable to write by hand but **wrong
to read is the wrong answer**, because the tedium disappears when a tool emits it; and where a
relationship cannot be represented in markdown, **record the relationship and accept the flat
rendering** — never abandon the relationship. ⚠️ None of this is license to over-build: the
author's instruction (2026-08-07) is to get the basics working first and refine placement later.

### Paths are the source; slices are derived

Ruled 2026-08-07 and extended 2026-08-08, and it inverts the obvious reading of this repository.
`event-model.md` looks like the definition and the paths look like instances of it; **the flow of
authority is the opposite** — the walks are the source, and the model is derived as the union of
what every walk contributed. The canonical statement is now the method repo's
[`layering.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/layering.md) — step,
slice, path, workflow, and which concern lives at which layer; the long-form reasoning stays in
this repo's exploration records. In FnEmail terms: a slice's specification set is a **result, not
an input** — `Reset` remains untouched by every walk and therefore has no evidence behind it at
all — and rule 8 is architecture rather than advice.

> ⚠️ **An earlier version of this section held walk and workflow as *probably* distinct**, flagged
> as untested. The 2026-08-08 layering ruling answered it: a **workflow** is the composed, generic
> timeline part, and **a path is that workflow with the data filled in** — the same shape at two
> layers, not two concepts. Recorded per rule 4 rather than silently overwritten. What remains
> true: a single walk crosses several workflows — the direct walk's steps would sit across roughly
> four — and a workflow contains slices a given walk never touches.

### Four repositories

The repos mirror the modeling layers. The canonical ruling record is
[`DECISIONS.md`](DECISIONS.md); this is the operational map:

| Repo | Layer | Holds |
|---|---|---|
| [`EventModeling`](https://github.com/IvanTheGeek/EventModeling) (public, AGPL) | method | generic modeling material — layering, path/step form, the three-fates split, rendering requirements; future home of the modeling tool |
| **`FnEmail-Model`** (this repo — renamed from `FnEmail` 2026-08-08, old URLs redirect) | model | the SMTP model, walks, RFCs, SMTP-specific references. Cites the method repo |
| `FnEmail` (future) | code | what the model generates — not yet created |
| `event-modeling-research` (private) | sources | the mirrored corpus — cite, never reproduce |

Method-generic insight flows to `EventModeling` with FnEmail demoted to example; SMTP specifics
stay here (AGENTS.md rule 11). The first extraction moved `state-view-todo-list-decision-model.md`
there on 2026-08-08; its FnEmail history survives in git. This repo's default branch is **main**.

---

## 2. Get set up

**Nothing is on disk yet.** Clone all three repos. *(Verified working 2026-08-08.)*

```bash
# 1. The model — this repo. Work is on main.
git clone git@github.com:IvanTheGeek/FnEmail-Model.git /home/ivan/FnEmail-Model

# 2. The method repo — PUBLIC
git clone git@github.com:IvanTheGeek/EventModeling.git /home/ivan/EventModeling

# 3. The research — PRIVATE, needs auth
git clone git@github.com:IvanTheGeek/event-modeling-research.git ~/event-modeling-research
```

All use SSH. If you get a 404 on the private repo that means **not authenticated**, not *does not
exist* — it is the confusing failure mode. Generate a key with `ssh-keygen -t ed25519`, add the
`.pub` half at <https://github.com/settings/keys>, and check with `ssh -T git@github.com`.

Verify:

```bash
git -C /home/ivan/FnEmail-Model branch --show-current   # → main
ls /home/ivan/FnEmail-Model/docs                        # → event-model.md, paths/, diagrams/, rfc/
ls /home/ivan/EventModeling/docs                        # → layering.md, path-and-step-form.md, …
ls ~/event-modeling-research                            # → research/, extracted/, 6 root files
```

⚠️ **Never copy anything from `~/event-modeling-research` into the public repos** — AGENTS.md
rule 7. See that repo's `extracted/README.md` and `research/archive/NOTICE.md` before quoting from
the extracted book or the machine transcripts; the transcripts carry three caveats that matter.

---

## 3. Read in this order

Start with the method layer, then this repo, then the research repo as needed.

| File | Repo | Why |
|---|---|---|
| `README.md`, then `docs/` (six files) | EventModeling | **The method layer.** Layering, path/step form, the three-fates split, rendering, altitude, the parked extensions — the canonical statements this repo cites |
| [`DECISIONS.md`](DECISIONS.md) | this repo | What is settled, and must not be re-litigated (AGENTS.md rule 9) |
| [`event-model.md`](event-model.md) | this repo | The model — v0.3, 12 slices. **Lags the v2 walk deliberately**; its banner lists the pending reconciliations |
| [`model-altitude.md`](model-altitude.md) | this repo | What belongs in this model — charter, gate sequence, four tiers, the classified events. Its generic sections moved to `EventModeling/docs/altitude.md` 2026-08-08 |
| [`smtp-path-vs-mailbox.md`](smtp-path-vs-mailbox.md) | this repo | Path versus mailbox — the SMTP reference distinction the walks lean on |
| [`paths/`](paths/) | this repo | The walked paths. `WORKING-helo-direct-single-recipient-v2.md` is the active walk; `STEP-FORM.md` defines the step convention (its banner says what has moved on) |
| [`diagrams/README.md`](diagrams/README.md) | this repo | The model as tables — renders on GitHub *and* the Claude apps |
| `docs/extensions.md` | EventModeling | The author's extension ideas — **parked, deliberately not applied** (rule 10). Moved 2026-08-08 from this repo's `event-model-extensions.md` |
| `research/METHOD-REFERENCE.md` | research | The method as its authors define it, contradictions named rather than smoothed over |
| `research/CORRECTIONS-v0.1.md` | research | 20 ways the first draft misapplied the method |

**The working stance** (AGENTS.md rule 13): the active `WORKING-` file is
[`paths/WORKING-helo-direct-single-recipient-v2.md`](paths/WORKING-helo-direct-single-recipient-v2.md)
— Ivan is walking through it; nothing is reconciled to it until the prefix drops. The three
`EXPLORE-` files are reasoning records whose headers state what each settled and what stays open;
during any open exploration, cite the EXPLORE file rather than `event-model.md` for what it
governs.

`research/archive/` mirrors the primary sources. **Read `archive/NOTICE.md` before redistributing
anything** — three of five mirrored repos carry no license.

---

## 4. Open items

### Hotspots

**The live ledger for everything the v2 walk touches is that walk's own *Hotspots* section** —
the walk is the ledger only for hotspots it meets; the rest live in `event-model.md` → *Hotspots*.
One line each:

| | State |
|---|---|
| **H1** | ✅ answered by the v2 walk — `DataPhaseEntered` has two consumers on the page. Formal closure in the model is pending (rule 9) |
| **H2** | are protocol errors events? **Untouched by the v2 walk** (no error branch taken). Statement in `event-model.md`; the corpus answer is a size test, not a kind test — `event-modeling-research:research/TALKS-FINDINGS.md` |
| **H4** | stream boundaries — a store-level ruling. The walk passes four findings upward and leaves three sub-rulings pending; D.2 is the designated test for the transaction question |
| **H6** | is `Received:`'s `BY` from config or `local_address`? Bites at the walk's step 13; `local_address` keeps exactly one unconsumed value |
| **H7** | does FnEmail ever refuse mail entirely (RFC 7504 `521`)? Product question |
| **H8** | does `SessionClosed` survive a forward completeness pass? Proposed in `model-altitude.md` Q6, never registered in the model — register it or close it |
| *(new, unnumbered)* | **the refused greeting still has a session** — a `554` at connection opening must still wait for `QUIT` (§3.1), contradicting the model's `AcceptConnection` contract *reply 554, no event*. Rule 4 correction pending in the model; numbering belongs to the model |

*(H3 and H5 are resolved — see `DECISIONS.md` and `event-model.md`. The spoken corpus has nothing
on H4 — swept 2026-08-06, 982,637 words, zero hits — so it has to be reasoned out, not looked up.)*

### Model work pending

The reconciliation pass runs **when the v2 walk sheds its `WORKING-` prefix**, not before
(AGENTS.md rule 13). Queued so far, each recorded in the walk or a commit:

- `AcceptConnection`'s contract *reply 554, no event* is contradicted by RFC 5321 §3.1 — rule 4
  correction pending (the walk's unnumbered hotspot).
- `SessionState` was never consulted by the walk; step 4 became `SessionReady{server_domain}`
  (commit cf3227d). The model still defines `SessionState`.
- The metadata table carries three correlation schemes — `session_id`, `correlation_id`,
  `queue_id` — against its own one-mechanism warning; `session_id` and `correlation_id` died on
  the path (commits 0e31109, and the walk's *The layering* block). Flagged upward, unresolved.
- `ClientIdentified.protocol` fell to the constant test; `WITH` joined the renderer's constants
  (commit 2673b1b). The model still carries the field.
- The six path-defined views and two seeded events await model-side homes; H1's formal closure
  rides along.
- Pending boolean rulings by the `accepting` precedent: `DataPrompt.awaiting_content`,
  `MessageQueued.accepted` (the walk's step 2 note).
- The `ServiceConfigured` origin decision — two candidate shapes recorded, neither chosen
  (commit 6ed62cf).
- The method repo's three-fates document says `TransactionState`'s empty-dataset question is
  *"parked with the walk's step 6"*, but the walk does not yet record it there — reconcile when
  the walk-through reaches step 6, and make the note this repo's first method-repo citation.

### Other open work

- **Third scenario — D.2, aborted transaction.** `Reset` is the only slice no path has touched;
  all three walked paths are RFC-derived and none aborts. Also the test for H4's
  transaction-as-stream-or-phase question.
- **Store-first or validate-first?** The corpus contradicts itself; this model takes
  validate-first (`ClientIdentified` on success, `501`/`500` produce no event), which was assumed
  rather than decided. Close kin to H2. `event-modeling-research:research/TALKS-FINDINGS-CORPUS.md` §3.
- **The author's disagreements with Adam's method** — mentioned but never captured. The likely
  home is now a departures register in the method repo (its rule already requires labeling
  departures; a register document does not exist yet — see the follow-ups below).
- **Vocabulary residue** — the DDD-avoidance premise was refuted 2026-08-06 and *context* keeps
  its name; what remains open is only whether an EM-native name would be *clearer* on its own
  merits. `event-modeling-research:research/model-altitude-theory.md` §2.0d.

### Cross-repo follow-ups

**EventModeling:** when the `WORKING-` prefix drops, three deep links there need the new filename
(`layering.md`, `path-and-step-form.md`, `state-view-todo-list-decision-model.md`); create the
departures register and add the minimal-`Given` departure (`STEP-FORM.md` records it here); the
three-fates step-6 mismatch above. Flow candidates when ripe: the cadence verdict, the
GWT-vs-timeline analysis, the view-`When` corpus contradiction, `diagrams/README.md`'s generic
halves, `DECISIONS.md`'s held method rows, the prefix convention, the rendering experiments (with
`rendering.md`'s evidence pointer updated in the same move).

**event-modeling-research:** `UPSTREAM-DEFECTS.md` still cites the deleted
`EXPERIMENT-block-labels.md` under the pre-rename repo name — repoint to the commit-pinned
permalink; `PR-HANDOFF-eventmodelers-kit.md` is a verified, ready, never-submitted upstream fix —
**re-verify before submitting**, it may already be fixed upstream; D3 (is `MD_STR` missing from
Mermaid's `block` grammar deliberately?) still needs `block.jison` read; `app.eventmodelers.ai`
remains unexplored; the 2018 Medium article still has no URL. Two connection quirks worth keeping:
`medium.com` returns 403 to curl (Cloudflare, not an egress block), and the `eventmodelers.ai`
apex fails TLS — use `www.`.

---

## 5. State

- **Model** — v0.3, 12 slices, contracts on each. Deliberately lags the v2 walk; the pending
  reconciliations are listed above and in the model's banner
- **Paths** — three complete walks (`helo-multi-recipient`, `helo-single-recipient`,
  `helo-direct-single-recipient`), covering 11 of 12 slices; the v2 re-walk of the direct path is
  the active `WORKING-` file — all 16 steps drafted, in-file ruling notes through step 4, and it
  defines 6 views and 2 seeded events beyond the model
- **Explorations** — three `EXPLORE-` reasoning records; headers state what each settled
- **Diagrams** — markdown tables rendering v0.3; rebuild queued behind the reconciliation pass
- **RFCs** — 5321 and 7504 under [`rfc/`](rfc/)
- **Code** — none yet, by design
- **Method repo** — seeded 2026-08-08 from these walk-throughs: layering, path/step form, the
  three-fates split, rendering; grew the same day by the parked extensions and the generic
  altitude sections, moved from this repo on Ivan's ruling
- **Research repo** — method reference, 20 corrections, 152 sources, the mirrored corpus, the
  extracted book (57 chapters), and 982,637 words of verified transcript across podcasts and talks
