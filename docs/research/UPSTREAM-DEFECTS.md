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

### Upstream location — confirmed

```
repo   : github.com/Nebulit-GmbH/Eventmodelers-Build-Kits   (branch: main)
path   : eventmodelers-cli/stacks/modeling-kit/templates/.claude/skills/
         eventmodeling-brainstorming-events/references/
         facilitating-event-modeling-workshops.md
lines  : 89, 93
```

The upstream file is **byte-identical** to the published npm copy, and these are the only three
colour claims in the entire repository.

The npm `package.json` carries **no `repository` field**, which is likely why this has gone
unreported — there is no discoverable link from the package to its source. Worth raising
alongside the fix. There is also no `CONTRIBUTING.md` or root `LICENSE` in the repo.

### Ready-to-apply patch

[`kit-colour-fix.patch`](kit-colour-fix.patch) — generated against `main` and verified to apply.
Two lines changed.

```bash
gh repo fork Nebulit-GmbH/Eventmodelers-Build-Kits --clone
cd Eventmodelers-Build-Kits
git checkout -b fix/workshop-sticky-colours
git apply /path/to/kit-colour-fix.patch
git commit -am "Fix transposed sticky-note colours in workshop facilitation reference"
git push -u origin fix/workshop-sticky-colours
gh pr create --title "Fix transposed Event/View sticky-note colours in workshop facilitation reference"
```

If you run this, consider `git config user.email` set to your GitHub noreply alias
(`<username>@users.noreply.github.com`) — a public commit otherwise puts your real address in the
history permanently.

### Suggested PR body

> The workshop facilitation reference has the Event and View sticky-note colours transposed.
>
> `eventmodelers-cli/stacks/modeling-kit/.../facilitating-event-modeling-workshops.md`, lines 89
> and 93, currently read `[Green] Event` and `[Orange] View`. This contradicts the Event Modeling
> Cheat Sheet and the figures in *Understanding Eventsourcing* (e.g. the cart diagrams), which
> both show **orange events** against **green read models**. `[Blue] Command` is correct.
>
> It matters a little more than a typo because it sits in the *facilitation* guide — it's what a
> room of first-timers copies onto real stickies while learning the convention.
>
> These are the only three colour claims in the repository, so the fix is fully isolated.
>
> Unrelated but possibly useful: the published `@eventmodelers/*` packages have no `repository`
> field in `package.json`, so there's no link from npm back here. Adding one would make issues
> like this easier to report.
>
> *Found while building an Event Modeling reference from primary sources. Research and patch
> prepared with Claude; reviewed and submitted by me.*

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
