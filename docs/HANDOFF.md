# Session Handoff

Everything a fresh session needs to pick this up. Written 2026-08-05 from a Claude Code container
session; revised 2026-08-06 on a local devbox, when the Event Modeling research was split into a
second repository.

---

## 1. What this project is

**FnEmail** — an email system being designed with **Event Modeling** (Adam Dymitruk's method)
before any code is written. The repo contains **no code**: it is the model, the worked examples,
and the rules deciding what belongs in the model.

Current scope: **inbound SMTP, `HELO` only** (no ESMTP, no relay, no outbound). Deliberately
narrow, so the method is exercised properly before the surface grows.

The author is building this for himself and intends to release it freely.

### It serves two purposes, and the second one explains the first

**FnEmail is a real product, and simultaneously the test bed for a modeling tool the author intends
to build.** Both are real; neither is a pretext. That is why the documents get worked to a degree a
single SMTP server would never justify — the markdown is a **prototype of generated output**.

**The long-run intent is that the markdown model and its diagrams are *generated*, not
handcrafted.** Everything being settled by hand right now — the step form, the styling convention,
where a specification attaches — is deciding **what the generator should emit**. A form that is
merely tolerable to write by hand but wrong to read is the wrong answer; a form that is tedious by
hand but right to read is the right one, because the tedium disappears when a tool emits it.

The tool is also expected to be **more advanced in how it relates elements to each other** than
markdown can express. So where a relationship cannot be represented here, the correct response is to
record the relationship and accept the flat rendering — not to abandon the relationship.

⚠️ **Do not read this as license to over-build.** The author's instruction on 2026-08-07 was to get
the basics working first and refine placement later. The two purposes justify care about *form*;
they do not justify inventing structure ahead of need.

### Paths are the source; slices are derived

Recorded 2026-08-07, and it inverts the obvious reading of this repository.

`event-model.md` looks like the definition and the paths look like instances of it. **The intended
flow of authority is the opposite.** A step in a path carries only what is at hand for that walk —
its own concrete data, its own examples, its own specifications. Those per-step sets are then
**combined and deduplicated across every walk**, and *that* union is the slice: the slice holds the
composition of every step that traversed it, for which real data exists.

Two consequences worth stating plainly:

- **A slice's specification set is a result, not an input.** It cannot be complete until enough
  paths have been walked, which is why `Reset` remains uncovered and why that matters — an untouched
  slice has no evidence behind it at all.
- **This is rule 8 as an architecture** rather than as advice. Walking with real data is not merely
  how defects get found; it is how the model gets *populated*.

### A walk is not a workflow

They look alike and are different concepts, and conflating them would be easy.

| | |
|:--|:--|
| **Walk** (a path document) | one concrete traversal, in time order, carrying real values |
| **Workflow** (the bars across a model) | a named region of the model, grouping slices structurally |

**A single walk crosses several workflows**, and a workflow contains slices a given walk never
touches. The ten steps of `helo-direct-single-recipient.md` would sit across roughly four workflows;
`Reset` sits inside one that no walk has entered. Held as *probably* distinct — the author flagged
the possibility that they converge, and nothing has tested it.

### Aggregates and DCB are out of scope

Recorded 2026-08-07 so it is not re-raised. **`Aggregate` is a DDD term with a real meaning in code**,
and *Dynamic Consistency Boundary* is the newer replacement for it. Both appear in the tooling —
EMSL lists `Aggregate` in its `Kind` enum, Dilger's toolkit README calls them *"domain entities
enforcing business rules"*, and his channel carries *Aggregateless Event Sourcing with DCB*,
*Aggregates in Miro Eventmodeling definieren* and *What is an Aggregate in Software?*.

**None of it is an Event Modeling question.** The position both authors take is that whatever an
aggregate is for, it falls out naturally from modeling the way the method already prescribes. The
topic arrives from **technical DDD people**, not from the method, and it is not a difference between
the two men worth tracking.

⚠️ **Do not add it to the Adam-vs-Martin table** in `research/GWT-FINDINGS.md` §6. It looks like a
divergence — two tooling artifacts treat the aggregate as a primitive while the podcast opens with
two episodes titled *Destroying the Aggregate* — and it is not one.

### Two repositories, and why

**FnEmail uses Event Modeling but is not about it.** As of 2026-08-06 the method research lives in
a separate private repo, and this one keeps only email and RFC material.

| | `FnEmail` (public-bound) | `event-modeling-research` (private) |
|---|---|---|
| Holds | the SMTP model, worked paths, diagrams, RFCs, the rules deciding model contents | the method as its authors define it, the source bibliography, the mirrored corpus, extracted book text and podcast transcripts |
| Licensing | everything is the author's own work, plus IETF RFCs, which are freely redistributable | third-party material, most of it unlicensed or commercial — **never redistribute** |

That split is what makes FnEmail safe to release. It was **not** the reason for it — the reason is
scope — but it settles the licensing question that `NOTICE.md` had left open.

---

## 2. Get set up

**Nothing is on disk yet.** Clone both repos.

```bash
# 1. The project. All work is on this branch — main is empty.
cd /home/ivan/FnEmail            # existing empty directory
git clone -b claude/github-access-v08b5c git@github.com:IvanTheGeek/FnEmail.git .

# 2. The research — PRIVATE, needs auth
git clone git@github.com:IvanTheGeek/event-modeling-research.git ~/event-modeling-research
```

Both use SSH. If you get a 404 on the private repo that means **not authenticated**, not *does not
exist* — it is the confusing failure mode. Generate a key with `ssh-keygen -t ed25519`, add the
`.pub` half at <https://github.com/settings/keys>, and check with `ssh -T git@github.com`.

Verify:

```bash
git -C /home/ivan/FnEmail branch --show-current      # → claude/github-access-v08b5c
ls /home/ivan/FnEmail/docs                           # → event-model.md, paths/, diagrams/, rfc/
ls ~/event-modeling-research                         # → research/, extracted/, 5 root files
```

⚠️ **Never copy anything from `~/event-modeling-research` into FnEmail.** It holds two commercial
works, a mirrored corpus that is mostly unlicensed, and machine transcripts of copyrighted speech.
FnEmail cites and synthesizes; it never reproduces. See that repo's `extracted/README.md` and
`research/archive/NOTICE.md`.

### The book text is already extracted

The per-chapter extraction that a previous session lost with its container is now durable, at
`~/event-modeling-research/extracted/book/` — 57 chapters, 86,450 words, named so the research
documents' citations resolve (`0152-domain-driven-design.txt` ll. 175–178, and so on). The
reproduction recipe and its two Debian gotchas are in that repo's `extracted/README.md`.

Podcast transcripts are alongside it: 46 episodes, 530,523 words. **Read the three caveats before
quoting from them** — no speaker identity, machine transcription, and the rights position.

---

## 3. Read in this order

Start in the research repo, then come back here.

| File | Repo | Why |
|---|---|---|
| `research/METHOD-REFERENCE.md` | research | **Start here.** The method as its authors define it, with the places sources contradict each other named rather than smoothed over |
| `docs/event-model.md` | FnEmail | The model — v0.3, 12 slices, contracts, 7 hotspots |
| `docs/model-altitude.md` | FnEmail | What belongs in this model at all — charter, gate sequence, four tiers |
| `research/model-altitude-theory.md` | research | The corpus theory behind it, and the vocabulary investigation |
| `docs/paths/` | FnEmail | Three worked paths with real data. `STEP-FORM.md` defines the step convention |
| `docs/diagrams/README.md` | FnEmail | The model as tables — renders on GitHub *and* Android |
| `docs/event-model-extensions.md` | FnEmail | The author's extension ideas — **parked, deliberately not applied** |
| `research/CORRECTIONS-v0.1.md` | research | 20 ways the first draft misapplied the method |

`research/archive/` mirrors the primary sources. **Read `archive/NOTICE.md` before redistributing
anything** — three of five mirrored repos carry no license.

---

## 4. Decisions already settled — do not re-litigate

| Decision | Where |
|---|---|
| **Command Slice / View Slice** — not "write column / read column". *Slice*, not column | `event-model.md` conventions |
| **Events are orange**, Command blue, Read Model green, Screen white, external yellow, hotspot red | research: `METHOD-REFERENCE.md` |
| **No arrows** — meaning is row position and left-to-right time | `diagrams/README.md` — but see the caveat below |
| **US spelling** throughout (color, modeling, centered, neighbor) | corrected twice; keep it |
| **One event per command** is a design target, stronger than the corpus's "keep an eye on it" | `diagrams/README.md` §1 |
| **Two slice types, not four patterns** — Automation and Translation are compositions | research: `METHOD-REFERENCE.md` |
| **Context and concern** are the only independent axes; altitude falls out of the charter | research: `model-altitude-theory.md` §2.0b–c |
| **H3 resolved** — Directory is a separate context, joined by orthodox Translation | `event-model.md` |
| **Golden path deliberately unassigned** — *happy* is descriptive, *golden* is a designation | `paths/helo-multi-recipient.md` |
| ~~**Mermaid label vocabulary**~~ — **superseded 2026-08-06**, diagrams are now tables | `diagrams/README.md` → *Why tables* |
| **The method research is a separate repo** — FnEmail uses Event Modeling, is not about it | §1 above |
| **Workflow, never *chapter*** — for a named group of slices. Dymitruk's word 241× to 10×; *chapter* is Dilger's | research: `open-spaces-comocamp/_STRUCTURE.md` |
| **Workflow nesting has no fixed depth** — as many levels as make sense | same |
| **Learn the method before extending it** — rule 10's reason, stated | §7 below |
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
> Detail in research: `TALKS-FINDINGS-CORPUS.md` §1.
>
> ⚠️ **And hotspots are Dilger's device, not Dymitruk's.** He removed them from what he took over
> from Event Storming — *"there is no there's no hotspots"* — and refuses to standardise any marker
> for open questions. This project's seven hotspots are legitimate, since Dilger is canonical, but
> §7 rule 4 presents "unknowns become hotspots" as method-neutral and it is not. `TALKS-FINDINGS-CORPUS.md` §2.

### The mermaid vocabulary in one block

```
block
  columns 2
  a["<b>Title</b>\ndetail line"]  b["<b>Other</b>\ndetail"]
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:75px
  class a,b event
```

- `block`, **not** `block-beta` (the suffix is gone as of 11.16.1, though the old form still parses)
- **Markdown never works** in block labels — no spelling of it. HTML only
- `<b>` for partial bold; `\n` **or** `<br/>` for breaks; `&lt;` `&gt;` for angle brackets
- **Boxes never wrap** — explicit breaks are mandatory
- `classDef` carries color **and** `text-align:left` **and** `height` — no per-block `style` needed
- **Vertical alignment has no CSS route.** Text always centers vertically; only trailing `<br/>`
  padding moves it

---

## 5. Open items

### Hotspots in the model

| | Question |
|---|---|
| **H1** | Does `DataPhaseEntered` earn its place? Only candidate consumer is the transcript rendering `354` |
| **H2** | Are protocol errors events? `RecipientRejected` is; a `503` is not |
| **H4** | Stream boundaries — session, transaction, or message? **Most likely to force a restructure** |
| **H5** | ~~Domain fact or infrastructure noise?~~ **Classification settled 2026-08-06** — *infrastructure in shape, admitted by consumer*, and **Product** tier, not domain: §4.4's address literal is SHOULD, not MUST. What remains is the product decision, which is H7's decision. See `event-model.md` → H5 |
| **H6** | Is `Received:`'s `BY` from config or `local_address`? Decides whether `local_address` is an orphan |
| **H7** | Does FnEmail ever refuse mail entirely (RFC 7504 `521`)? A product question |

*(H3 is resolved.)*

> **Three hotspots gained corpus evidence on 2026-08-06**, from reading the workshop recording and
> the Vernon deep dive in full. Detail and quotations in research: `TALKS-FINDINGS.md`.
>
> - **H2 — are protocol errors events?** The corpus answer is a **size test, not a kind test**. A
>   large divergence gets its own workflow placed to the right with a cross-reference back; a small
>   one stays as a red error scenario on the slice. *"if there's really large workflow changes for
>   the sad path we generally put them all the way to the right and start a new workflow and
>   cross-reference it back to where the bad things happened in the middle."*
> - **H5 — is `AcceptConnection` a domain fact or infrastructure noise?** Neither, and the framing
>   was a false choice. Dymitruk puts infrastructure on the model in **its own lower swimlane,
>   hidden when discussing business**, admitted exactly when *"our business is so tied to a piece of
>   infrastructure working correctly that it makes sense to at least have a place to discuss that."*
>   For a conformant SMTP server, TCP acceptance is that.
> - **H4 — stream boundaries.** No direct answer, but the **reservation pattern** is an argument for
>   the *transaction*: *"the structure is always a two-phase commit type … you always have two events
>   in that scenario … reserve, actually debit, this is your pattern."* SMTP's `RCPT TO` → final
>   `CRLF.CRLF` is that shape, arrived at from the RFC rather than from the method.

> **H4 gets no help from the spoken corpus.** Swept 2026-08-06 across both bodies of transcript —
> 46 podcast episodes and 35 recorded talks, **982,637 words**. `stream boundary` and `stream
> design` return **zero hits in either**. So does `SMTP`, in the talks. Two independent corpora,
> nothing. That avenue is closed; H4 has to be reasoned out rather than looked up.

### Other pending work

- **Third path — D.2, aborted transaction.** `Reset` is the only slice no path has touched. Both existing paths are RFC-derived; D.2 covers the gap.
- **research: `PR-HANDOFF-eventmodelers-kit.md`** — ⏳ a two-line upstream fix to
  `Nebulit-GmbH/Eventmodelers-Build-Kits`, verified and ready, never submitted. Self-contained
  runbook; **re-verify first**, it may already be fixed.
- **D3 in research: `UPSTREAM-DEFECTS.md`** — is `MD_STR` missing from Mermaid's `block` grammar
  deliberately or by omission? Read `block.jison` against the flowchart grammar.
- **Vocabulary question** — ~~"context" is Evans' *bounded context* under a shorter name, and
  Dymitruk deliberately avoids DDD vocabulary. Should our axes get EM-native names?~~
  **Largely answered 2026-08-06** — the premise was false. Dymitruk does not avoid DDD vocabulary;
  he avoids DDD as a *prerequisite*. See research: `model-altitude-theory.md` §2.0d. **Context
  keeps its name.** What remains open is the narrower question the evidence does not touch:
  whether an EM-native name would be *clearer* on its own merits.
- **Store-first or validate-first? A new open decision, 2026-08-06.** The corpus contradicts itself
  on whether an event is written before or after validation. Dymitruk: *"the first thing you do
  after you click a button on a form is just store the damn thing as fast as you can just write to
  disk boom."* Dilger, opposite: *"a successful command execution means there is a new event."*
  **This model takes the second position** — `ClientIdentified` on success, `501`/`500` produce no
  event — which was assumed rather than decided. It is close kin to H2. See research:
  `TALKS-FINDINGS-CORPUS.md` §3.
- **The author's disagreements with Adam's method** — mentioned but never captured. Worth its own
  document; a disagreement is a different category from an extension.
- ~~**Written permission from Dilger.**~~ **Settled 2026-08-06 — not needed.** His verbal grant
  covering the posts at `www.eventmodelers.ai` stands, and the author's rule is **use, do not
  republish**. The material is archived in the private research repo and may be read, quoted with
  attribution, and reasoned from; it is never reproduced wholesale and never enters FnEmail.
  Written permission was considered and consciously not pursued, because nothing derived from it
  will be published in a form requiring one. Recorded in research: `archive/NOTICE.md`.

---

## 6. Egress — resolved, but the walk is not finished

The container this was built in had a **restrictive egress policy**: only `github.com`,
`raw.githubusercontent.com` and package registries. That is history — see research:
`ACCESS-NOTES.md`, which is marked as such.

**A local session has no such policy.** Every previously blocked host was probed on 2026-08-06 and
is reachable. Two quirks: `medium.com` returns 403 to `curl` even with a browser user-agent, which
is Cloudflare bot protection rather than an egress block; and the `eventmodelers.ai` apex fails
TLS, so use `www.`.

**What has been done since:**

- All 77 bibliography URLs probed. **72 return 200.** The five that don't are the known-suspect CACM
  staging host, Medium's Cloudflare, one malformed URL, and `axoniq.io/podcasts/event-modeling`,
  which is genuinely dead (404).
- SE Radio 539 and Semaphore Uncut read in full. They overturned the DDD-vocabulary claim.
- **The podcast corpus** — 46 episodes, 530,523 words.
- **A talks corpus the bibliography did not know existed** — a YouTube sweep found **38 videos
  naming either author, 49.7 hours**, where the bibliography had four. 35 transcribed, **452,114
  words**. Includes a 3.7-hour workshop, a deep dive with **Vaughn Vernon**, three talks at DDD
  meetups, a second YOW! talk, and **SE Radio 720**, which this section previously listed as unread.
- **`www.eventmodelers.ai` archived** — 77 blog posts, 46 podcast pages, 21 modeling topics,
  171,925 words. Dilger's posts, under **use-don't-republish**.
- Dilger's 171 public repos cataloged.

Spoken corpus now **982,637 words** across **33 distinct recordings** (two were the same talk
uploaded twice). **All 33 talks have been read in full** — 687 findings, every quotation verified by
grep against source, 687/687 exact. See research: `TALKS-FINDINGS.md` and `TALKS-FINDINGS-CORPUS.md`.

- **The bibliography's text sources have been verified** — 26 sources, 89 attributed claims: 53
  confirmed, 19 refuted, 6 refuted on wording, 7 not found. The block is lifted *and* the reading is
  now done. Corrections are in research: `BIBLIOGRAPHY.md`.
- **The method's origin story does not hold.** Hacker News has no Event Modeling story in 2018; the
  front-page item is 2019-09-29 and points at the website, not Medium; and the canonical article was
  composed in the git repo across June–July 2019 rather than moved there finished. Verified directly
  against the HN API and the repo history.

**What has not:**

- Four sources remain genuinely **unverifiable** — LinkedIn (blocks fetching) and one paywalled
  page. They keep their caveats rather than losing them.
- `app.eventmodelers.ai` — the platform the archived skills are written against.
- The Mermaid repo, for D3.
- The 2018 Medium article. Still no URL.

research: `BIBLIOGRAPHY.md` has 152 sources with live URLs — it was built as the walk-list for
exactly this.

---

## 7. How this project works

Conventions that matter more than they look:

1. **Primary sources, not recall.** Every method claim is traced to a file. v0.1 was written from
   memory and `CORRECTIONS-v0.1.md` is the 20-item record of how that went.
2. **Record corrections rather than quietly fixing them.** Several documents carry ⚠️ blocks
   saying "an earlier draft claimed X; that was wrong because Y." Keep doing this — the reasoning
   is worth more than a clean surface.
3. **Quotations keep their own words.** The corpus says *state-change*, *write column*, *colour*.
   Never rewrite a quote to match project style — it silently corrupts the sourcing.
4. **Unknowns become hotspots**, on the model, not in an appendix.
5. **Extensions stay parked.** `event-model-extensions.md` is deliberately not applied, so the
   orthodox model stays measurable against it.
6. **Walk paths with real data.** Two payload defects, an orphan field, and H3's resolution all
   came from instantiating the model, not reading it. A clean path confirms; a messy one discovers.
7. **Commit messages carry the reasoning.** They are long here on purpose.
8. **Refer to slices by name, not number.** Slice numbering is an artifact of the current altitude
   and scope — change either and the whole sequence renumbers, silently breaking every
   cross-reference. Position on the timeline already carries order, so the number adds fragility
   without adding meaning. Counts are fine: "twelve slices" is a fact about the model, "the `MailFrom` slice" is
   a handle that will move.
9. **Nothing third-party enters this repo.** Cite it, quote it briefly with attribution, link it —
   but the copy lives in the research repo. This is what keeps FnEmail releasable.

---

## 8. State

- **Model** — v0.3, 12 slices, contracts on each, 6 open hotspots
- **Paths** — 3 walked, 11 of 12 slices covered; `helo-direct-single-recipient.md` is the default illustration
- **Diagrams** — rebuilt on the settled mermaid vocabulary
- **RFCs** — 5321 and 7504 under `docs/rfc/`
- **Code** — none yet, by design
- **Research** — moved out 2026-08-06 to `event-modeling-research`: method reference, 20
  corrections, 152 sources, the mirrored corpus, the extracted book, and the podcast transcripts
