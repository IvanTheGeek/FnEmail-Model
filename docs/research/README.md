# Event Modeling Research

Primary-source research behind FnEmail's event model.

## Read in this order

| File | What it is |
|---|---|
| [`METHOD-REFERENCE.md`](METHOD-REFERENCE.md) | **Start here.** The method as its authors define it, with the places the sources contradict each other named rather than smoothed over. |
| [`../event-model.md`](../event-model.md) | The model itself — RFC 5321 inbound, v0.2. |
| [`CORRECTIONS-v0.1.md`](CORRECTIONS-v0.1.md) | 20 corrections from the first draft. Useful as a catalogue of ways to misapply the method. |
| [`BIBLIOGRAPHY.md`](BIBLIOGRAPHY.md) | 152 sources across both authors — articles, talks, podcasts, repos, tools. |
| [`METHOD-REFERENCE-DETAIL.md`](METHOD-REFERENCE-DETAIL.md) | Long-form companion. Two known defects, flagged in its header. |
| [`UPSTREAM-DEFECTS.md`](UPSTREAM-DEFECTS.md) | Confirmed defects in third-party material, with exact line numbers, for reporting upstream. |
| [`ACCESS-NOTES.md`](ACCESS-NOTES.md) | Why this is partly an index rather than a mirror, and how to lift the limit. |
| [`archive/`](archive/) | Mirrored primary sources. **Read [`archive/NOTICE.md`](archive/NOTICE.md) before redistributing anything.** |

## What the archive contains

| | Source | License |
|---|---|---|
| `archive/eventmodeling-org/` | Dymitruk's canonical article + the site's posts, from the public Hugo repo | ⚠️ none |
| `archive/eventmodelers-kits/` | The method as ~650 KB of Claude Code skills | MIT |
| `archive/eventmodeling-toolkit/` | EMSL — the specification language | MIT |
| `archive/event-modeling-spec/` | JSON Schema for event models | ⚠️ none |
| `archive/awesome-eventmodeling/` | Community link list | ⚠️ none |

The two commercial books are cited throughout and reproduced nowhere.

## Findings that mattered most

1. **Two column types, not four patterns.** Every column either writes (command → events) or
   reads (events → read model). Automation and Translation are compositions. This is Dymitruk's
   own framing — *"3 moving pieces and 4 patterns based on **2 ideas**… empowering the user and
   informing the user"* — and he specifies automation as two Given-When-Thens.

2. **"Trigger" resolves systems with no UI.** eventmodeling.org admits *"the route of an http
   endpoint"* as a trigger. An SMTP verb qualifies. No adaptation needed.

3. **Swimlane means two different things**, both Dymitruk's: actor lanes at the top (step 3.1)
   and event-ownership lanes (step 6).

4. **The completeness check is bidirectional** — origin *and* destination. *"All information has
   to have an origin and a destination."*

5. **Read models sit at their consumer**, not next to the events they project.

6. **Colour is settled**: Command always blue, View always green, Event orange (current) or
   yellow (older). One upstream file transposes Event and View — a confirmed defect, logged in
   `UPSTREAM-DEFECTS.md` with line numbers.

7. **Source precedence**: recent Dymitruk/Dilger beats older; both beat third-party material on
   canonical sites — including the eventmodeling.org cheat sheet, which is authored by `sbortz`.

## Method

`WebSearch` for discovery (152 sources), then primary sources read directly — 650 pages of book,
~650 KB of skill definitions, and the canonical article from GitHub. Two multi-agent workflows,
25 agents, ~2.3 M tokens. Nothing in `METHOD-REFERENCE.md` rests on recall.
