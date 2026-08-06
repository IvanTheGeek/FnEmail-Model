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

> ⚠️ **Tested 2026-08-06 — colour works in both, but `\text{}` is honoured by only one.**
>
> | | GitHub | Claude app |
> |:--|:--|:--|
> | colour appears | ✅ | ✅ |
> | bare `$\color{}{}$` | italic serif, `-` → minus sign | italic serif, `-` → minus sign |
> | **`$\color{}{\text{}}$`** | ✅ **normal body font, hyphens intact** | ❌ **still italic serif, still minus signs** |
> | in a table cell | ✅ normal font, correctly coloured | ⚠️ coloured but serif |
>
> **This is the asymmetry section 1 was checking for**, in a milder form. Not colour-versus-nothing,
> but **clean-versus-mangled**: `$\color{red}{\text{math-color-red}}$` renders as `math-color-red`
> on GitHub and as *math − color − red* in the app. You would author it, see it correct, and it
> would degrade where the model is read on a phone.
>
> The failure is *legible* rather than silent, which is better than `<br>` was — a reader sees odd
> typography rather than two words fused together. But it is still one renderer disagreeing with
> the other about the same source.

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

> ⚠️ **Tested 2026-08-06 — the two renderers give different numbers of colours.**
>
> | line | GitHub | Claude app |
> |:--|:--|:--|
> | `+` | green **+ green background** | green text |
> | `-` | red **+ red background** | red text |
> | `!` | **orange + orange background** | **green — wrong** |
> | `@@` | **purple** | plain |
> | `#` | grey comment | plain |
>
> GitHub gives five distinguishable treatments with background fills; the app gives **two**. The
> `!` line is actively misleading — orange on GitHub, green in the app, so the same source says
> "warning" in one place and "added" in the other.
>
> Two usable colours, code-block-only, first character hijacked. Confirms the original assessment.

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

> ✅ **Tested 2026-08-06 — identical in both renderers.** All seven chips render as distinct
> squares, ⬜ and ⬛ are clearly told apart, `&#10;<br>` breaks correctly, code spans are monospace.
> The one difference is cosmetic and carries no meaning: GitHub draws a grey pill behind code
> spans, the app does not.
>
> **This is the only technique in the experiment that renders the same in both.**

---

## Verdicts

Fill in from both renderers. A technique needs **two ✅** to be usable.

| # | Technique | GitHub | Claude Android | Usable in a table cell? | Notes |
|:--|:--|:--|:--|:--|:--|
| 1A | `<span style>` | ❌ | ❌ | ❌ | plain text in both |
| 1B | `<font color>` | ❌ | ❌ | ❌ | plain text in both |
| 1C | `<span class>` | ❌ | ❌ | ❌ | plain text in both |
| 1D | `<div style>` | ❌ | ❌ | ❌ | plain text in both; also **breaks paragraph flow** — GitHub runs the next paragraph on, the app leaves a gap |
| 2 | `$\color{}{}$` | ⚠️ | ⚠️ | ✅ | colours, but italic serif and `-` → minus in **both** |
| 2 | `$\color{}{\text{}}$` | ✅ | ❌ | ✅ | **GitHub honours `\text{}`, the app ignores it** — clean vs mangled |
| 3 | ` ```diff ` | ✅ 5 colours | ⚠️ 2 colours | ❌ code block only | `!` is **orange on GitHub, green in the app** |
| 4 | emoji chip | ✅ | ✅ | ✅ | **identical in both** — the only one |

---

## Conclusion — closed 2026-08-06

**Chips stay. Nothing else survives both renderers.**

| | outcome |
|:--|:--|
| **Inline HTML** | fails in both. Not a trap, just unavailable |
| **LaTeX maths** | colours in both, but `\text{}` works on GitHub only — **clean on GitHub, mangled in the app** |
| **Diff blocks** | five colours on GitHub, two in the app, and `!` means *warning* in one and *added* in the other |
| **Emoji chips** | identical in both |

The prediction written below, before any of this was run, held up. What the testing added was a
reason the prediction did not contain: **both alternatives disagree between the two renderers this
project has to satisfy.** Reasoning alone would not have found that — it needed the phone and the
browser side by side.

One meta-result worth keeping. The failure modes differ in how loudly they fail, and that ordering
matters more than the pass/fail: `<br>` failed **silently** (two words fused, nothing to notice),
maths fails **legibly** (odd typography, obviously wrong), and diff's `!` line fails
**deceptively** (a different meaning in each renderer, both plausible). Deceptive is the worst of
the three, and it is the one a reader is least likely to catch.

---

## What would have changed if maths had passed both

Written before the test, kept unedited.

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
