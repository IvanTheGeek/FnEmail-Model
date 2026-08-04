# Archive Provenance and Licensing

Everything under `docs/research/archive/` is **third-party material**, mirrored for reference.
Nothing here is FnEmail's own work. This file records where each item came from and under what
terms — read it before redistributing anything in this directory.

Retrieved **2026-08-04**.

## How this became possible

`ACCESS-NOTES.md` records that this environment's egress policy blocks `eventmodeling.org` and
every other primary-source host. That is still true for the *website*. What changed is the
discovery that the site's content is maintained as **Markdown in a public GitHub repository** —
and GitHub is reachable. The canonical text was therefore retrieved from source rather than
from the web.

The same applies to the method's machine-readable form on npm. Between GitHub and the npm
registry, most of the corpus turned out to be reachable after all.

## Inventory

| Directory | Source | License | Notes |
|---|---|---|---|
| `eventmodelers-kits/` | npm `@eventmodelers/*` | **MIT** (Martin Dilger) | Method as Claude Code skills |
| `eventmodeling-toolkit/` | `github.com/event-modeling/eventmodeling-toolkit` | **MIT** (© 2023 Event Modeling) | EMSL — the specification language |
| `eventmodeling-org/` | `github.com/event-modeling/eventmodeling.org` | **no license file** ⚠️ | Canonical site content |
| `event-modeling-spec/` | `github.com/dilgerma/event-modeling-spec` | **no license file** ⚠️ | JSON Schema for event models |
| `awesome-eventmodeling/` | `github.com/MateuszNaKodach/awesome-eventmodeling` | **no license file** ⚠️ | Curated link list |

### ⚠️ Unlicensed items

Three of the five repositories carry **no LICENSE file**. Publishing a repository publicly is
not itself a grant of rights, so by default these remain all-rights-reserved by their authors.

They are mirrored here as a **reference copy**, with attribution intact and content unmodified,
because the material documents a method this project is adopting and the upstream host is
unreachable from the build environment. That is a defensible reason to keep a local copy; it is
**not** permission to redistribute.

If FnEmail is ever made public, revisit this. The safe options are to drop these directories and
keep only `BIBLIOGRAPHY.md`'s links, or to ask the authors for an explicit license. The MIT-licensed
items (`eventmodelers-kits/`, `eventmodeling-toolkit/`) can stay either way, provided their
copyright notices travel with them.

## What was deliberately excluded

- **Adam Dymitruk — *Event Modeling*** and **Martin Dilger — *Understanding Eventsourcing***.
  Commercial books. Cited and synthesized throughout `docs/research/`, never reproduced.
- **`event-modeling/podcast`** — 125 MB, almost entirely non-text assets, and it contains no
  transcripts. Only `eventmodel.drawio` was taken, as a worked example of a real event model.
- **Build-kit JavaScript runtimes** — re-fetchable from npm, not method documentation.

## Why `eventmodeling-org/` matters most

`posts/what-is-event-modeling/index.md` (~25 KB) is Adam Dymitruk's foundational article and the
single most authoritative statement of the method. `posts/specifying-complex-views/index.md` is
an **unpublished draft** present in the repo but not on the live site.

Note that `posts/event-modeling-cheatsheet/index.md` is authored by `sbortz`, not Dymitruk, and
its vocabulary differs from both Dymitruk's article and Dilger's 2026 cheat sheet. Those three
framings are reconciled in `../METHOD-REFERENCE.md` rather than silently merged.
