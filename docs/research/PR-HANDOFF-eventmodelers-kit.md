# PR Handoff — eventmodelers kit color fix

**Self-contained.** Everything needed to open this PR is in this one file. Nothing else has to be
read first, and it is written so a fresh session can pick it up cold.

- **Status:** ready to submit, not submitted
- **Blocked by:** permission classifier denied adding a third-party repo with push access
- **Verified:** 2026-08-04, against `Nebulit-GmbH/Eventmodelers-Build-Kits` @ `e317bdf`
- **Effort:** two lines

---

## When you're at a laptop

Say something like *"do the eventmodelers PR"* and point me at this file.

For me to do it end-to-end I need **push-scoped access to a third-party repo** so I can fork.
That was denied last time by the auto-mode classifier. Either approve it when prompted, or run
[§4](#4-commands) yourself and hand me the PR URL — I can take it from there.

If nothing has changed upstream, [§4](#4-commands) works verbatim. Run [§3](#3-re-verify-first)
first regardless; it takes seconds and tells you whether it's already fixed.

---

## 1. The defect

The workshop facilitation reference has the **Event and View sticky-note colors transposed**.

```
Physical sticky notes:
[Green] Event: ________________     ← wrong, should be Orange
[Blue] Command: ________________    ← correct
[Orange] View: ________________     ← wrong, should be Green
```

### Why it's wrong

| Source | Command | Event | View / Read Model |
|---|---|---|---|
| **Dilger, Event Modeling Cheat Sheet (2026-07)** | blue | **orange** | **green** |
| **Dilger, *Understanding Eventsourcing*** — figures p222, p234 | blue | **orange** | **green** |
| eventmodeling.org cheat sheet (2022) | blue | yellow | **green** |
| Dymitruk, *"Event Modeling: What is it?"* | blue | — | **green** ("green to-do list") |
| **this file** | blue | **green** ❌ | **orange** ❌ |

Every source puts View at green. Event is legitimately yellow (older) or orange (current) — green
is not among its readings. Command is blue everywhere, including here.

This is an **internal inconsistency**, not a disputed convention: it contradicts the same author's
own cheat sheet and his own book figures.

### Why it's worth fixing

It sits in the *facilitation* guide — the page a facilitator follows while a room of first-timers
copies the convention onto real stickies. Wrong there propagates further than wrong in a reference
table.

### Scope

These are the **only three color claims** in the entire repository, and in all 68 markdown files
across the four published `@eventmodelers/*` packages. Nothing else repeats it; no code depends
on it. The fix is fully isolated.

---

## 2. Location

```
repo    github.com/Nebulit-GmbH/Eventmodelers-Build-Kits
branch  main
path    eventmodelers-cli/stacks/modeling-kit/templates/.claude/skills/
        eventmodeling-brainstorming-events/references/
        facilitating-event-modeling-workshops.md
lines   89 and 93   (section "### 4. Create Templates")
```

Published as `@eventmodelers/agent-modeling-kit` (MIT, Martin Dilger). The upstream file is
**byte-identical** to the published npm copy — diffed, confirmed.

Repo has **no CONTRIBUTING and no root LICENSE**, so there's no stated process. A plain PR
against `main` is the reasonable default.

---

## 3. Re-verify first

```bash
curl -s https://raw.githubusercontent.com/Nebulit-GmbH/Eventmodelers-Build-Kits/main/eventmodelers-cli/stacks/modeling-kit/templates/.claude/skills/eventmodeling-brainstorming-events/references/facilitating-event-modeling-workshops.md \
  | sed -n '85,97p'
```

Expect `[Green] Event` at line 89 and `[Orange] View` at line 93. If they're already correct,
it's been fixed — stop, and delete this file.

---

## 4. Commands

```bash
gh repo fork Nebulit-GmbH/Eventmodelers-Build-Kits --clone --remote=false
cd Eventmodelers-Build-Kits

# don't leak a real address into a public repo's history
git config user.email "ivanthegeek@users.noreply.github.com"
git config user.name  "ivanthegeek"

git checkout -b fix/workshop-sticky-colors
git apply /path/to/FnEmail/docs/research/kit-color-fix.patch
git diff --stat          # expect: 1 file changed, 2 insertions(+), 2 deletions(-)

git commit -am "Fix transposed Event/View sticky-note colors in workshop facilitation reference"
git push -u origin fix/workshop-sticky-colors
gh pr create --repo Nebulit-GmbH/Eventmodelers-Build-Kits \
  --title "Fix transposed Event/View sticky-note colors in workshop facilitation reference" \
  --body-file pr-body.md
```

**On the email line:** a public commit with `ivan@claude.ai.irxops.com` puts that address in a
third-party repository's history permanently. The noreply alias avoids it. Change it if you'd
rather use a different identity — just decide before committing, not after.

---

## 5. The patch

Also at [`kit-color-fix.patch`](kit-color-fix.patch). Generated against `main` @ `e317bdf`,
verified to apply. Inlined here so this file stands alone.

```diff
diff --git a/eventmodelers-cli/stacks/modeling-kit/templates/.claude/skills/eventmodeling-brainstorming-events/references/facilitating-event-modeling-workshops.md b/eventmodelers-cli/stacks/modeling-kit/templates/.claude/skills/eventmodeling-brainstorming-events/references/facilitating-event-modeling-workshops.md
index 0dd6b09..6d10f6c 100644
--- a/eventmodelers-cli/stacks/modeling-kit/templates/.claude/skills/eventmodeling-brainstorming-events/references/facilitating-event-modeling-workshops.md
+++ b/eventmodelers-cli/stacks/modeling-kit/templates/.claude/skills/eventmodeling-brainstorming-events/references/facilitating-event-modeling-workshops.md
@@ -86,11 +86,11 @@ Prepare sticky note templates or digital shapes:
 
 ```
 Physical sticky notes:
-[Green] Event: ________________
+[Orange] Event: ________________
 
 [Blue] Command: ________________
 
-[Orange] View: ________________
+[Green] View: ________________
 
 Virtual shapes:
 Same colors with text fields
```

If the patch won't apply because the file moved, the edit is trivial to redo by hand: swap
`Green` and `Orange` on the Event and View lines. Leave `[Blue] Command` alone.

---

## 6. PR title

```
Fix transposed Event/View sticky-note colors in workshop facilitation reference
```

## 7. PR body

Copy from here down into `pr-body.md`.

---

The workshop facilitation reference has the Event and View sticky-note colors transposed.

`eventmodelers-cli/stacks/modeling-kit/templates/.claude/skills/eventmodeling-brainstorming-events/references/facilitating-event-modeling-workshops.md`,
lines 89 and 93, currently read `[Green] Event` and `[Orange] View`.

This contradicts the Event Modeling Cheat Sheet and the figures in *Understanding Eventsourcing*,
which both show **orange events** against **green read models**. `[Blue] Command` is correct and
is unchanged.

It matters slightly more than a typo because it's in the *facilitation* guide — it's what a room
of first-timers copies onto real stickies while they're still learning the convention.

These are the only three color claims in the repository, so the change is fully isolated.

Unrelated but possibly useful: the published `@eventmodelers/*` packages have no `repository`
field in `package.json`, so there's no link from npm back to this repo. Adding one would make
issues like this easier to report — I only found the source by guessing at org names.

*Found while building an Event Modeling reference from primary sources. Research and patch
prepared with Claude; reviewed and submitted by me.*

---

## 8. Optional follow-up

Worth a separate issue if you want to be thorough — add `"repository"` to `package.json` for all
four packages (`agent-modeling-kit`, `build-kit-node`, `build-kit-supabase`, `build-kit-axon`).
None of them have it. Mentioned in the PR body above; splitting it out is cleaner if the
maintainer prefers.

## 9. Context

Full write-up: [`UPSTREAM-DEFECTS.md`](UPSTREAM-DEFECTS.md).
Color convention as this project settled it: [`METHOD-REFERENCE.md`](METHOD-REFERENCE.md).
Local mirror of the affected file:
[`archive/eventmodelers-kits/agent-modeling-kit/skills/eventmodeling-brainstorming-events/references/facilitating-event-modeling-workshops.md`](archive/eventmodelers-kits/agent-modeling-kit/skills/eventmodeling-brainstorming-events/references/facilitating-event-modeling-workshops.md)
