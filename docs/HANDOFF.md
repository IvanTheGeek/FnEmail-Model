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
| `docs/paths/` | FnEmail | Two worked paths with real data |
| `docs/diagrams/README.md` | FnEmail | Mermaid renderings |
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
| **No arrows** — meaning is row position and left-to-right time | `diagrams/README.md` |
| **US spelling** throughout (color, modeling, centered, neighbor) | corrected twice; keep it |
| **One event per command** is a design target, stronger than the corpus's "keep an eye on it" | `diagrams/README.md` §1 |
| **Two slice types, not four patterns** — Automation and Translation are compositions | research: `METHOD-REFERENCE.md` |
| **Context and concern** are the only independent axes; altitude falls out of the charter | research: `model-altitude-theory.md` §2.0b–c |
| **H3 resolved** — Directory is a separate context, joined by orthodox Translation | `event-model.md` |
| **Golden path deliberately unassigned** — *happy* is descriptive, *golden* is a designation | `paths/helo-multi-recipient.md` |
| **Mermaid label vocabulary** — settled empirically | `diagrams/EXPERIMENT-block-labels.md` |
| **The method research is a separate repo** — FnEmail uses Event Modeling, is not about it | §1 above |

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
| **H5** | Is `AcceptConnection` a domain fact or infrastructure noise? |
| **H6** | Is `Received:`'s `BY` from config or `local_address`? Decides whether `local_address` is an orphan |
| **H7** | Does FnEmail ever refuse mail entirely (RFC 7504 `521`)? A product question |

*(H3 is resolved.)*

> **H4 gets no help from the spoken corpus.** Swept 2026-08-06 across both bodies of transcript —
> 46 podcast episodes and 35 recorded talks, **982,637 words**. `stream boundary` and `stream
> design` return **zero hits in either**. So does `SMTP`, in the talks. Two independent corpora,
> nothing. That avenue is closed; H4 has to be reasoned out rather than looked up.

### Other pending work

- **Third path — D.2, aborted transaction.** `Reset` (slice 10) is the only slice no path has
  touched. Both existing paths are RFC-derived; D.2 covers the gap.
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

Spoken corpus now **982,637 words**. Findings in research: `model-altitude-theory.md` §2.0d and
`METHOD-REFERENCE.md`.

**What has not:**

- ~150 of the bibliography's *text* sources still carry search-summary confidence and have never
  been read directly — InfoQ, CACM, Hanselminutes, .NET Rocks, Loosely Coupled, Linux.com,
  DDD TW 2021, the Vancouver Tech episodes. The block is lifted; the reading is outstanding.
- The 452k-word talks corpus is **retrieved but barely read.** It has been term-swept, not studied.
  A 3.7-hour workshop recording and a 2.5-hour Vernon conversation are sitting there unexamined.
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
8. **Nothing third-party enters this repo.** Cite it, quote it briefly with attribution, link it —
   but the copy lives in the research repo. This is what keeps FnEmail releasable.

---

## 8. State

- **Model** — v0.3, 12 slices, contracts on each, 6 open hotspots
- **Paths** — 2 walked, 11 of 12 slices covered
- **Diagrams** — rebuilt on the settled mermaid vocabulary
- **RFCs** — 5321 and 7504 under `docs/rfc/`
- **Code** — none yet, by design
- **Research** — moved out 2026-08-06 to `event-modeling-research`: method reference, 20
  corrections, 152 sources, the mirrored corpus, the extracted book, and the podcast transcripts
