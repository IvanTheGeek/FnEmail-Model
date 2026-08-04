# eventmodelers Kits — Archived Corpus

Mirror of the canonical Event Modeling method as published in machine-readable form by
**Nebulit / eventmodelers**, authored by **Martin Dilger**.

## Provenance

Retrieved from the npm registry on **2026-08-04**. npm is one of the few hosts reachable
from this environment (see `../../ACCESS-NOTES.md`), which is why this is the one part of
the requested archive that could be mirrored verbatim rather than merely indexed.

| Package | Version | License | Author |
|---|---|---|---|
| `@eventmodelers/agent-modeling-kit` | 0.1.74 | MIT | Martin Dilger |
| `@eventmodelers/build-kit-node` | 0.2.24 | MIT | Martin Dilger |
| `@eventmodelers/build-kit-supabase` | 0.2.27 | MIT | Martin Dilger |
| `@eventmodelers/build-kit-axon` | 0.0.5 | MIT | Martin Dilger |

Reproduce with:

```bash
npm pack @eventmodelers/agent-modeling-kit
npm pack @eventmodelers/build-kit-node
npm pack @eventmodelers/build-kit-supabase
npm pack @eventmodelers/build-kit-axon
```

All four are MIT licensed, which is what makes this mirror permissible. The two commercial
books are **not** archived here — see `../../ACCESS-NOTES.md` for that boundary.

## What was kept

- `agent-modeling-kit/skills/` — the full skill corpus (~650 KB of method prose). This is the
  substantive material: the method written out as executable instructions.
- `agent-modeling-kit/CLAUDE.md`, `README.md` — kit-level framing.
- `build-kit-*/` — **markdown only**. The JavaScript runtime, `package.json`, and platform
  client code were dropped; they are re-fetchable from npm and are not method documentation.

## Why this matters

The `agent-modeling-kit` skill names *are* Adam Dymitruk's seven steps, which makes this the
most precise statement of the method available to us:

| Step | Skill |
|---|---|
| 1. Brainstorming events | `eventmodeling-brainstorming-events` |
| 2. The plot | `eventmodeling-plotting-events` |
| 3. Storyboarding | `eventmodeling-storyboarding-events` |
| 4. Identify inputs | `eventmodeling-identifying-inputs` |
| 5. Identify outputs | `eventmodeling-identifying-outputs` |
| 6. Apply Conway's Law | `eventmodeling-applying-conways-law` |
| 7. Elaborate scenarios | `eventmodeling-elaborating-scenarios` |
| 8. Completeness check | `eventmodeling-checking-completeness` |

Note that the kit numbers the completeness check as **step 8**, while Dilger's 2026-07 cheat
sheet presents a **six-step** workshop that merges "identify inputs" and "identify outputs"
into a single `04 - Input / Output`. Both framings are recorded rather than reconciled.

Supporting skills carry material with no direct equivalent in the seven steps:
`eventmodeling-slicing-event-models`, `eventmodeling-validating-event-models`(+`-checklist`),
`eventmodeling-optimizing-stream-design`, `eventmodeling-translating-external-events`,
`eventmodeling-integrating-legacy-systems`, `eventmodeling-designing-event-models`.

The `build-kit-*` packages are the downstream half — they generate code from a model, one
skill per pattern (`build-state-change`, `build-state-view`, `build-automation`). They are
useful here as evidence of how the four patterns are expected to map onto implementation.

## Caveat

These skills are written against the hosted eventmodelers platform API (`app.eventmodelers.ai`)
and many of them open with `curl` calls to a board. Read them for the **method** — the rules,
checklists, patterns, and anti-patterns — not as runnable procedures. The platform is not
reachable from this environment.
