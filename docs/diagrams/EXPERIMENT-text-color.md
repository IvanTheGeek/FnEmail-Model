# Experiment — can markdown colour text?

Testing whether coloured **text** is available as an alternative or supplement to the emoji chips
used in [`README.md`](README.md). Same empirical method as
[`EXPERIMENT-block-labels.md`](EXPERIMENT-block-labels.md): try every candidate, record what each
renderer actually does, keep only what works in both.

**Two renderers must agree** — GitHub (browser, desktop and phone) and the Claude Android app.

Open this page in both and fill in the verdict table at the bottom.

---

## 1. Inline HTML — the obvious approaches

<span style="color:red">A. span with a style attribute</span>

<font color="red">B. font tag with a color attribute</font>

<span class="red">C. span with a class</span>

<div style="color:red">D. div with a style attribute</div>

> ✅ **Tested 2026-08-06 — all four fail in *both* renderers.** No colour anywhere; every one is
> plain black text on GitHub and in the Claude app alike. The feared asymmetry — colour in one
> renderer, silence in the other — **does not occur**, so there is no trap here to avoid.
>
> One incidental difference: `<div>` is a block element, and the two renderers disagree on the
> break around it. GitHub runs the following paragraph straight on; the app leaves a gap. Neither
> colours it, but it is a reason to avoid raw block-level HTML in these documents regardless.

**Expected: all four fail on GitHub.** Its sanitiser strips `style` and `class` attributes and
removes `<font>` entirely — confirmed against the `/markdown` API. Included anyway because the
Claude app may not sanitise, and a technique that works in only one renderer is worse than none:
it looks fine where you author it and silently degrades where the model is read.

---

## 2. LaTeX math — the one GitHub actually processes

Plain words: $\color{red}{red}$ and $\color{green}{green}$ and $\color{blue}{blue}$

A phrase: $\color{red}{\text{a phrase in red}}$

**Without** `\text{}`, hyphens become minus signs: $\color{red}{math-color-red}$

**With** `\text{}`: $\color{red}{\text{math-color-red}}$

Inside a table cell, which is where the chips live:

| element | chip | coloured text |
|:--|:--|:--|
| Event | 🟧 | $\color{orange}{\text{Event}}$ |
| Command | 🟦 | $\color{blue}{\text{Command}}$ |
| Read Model | 🟩 | $\color{green}{\text{Read Model}}$ |
| hotspot | 🟥 | $\color{red}{\text{hotspot}}$ |

⚠️ **Known distortion.** In the Claude app this renders in LaTeX math style — italic serif — and
without `\text{}` a hyphen becomes a minus sign with maths spacing. `math-color-red` came out as
*math − color − red*. The `\text{}` wrapper is what to watch in the results above.

---

## 3. Diff code blocks — colour via the syntax highlighter

```diff
+ a plus line renders green
- a minus line renders red
! a bang line may render orange
@@ an at-at line may render blue @@
# a hash line is a comment
  an unmarked line is plain
```

Works on GitHub — confirmed, the API emits `pl-mi1` and `pl-md` highlight classes. But it is only
available **inside a code block**, offers roughly four colours whose meaning is fixed by diff
semantics, and hijacks the first character of every line. Useless for a table cell.

---

## 4. The control — what we use now

Emoji chips are **characters, not markup**, so no sanitiser can touch them:

🟧 Event · 🟦 Command · 🟩 Read Model · ⬜ rendered UI · ⬛ wire · 🟨 external · 🟥 hotspot

| | **`Helo`** — C |
|:--|:--|
| ⬛ **Actor** | `C: HELO bar.com`&#10;<br>`S: 250 foo.com` |
| 🟦 **Command** | **Helo** |
| 🟧 **Event** | **ClientIdentified**&#10;<br>`claimed_domain: "bar.com"` |

---

## Verdicts

Fill in from both renderers. A technique needs **two ✅** to be usable.

| # | Technique | GitHub | Claude Android | Usable in a table cell? | Notes |
|:--|:--|:--|:--|:--|:--|
| 1A | `<span style>` | ❌ | ❌ | ❌ | plain text in both |
| 1B | `<font color>` | ❌ | ❌ | ❌ | plain text in both |
| 1C | `<span class>` | ❌ | ❌ | ❌ | plain text in both |
| 1D | `<div style>` | ❌ | ❌ | ❌ | plain text in both; also **breaks paragraph flow** — GitHub runs the next paragraph on, the app leaves a gap |
| 2 | `$\color{}{}$` | | | ✅ syntactically | distorts to math italic |
| 2 | `$\color{}{\text{}}$` | | | ✅ syntactically | `\text{}` should stop the distortion |
| 3 | ` ```diff ` | ✅ | ✅ | ❌ code block only | ~4 fixed colours |
| 4 | emoji chip | ✅ | ✅ | ✅ | current convention |

---

## What would change if maths passes both

Nothing immediately, and probably nothing ever — but worth stating so the question is closed rather
than left open.

**The chip is a swatch, not a label.** Colour in Event Modeling *is* the element type, and a chip
puts that colour beside the name without altering the name. Coloured text would have to recolour
the name itself, which is a different claim: it says the *word* is orange rather than that the
*thing* is.

**Text stays greppable; a swatch stays a swatch.** `🟧 **Event**` greps as `Event`.
`$\color{orange}{\text{Event}}$` does not, and the project already chose the greppable form once —
that is why the slice-type chip is `🟦C` and not a bare square.

**And maths is a heavier dependency.** It needs a maths renderer, not a markdown renderer, which is
the same class of dependency Mermaid was — and shedding that dependency is the whole reason these
tables exist.

So the likely outcome is that maths works and is still not adopted. Recording that is the point:
the next person to ask *"can we just colour the text?"* gets an answer with evidence rather than an
opinion.
