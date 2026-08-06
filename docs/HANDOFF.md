# Session Handoff

Everything a fresh session needs to pick this up. Written 2026-08-05, from a Claude Code container
session, for continuation on a local devbox.

---

## 1. What this project is

**FnEmail** — an email system being designed with **Event Modeling** (Adam Dymitruk's method)
before any code is written. The repo currently contains **no code**: it is the model, the research
behind it, and the worked examples.

Current scope: **inbound SMTP, `HELO` only** (no ESMTP, no relay, no outbound). Deliberately
narrow, so the method is exercised properly before the surface grows.

The author is building this for himself and intends to release it freely.

---

## 2. Get set up

```bash
cd /home/ivan/FnEmail
git fetch origin
git checkout claude/github-access-v08b5c     # ← all work is on this branch, not main
git pull
```

**Reference repo** — private, holds material that must not be committed to FnEmail:

```bash
git clone https://github.com/ivanthegeek/reference ~/reference
```

| File | What it is |
|---|---|
| `eventmodeling-and-eventsourcing-compressed.pdf` | Martin Dilger, *Understanding Eventsourcing*, 650 pp — **commercial, do not commit** |
| `event-modeling-cheat-sheet.pdf` | Dilger's 2026-07 cheat sheet, 2 pp — **do not commit** |
| `RFC 5321_*.mht`, `RFC 7504_*.mht` | MHTML captures of the RFCs |
| `Block Diagram Syntax _ Mermaid.mht` | Mermaid block-diagram docs |

### Extracting the reference material

The book's text was previously extracted per chapter and used heavily. That extraction lived in a
scratchpad that **did not survive** — redo it if needed:

```bash
pip install pymupdf
python3 - <<'PY'
import fitz, os, re
d = fitz.open('/home/ivan/reference/eventmodeling-and-eventsourcing-compressed.pdf')
os.makedirs('/tmp/book', exist_ok=True)
marks = sorted((pg, t) for lvl, t, pg in d.get_toc())
for i, (pg, title) in enumerate(marks):
    end = marks[i+1][0]-1 if i+1 < len(marks) else d.page_count
    slug = re.sub(r'[^a-z0-9]+', '-', title.lower()).strip('-')[:60]
    open(f'/tmp/book/{pg:04d}-{slug}.txt','w').write(
        '\n'.join(d[p-1].get_text() for p in range(pg, end+1)))
print('ok')
PY
```

Diagrams are ~550 px raster — legible, but extract them individually if a figure matters:
`fitz.Pixmap(doc, xref).save(...)`.

MHTML files unpack with Python's `email` module — see the same pattern used for the RFCs.

---

## 3. Read in this order

| File | Why |
|---|---|
| `docs/research/METHOD-REFERENCE.md` | **Start here.** The method as its authors define it, with the places sources contradict each other named rather than smoothed over |
| `docs/event-model.md` | The model — v0.3, 12 slices, contracts, 7 hotspots |
| `docs/model-altitude.md` | What belongs in the model at all — gate sequence, four tiers |
| `docs/paths/` | Two worked paths with real data |
| `docs/diagrams/README.md` | Mermaid renderings |
| `docs/event-model-extensions.md` | The author's extension ideas — **parked, deliberately not applied** |
| `docs/research/CORRECTIONS-v0.1.md` | 20 ways the first draft misapplied the method |

`docs/research/archive/` mirrors the primary sources. **Read `archive/NOTICE.md` before
redistributing anything** — three of five mirrored repos carry no license.

---

## 4. Decisions already settled — do not re-litigate

| Decision | Where |
|---|---|
| **Command Slice / View Slice** — not "write column / read column". *Slice*, not column | `event-model.md` conventions |
| **Events are orange**, Command blue, Read Model green, Screen white, external yellow, hotspot red | `METHOD-REFERENCE.md` |
| **No arrows** — meaning is row position and left-to-right time | `diagrams/README.md` |
| **US spelling** throughout (color, modeling, centered, neighbor) | corrected twice; keep it |
| **One event per command** is a design target, stronger than the corpus's "keep an eye on it" | `diagrams/README.md` §1 |
| **Two slice types, not four patterns** — Automation and Translation are compositions | `METHOD-REFERENCE.md` |
| **Context and concern** are the only independent axes; altitude falls out of the charter | `model-altitude.md` §2.0b–c |
| **H3 resolved** — Directory is a separate context, joined by orthodox Translation | `event-model.md` |
| **Golden path deliberately unassigned** — *happy* is descriptive, *golden* is a designation | `paths/helo-multi-recipient.md` |
| **Mermaid label vocabulary** — settled empirically | `diagrams/EXPERIMENT-block-labels.md` |

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

### Other pending work

- **Third path — D.2, aborted transaction.** `Reset` (slice 10) is the only slice no path has
  touched. Both existing paths are RFC-derived; D.2 covers the gap.
- **`docs/research/PR-HANDOFF-eventmodelers-kit.md`** — ⏳ a two-line upstream fix to
  `Nebulit-GmbH/Eventmodelers-Build-Kits`, verified and ready, never submitted. Self-contained
  runbook; **re-verify first**, it may already be fixed.
- **D3 in `docs/research/UPSTREAM-DEFECTS.md`** — is `MD_STR` missing from Mermaid's `block`
  grammar deliberately or by omission? Read `block.jison` against the flowchart grammar.
- **Vocabulary question** — "context" is Evans' *bounded context* under a shorter name, and
  Dymitruk deliberately avoids DDD vocabulary. Should our axes get EM-native names?
- **The author's disagreements with Adam's method** — mentioned but never captured. Worth its own
  document; a disagreement is a different category from an extension.

---

## 6. What a local session unblocks

The container this was built in had a **restrictive egress policy** — only `github.com`,
`raw.githubusercontent.com` and package registries. `eventmodeling.org`, `rfc-editor.org`,
YouTube, Medium and Leanpub were all blocked. See `docs/research/ACCESS-NOTES.md`.

A local devbox probably has none of that. If so:

- **Talk and podcast transcripts** — never obtained. `podcast.eventmodeling.org`,
  `youtube.com/@eventdrivenpodcast`, SE Radio 539 (Dymitruk) and 720 (Dilger). The one claim in
  `METHOD-REFERENCE.md` most likely to be overturned by new evidence is that Dymitruk avoids
  "domain" — that rests on his **written** work only.
- **`eventmodelers.ai`** and the live cheat sheet.
- **`app.eventmodelers.ai`** — the platform the archived skills are written against.
- **The Mermaid repo** for D3.

`docs/research/BIBLIOGRAPHY.md` has 152 sources with live URLs — it was built as the walk-list for
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

---

## 8. State

- **Model** — v0.3, 12 slices, contracts on each, 6 open hotspots
- **Paths** — 2 walked, 11 of 12 slices covered
- **Research** — method reference, 20 corrections, 152 sources, archived corpus
- **Diagrams** — rebuilt on the settled mermaid vocabulary
- **Code** — none yet, by design
