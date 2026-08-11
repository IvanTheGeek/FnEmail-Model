# FnEmail-Model

**FnEmail** — an email system being designed with **Event Modeling** (Adam Dymitruk's method)
before any code is written. This repository contains **no code**: it is the model, the worked
examples, and the rules deciding what belongs in the model.

Current scope: **inbound SMTP, `HELO` only** — no ESMTP, no relay, no outbound. Deliberately
narrow, so the method is exercised properly before the surface grows.

The author is building this for himself and intends to release it freely — and the project serves
a second purpose that explains the first. FnEmail is simultaneously the first worked example for a
modeling tool (see the [`EventModeling`](https://github.com/IvanTheGeek/EventModeling) repo), and
the markdown here is a **prototype of generated output**. Two operative consequences: a form that
is merely tolerable to write by hand but **wrong to read is the wrong answer**, because the tedium
disappears when a tool emits it; and where a relationship cannot be represented in markdown,
**record the relationship and accept the flat rendering** — never abandon the relationship.
⚠️ None of this is license to over-build: the standing instruction (2026-08-07) is to get the
basics working first and refine placement later.

**Paths are the source; slices are derived.** The model document looks like the definition and the
walked paths look like instances of it; the flow of authority is the opposite. Held as a working
hypothesis, not a settled ruling — the statement is the method repo's
[`layering.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/layering.md), whose
status banner records the demotion from "ruled"; the register row in
[`docs/DECISIONS.md`](docs/DECISIONS.md) carries the standing change. It is a change of standing,
not a correction — the position was accepted and is now held provisionally, which is a different
thing from having been wrong (AGENTS.md rule 14).

## What FnEmail is

*Not normative.* This section is orientation for a newcomer. It is not an input to any test,
nothing in the model reads it, and the owner changes it without a ruling. It lives in the README
because the README already answers *what is this repository*, and *what is the thing being
modeled* is that same question one step out. What the model claims conformance **against** is a
separate and checkable thing — [`docs/event-model.md`](docs/event-model.md) →
*The normative set*. A sentence of prose is not a conformance claim, which is why this one is not
asked to be.

FnEmail is an open source, AGPL email server that will be RFC compliant. That includes SMTP for
sending and receiving; local users, across multiple domains and multiple accounts; relays,
retries and bounces; and IMAP. It will **probably** end up having its own web-based email client,
and spam filtering and prevention will **probably** work their way in as well. Stated by the
owner, 2026-08-09 — and both *probably*s are his: those two are likely, not committed.

## Five repositories

The repos mirror the modeling layers. Canonical ruling record:
[`docs/DECISIONS.md`](docs/DECISIONS.md).

| Repo | Layer | Holds |
|---|---|---|
| [`EventModeling`](https://github.com/IvanTheGeek/EventModeling) (public) | method | generic modeling material — the canon primer, layering, path/step form, the three-fates split, rendering, altitude, the parked extensions; the modeling tool it works toward will live in repos of its own |
| `FnEmail-Model` — **this repo** (renamed from `FnEmail` 2026-08-08, old URLs redirect) | model | the SMTP model, walks, RFCs, SMTP-specific references. Cites the method repo |
| `FnEmail` (future) | code | what the model generates — not yet created |
| [`ModelingTool-Model`](https://github.com/IvanTheGeek/ModelingTool-Model) (public) | the tool's model | ideas for the modeling tool now, its model later — pre-model, and nothing in it is a decision |
| `event-modeling-research` (private) | sources | the mirrored corpus — cite, never reproduce |

Method-generic insight flows to `EventModeling` with FnEmail demoted to example; SMTP specifics
stay here ([`AGENTS.md`](AGENTS.md) rule 11). `ModelingTool-Model` models a tool that *implements*
the method, so it is a consumer of `EventModeling` rather than a second copy — insight found while
modeling the tool still flows up.

## Get set up

Clone the repos you need. *(Verified working 2026-08-08.)*

```bash
# 1. The model — this repo. Work is on main; an explore/ branch may be
#    checked out while an exploration runs (AGENTS.md rules 13 and 15).
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
git -C /home/ivan/FnEmail-Model branch --show-current   # → main, or an explore/ branch
ls /home/ivan/FnEmail-Model/docs                        # → DECISIONS.md, event-model.md, paths/, …
ls /home/ivan/EventModeling/docs                        # → layering.md, path-and-step-form.md, …
ls ~/event-modeling-research                            # → research/, extracted/, 6 root files
```

⚠️ **Never copy anything from `~/event-modeling-research` into the public repos** — AGENTS.md
rule 7. Read that repo's `extracted/README.md` and `research/archive/NOTICE.md` before quoting
from the extracted book or the machine transcripts.

## Read in this order

Start with the method layer, then this repo, then the research repo as needed.

| File | Repo | Why |
|---|---|---|
| `README.md`, then `docs/` | EventModeling | **The method layer.** The canon primer, layering, path/step form, the three-fates split, rendering, altitude, the parked extensions — the canonical statements this repo cites |
| [`AGENTS.md`](AGENTS.md) | this repo | The working rules — fifteen, each learned the hard way |
| [`docs/DECISIONS.md`](docs/DECISIONS.md) | this repo | What is settled, and must not be re-litigated (AGENTS.md rule 9) |
| [`docs/event-model.md`](docs/event-model.md) | this repo | The model — v0.3, 12 slices, **derived**: reconciled 2026-08-08 to the active walk on the owner's waiver of the rule-13 hold; its banner lists what stays open |
| [`docs/model-altitude.md`](docs/model-altitude.md) | this repo | **A signpost, not a ruleset** — read it to resolve a reference you met somewhere else. Dated notices say where the altitude material went: the gate sequence and the four tiers unadopted 2026-08-10 and parked in the method repo, the nine classified events retired to git, and **there is no charter** — the normative set in `event-model.md` and the product statement above are what replaced it. Old filename and section numbers kept, so the references in the walks still land |
| [`docs/smtp-path-vs-mailbox.md`](docs/smtp-path-vs-mailbox.md) | this repo | Path versus mailbox — the SMTP reference distinction the walks lean on |
| [`docs/paths/`](docs/paths/) | this repo | The walked paths. `STEP-FORM.md` carries what SMTP adds to the step form; the generic form is the method repo's |
| [`docs/diagrams/README.md`](docs/diagrams/README.md) | this repo | The model as tables — renders on GitHub *and* the Claude apps |
| `research/METHOD-REFERENCE.md` | research | The method as its authors define it, contradictions named rather than smoothed over |
| `research/CORRECTIONS-v0.1.md` | research | 20 ways the first draft misapplied the method |

**The working stance** (AGENTS.md rule 13): the active `WORKING-` file is
[`docs/paths/WORKING-helo-direct-single-recipient-v2.md`](docs/paths/WORKING-helo-direct-single-recipient-v2.md)
— under walk-through review; nothing is reconciled to it until the prefix drops. The five
`EXPLORE-` files are reasoning records whose headers state what each settled and what stays open;
the active exploration is
[`docs/paths/EXPLORE-declaration-vs-status.md`](docs/paths/EXPLORE-declaration-vs-status.md) —
cite it, not `event-model.md`, for anything it governs (AGENTS.md rule 13). Four sit in
`docs/paths/` because they came out of a walk; the fifth,
[`docs/EXPLORE-view-semantics.md`](docs/EXPLORE-view-semantics.md), sits a level up because it is
a **corpus survey** rather than a walk product — what the sources say a 🟩 View *is*. It governs
nothing and rules nothing; it supplies the citation trail the still-open view questions were
missing.

## Where status lives

Status is tracked at its sources, deliberately — a parallel ledger drifts (this README absorbed
`docs/HANDOFF.md` on 2026-08-08 for exactly that reason; its history is in git):

- **Working rules** — [`AGENTS.md`](AGENTS.md). **Settled decisions** —
  [`docs/DECISIONS.md`](docs/DECISIONS.md).
- **What stays open after the model's reconciliation** — the ⚠️ banner atop
  [`docs/event-model.md`](docs/event-model.md).
- **Hotspots** — [`docs/event-model.md`](docs/event-model.md) → *Hotspots*; the active walk's own
  *Hotspots* section is the live ledger for the ones it touches.
- **Work only other repos can do** — [`docs/FOLLOW-UPS.md`](docs/FOLLOW-UPS.md).
- **Everything else, including why any current statement is what it is** — the commit messages,
  long on purpose (AGENTS.md rules 6 and 14): documents carry the present, git history carries
  the why, and ruling-bearing commits are titled for verbatim retrieval with
  `git log -F --grep='<title>'`.

## License

**Code is AGPL-3.0, documents are CC BY-SA 4.0** — the family ruling, registered in
[`docs/DECISIONS.md`](docs/DECISIONS.md); the ruling commit carries the reasoning. Applied here:

- **The documents — everything this repository currently holds** — are licensed
  [CC BY-SA 4.0](LICENSE).
- **Code, if this repository ever comes to hold any** (checks, scripts), is licensed
  [AGPL-3.0](LICENSE-code) — the license the future modeling tool and the future `FnEmail` code
  repo will take.
- **Not relicensed:** third-party material included here retains its original license and is not
  covered by either grant — currently the IETF RFC texts in [`docs/rfc/`](docs/rfc/), reproduced
  verbatim under the IETF Trust's own terms (BCP 78). Brief quotations from the method corpus
  likewise remain their authors' — AGENTS.md rules 2 and 7. Beyond brief quotation, third-party
  material enters only when its own terms permit redistribution, always attributed, its terms
  named where it sits.
