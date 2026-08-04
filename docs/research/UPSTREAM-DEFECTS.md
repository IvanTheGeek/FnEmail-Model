# Upstream Defects

Defects found in third-party Event Modeling material while building this model. Recorded so they
can be reported, and so this repo does not silently propagate them.

---

## D1 — Sticky-note colours transposed in `agent-modeling-kit`

**Status:** confirmed, unreported
**Severity:** low mechanically, high pedagogically — it is in the *workshop facilitation* guide,
so it teaches the wrong convention to a room of beginners at the moment they are learning it.

### Location

```
package : @eventmodelers/agent-modeling-kit@0.1.74   (MIT, Martin Dilger)
file    : templates/.claude/skills/eventmodeling-brainstorming-events/
          references/facilitating-event-modeling-workshops.md
section : "### 4. Create Templates"
lines   : 89, 91, 93
```

Local copy: [`archive/eventmodelers-kits/agent-modeling-kit/skills/eventmodeling-brainstorming-events/references/facilitating-event-modeling-workshops.md`](archive/eventmodelers-kits/agent-modeling-kit/skills/eventmodeling-brainstorming-events/references/facilitating-event-modeling-workshops.md)

### Current text

```
Physical sticky notes:
[Green] Event: ________________

[Blue] Command: ________________

[Orange] View: ________________

Virtual shapes:
Same colors with text fields
```

### Defect

**Event and View are transposed.** Should be:

```
[Orange] Event: ________________

[Blue] Command: ________________

[Green] View: ________________
```

### Evidence

The kit contradicts its own author's other published material:

| Source | Command | Event | View / Read Model |
|---|---|---|---|
| **Dilger, Event Modeling Cheat Sheet (2026-07)** | blue | **orange** | **green** |
| **Dilger, *Understanding Eventsourcing*, figures** (e.g. p222, p234) | blue | **orange** | **green** |
| eventmodeling.org cheat sheet (2022) | blue | yellow | **green** |
| Dymitruk, *"Event Modeling: What is it?"* | blue | — | **green** ("green to-do list") |
| **This file** | blue | **green** ❌ | **orange** ❌ |

Every other source places View at green. The cheat sheet and the book — both by the same author as
this kit — place Event at orange. Blue = Command is invariant everywhere and is correct here.

Event is legitimately **yellow in older material and orange in current material**; both readings
have standing. Green is not among them. View being anything other than green has no support in
any source examined.

### Scope

`grep` across all 68 markdown files in the four `@eventmodelers/*` packages finds colour claims
in **only these three lines**. The defect is fully isolated — nothing else in the corpus repeats
it, and no code depends on it.

### Where to report

The package's `package.json` carries **no `repository` field**, which makes issue-filing harder
than it should be — worth mentioning alongside the fix. Candidate venues:

- `github.com/Nebulit-GmbH/Eventmodelers-Build-Kits`
- `github.com/Nebulit-GmbH/EventModeling-Toolkit`
- Martin Dilger directly — `newsletter.nebulit.de`, `app.eventmodelers.ai`

Suggested framing: *three transposed lines in the workshop facilitation reference, contradicting
your own cheat sheet and book figures; adding a `repository` field to package.json would let
people report this kind of thing directly.*

---

## D2 — `eventmodeling.org` cheat sheet is third-party, and reads as canonical

**Status:** observation, not a defect to report
**Severity:** informational

[`archive/eventmodeling-org/posts/event-modeling-cheatsheet/index.md`](archive/eventmodeling-org/posts/event-modeling-cheatsheet/index.md)
carries `author: sbortz` in its front matter — **not Adam Dymitruk**. It is hosted on the canonical
site and reads as authoritative, and it introduces vocabulary that appears nowhere in Dymitruk's
own article: the four-building-block framing, and **"Trigger"** as the name for the top-row element.

That vocabulary is useful and is used in this repo. But under this project's source-precedence
rule it ranks below Dymitruk's and Dilger's own material, and claims resting on it should say so.
See `METHOD-REFERENCE.md` § *Source precedence*.
