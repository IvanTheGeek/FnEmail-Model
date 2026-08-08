# Decisions already settled — do not re-litigate

The register that AGENTS.md rule 9 points at. If evidence genuinely overturns a row, that is a ⚠️
correction under rule 4 with the evidence attached — never a quiet edit. Until 2026-08-08 this
register lived inside `HANDOFF.md` §4, where a section number was its only handle; it is a file
now, so the handle cannot be silently renumbered.

---

## The repository architecture — ruled 2026-08-08

**The repositories mirror the modeling layers, and insight flows in one direction.** This is the
canonical record of the ruling (commit fd8b2e4); AGENTS.md rule 11 states it as a working rule,
and `HANDOFF.md` carries the operational map with clone instructions.

| Repo | Layer | Holds |
|---|---|---|
| [`EventModeling`](https://github.com/IvanTheGeek/EventModeling) (public, AGPL) | method | generic modeling material; future home of the modeling tool. Modeled systems appear there only as examples |
| **`FnEmail-Model`** (this repo, public-bound) | model | the SMTP model, walks, RFCs, SMTP-specific references. Cites the method repo |
| `FnEmail` (future — the bare name is reserved for it) | code | what the model generates; not yet created |
| `event-modeling-research` (private) | sources | the third-party corpus — cite, never reproduce (rule 7) |

**Method-generic insight flows to `EventModeling` with FnEmail demoted to worked example; SMTP
specifics stay here and cite the method repo.** The repo was renamed from `FnEmail` to
`FnEmail-Model` the same day — history intact, old URLs redirecting — and its default branch is
now **main**.

The split also settles licensing, the question `NOTICE.md` had left open: everything in the two
public repos is the author's own work plus IETF RFCs, which are freely redistributable; everything
third-party lives in the private repo. That is what keeps the public repos releasable.

*(The 2026-08-06 two-repo split — research out of the model repo — is superseded in count but not
in reasoning; its record survives in git history, in `HANDOFF.md` as of commit 94bed9d.)*

## Model decisions

| Decision | Where |
|---|---|
| **H3 resolved** — Directory is a separate context, joined by orthodox Translation | `event-model.md` → *H3 resolved* |
| **Golden path deliberately unassigned** — *happy* is descriptive, *golden* is a designation | `paths/helo-multi-recipient.md` |
| **One event per command** is a design target, stronger than the corpus's "keep an eye on it" | `diagrams/README.md` §1 |

## Method decisions

The canonical home for method-generic decisions is the `EventModeling` repo. Three of these have
already flowed there and are cited rather than restated:

| Decision | Canonical home |
|---|---|
| **Paths are the source; slices are derived** — and a path is a workflow with the data filled in | `EventModeling/docs/layering.md` |
| **One fold, three consumers** — state view, todo list, or decision model, decided by who consumes the fold | `EventModeling/docs/state-view-todo-list-decision-model.md` |
| **The rendering conventions** — measured against all three renderers | `EventModeling/docs/rendering.md`; the evidence experiments live here, in `docs/diagrams/` |

The rest were settled here before that repo existed and its documents do not yet carry them. They
stay recorded in full until they flow (AGENTS.md rule 11):

| Decision | Where |
|---|---|
| **Command Slice / View Slice** — not "write column / read column". *Slice*, not column | `event-model.md` conventions; `diagrams/README.md` → *Terminology* |
| **Events are orange**, Command blue, Read Model green, Screen white, external yellow, hotspot red — carried in markdown by the emoji chips of AGENTS.md rule 5 | `event-modeling-research:research/METHOD-REFERENCE.md` |
| **No arrows** — meaning is row position and left-to-right time. See the caveat below | `diagrams/README.md` |
| **Two slice types, not four patterns** — Automation and Translation are compositions | `event-modeling-research:research/METHOD-REFERENCE.md` |
| **Context and concern are the only independent axes**; altitude falls out of the charter | `event-modeling-research:research/model-altitude-theory.md` §2.0b–c |
| **Workflow, never *chapter*** — for a named group of slices. Dymitruk's word 241× to 10×; *chapter* is Dilger's | `event-modeling-research:research/archive/open-spaces-comocamp/_STRUCTURE.md` |
| **Workflow nesting has no fixed depth** — as many levels as make sense | same |
| **Aggregates and DCB are out of scope** — a DDD import, not an Event Modeling question | below |

> ⚠️ **The "no arrows" decision needs a caveat, added 2026-08-06.** It stands as a *diagramming*
> choice, but the corpus shows arrows are working notation in 12+ independent talks, and they have a
> job the project was not accounting for.
>
> `METHOD-REFERENCE.md`'s claim that *"arrows are notation, not semantics"* is **confirmed** —
> Dymitruk says *"as the tooling matures those those arrows should just autofill"*, so they are
> derived, not authored. But their purpose is **information-completeness tracing**: *"they're really
> about pointing … for the purpose of information completeness where you do use the arrows to make
> sure that you can follow a piece of information."* Arrow-free diagrams forgo that affordance. That
> is a fair trade — this project does the completeness check in prose — but it **is** a trade.
>
> Three further points the rule was conflating:
> - The one hard rule is **directional**: no arrow points backwards.
> - **Fan-out is not branching.** One event feeding two read models is *"just lines of where the
>   information goes but it's really one story."* "No arrows" and "no branching" are separate claims.
> - Elements with **no inbound arrow are the model's origins** — the first screen, external inputs.
>
> Detail in `event-modeling-research:research/TALKS-FINDINGS-CORPUS.md` §1.
>
> ⚠️ **And hotspots are Dilger's device, not Dymitruk's.** He removed them from what he took over
> from Event Storming — *"there is no there's no hotspots"* — and refuses to standardize any marker
> for open questions. This project's hotspots are legitimate, since Dilger is canonical, and
> AGENTS.md rule 12 carries this caveat for exactly that reason — the practice is method-legitimate
> but not method-neutral. *(The caveat originally counted "seven hotspots" and criticized a
> HANDOFF conventions item that stated the practice without the caveat; on the 2026-08-08 move the
> count was dropped as volatile — H1 is answered, H8 only proposed, and the walk added an
> unnumbered one — and the criticized item became rule 12, which states the caveat itself.)*
> `event-modeling-research:research/TALKS-FINDINGS-CORPUS.md` §2.

### Aggregates and DCB are out of scope

Recorded 2026-08-07 so it is not re-raised. **`Aggregate` is a DDD term with a real meaning in
code**, and *Dynamic Consistency Boundary* is the newer replacement for it. Both appear in the
tooling — EMSL lists `Aggregate` in its `Kind` enum, Dilger's toolkit README calls them *"domain
entities enforcing business rules"*, and his channel carries *Aggregateless Event Sourcing with
DCB*, *Aggregates in Miro Eventmodeling definieren* and *What is an Aggregate in Software?*.

**None of it is an Event Modeling question.** The position both authors take is that whatever an
aggregate is for, it falls out naturally from modeling the way the method already prescribes. The
topic arrives from **technical DDD people**, not from the method, and it is not a difference
between the two men worth tracking.

⚠️ **Do not add it to the Adam-vs-Martin table** in
`event-modeling-research:research/GWT-FINDINGS.md` §6. It looks like a divergence — two tooling
artifacts treat the aggregate as a primitive while the podcast opens with two episodes titled
*Destroying the Aggregate* — and it is not one.

## Superseded

| Was | Replaced by | When |
|---|---|---|
| **Mermaid diagrams and their label vocabulary** | markdown tables | 2026-08-06 — see `diagrams/README.md` → *Why tables*. The upstream Mermaid defect thread (D3) lives in `event-modeling-research:research/UPSTREAM-DEFECTS.md` |

Two rows of the old register are deliberately absent rather than moved: **US spelling** is
AGENTS.md rule 1, and **learn the method before extending it** is stated inside AGENTS.md rule 10.
