# Experiment — can a table cell be top-aligned?

Left alignment is settled: `|:--|` in the delimiter row, which GitHub emits as `align="left"`.
**Vertical alignment has no markdown syntax at all.** The delimiter row controls one axis and there
is no second one.

The symptom is visible in every step of
[`../paths/helo-direct-single-recipient.md`](../paths/helo-direct-single-recipient.md): a one-word
label like `MTA Client` sits against a three-line payload and floats to the middle of it, so the
label and the line it names are not level.

**Two renderers must agree** — GitHub (browser, desktop and phone) and the Claude Android app.

| Renderer | Version |
|:--|:--|
| **Claude Android app** | fill in at time of test — the app updates independently of this repository |
| **Claude Code desktop app** | fill in at time of test |
| **GitHub** | web renderer, viewed in Chrome on Android and on desktop; predictions cross-checked against the `POST /markdown` API |

⚠️ **There are three renderers, not two.** Every experiment in this directory before 2026-08-07 was
recorded in two columns, GitHub and *the Claude app*, on the assumption that inlining something into
chat tested one thing. Variant C below shows it tests **two**, and they disagree: the Android app
**strips** raw HTML and runs the pieces together, while the Claude Code desktop app **prints the
tags as literal text**. Both fail, so no earlier verdict is overturned — every technique adopted so
far is pure markdown, which all three handle. But the earlier results were recorded against a
renderer set that was one short, and *"tested in the app"* is from here on an ambiguous claim.
**Say which app.**

---

## ⚠️ Why this document exists — a failed test, recorded

On 2026-08-07 this question was declared closed on evidence that did not support it. A raw HTML
table was **pasted into a chat message**, and it broke the Claude app. That much was real. It was
then compared against the same document **on GitHub**, where it looked fine — and the difference
was read as *the app cannot render HTML tables*.

**The two sides were not the same input.** The HTML had never been committed. GitHub was rendering
the plain markdown file; the app was rendering a chat message containing markup the file did not
have. The only thing the comparison established is that the two sources differed, which was already
known, because one of them was written that way on purpose.

**A comparison is evidence only when both sides receive the same input.** That is the lesson, and
it is more useful than the verdict it destroyed. This document exists so the question can be
settled with the markup *committed*, so that both renderers are fed from this file.

Two things remain genuinely untested, and the variants below separate them:

1. Does the Claude app render a raw `<table>` **at all**?
2. If the app broke on something narrower — the **blank lines inside the cells** — then the table
   was never the problem.

---

## A. Control — markdown table, no technique

The current form. Expect `MTA Client` and `Event` centered against their tall neighbors.

| 🟦 C · Step 1 | `AcceptConnection` |
|:--|:--|
| MTA Client | ⬛ `220` foo.com Simple Mail Transfer Service Ready |
| | 🟦 **AcceptConnection** |
| Event | 🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |

---

## B. Padding — pure markdown, no HTML at all

No HTML, so **neither renderer can reject it**. The short cell is padded with trailing line breaks
until it is as tall as its neighbor; a centered three-line cell whose content is on line one puts
that content at the top.

| 🟦 C · Step 1 | `AcceptConnection` |
|:--|:--|
| MTA Client&#10;<br>&#10;<br> | ⬛ `220` foo.com Simple Mail Transfer Service Ready |
| | 🟦 **AcceptConnection** |
| Event&#10;<br>&#10;<br> | 🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |

**The known weakness is wrapping.** The padding is a fixed line count, but the *rendered* height of
the tall cell depends on the viewport. On a narrow phone the payload wraps to more lines than it has
on a desktop, and the padding no longer matches — it under-pads and the label drifts back toward the
middle, or over-pads and the label rises above the first line. It is correct only at the width it
was counted for. **Judge this one on the phone, not on the desktop**, since the desktop is the case
it is most likely to get right by accident.

---

## C. Raw HTML table, minimal — does the app render one at all?

No `valign`, no blank lines, no attributes. This variant asks **one** question: does the Claude app
render a raw `<table>`? If this renders as a table, HTML tables are available and the earlier
breakage was caused by something narrower. If it renders as a pile of text, the route is dead and
D and E cannot be rescued.

`<code>` is used instead of backticks because markdown does not run inside a raw HTML block. The app
is known to strip `<code>`, so **monospace is expected to be lost here even if the table works** —
that is a separate defect from whether the table renders.

<table>
<tr><th>🟦 C · Step 1</th><th><code>AcceptConnection</code></th></tr>
<tr><td>MTA Client</td><td>⬛ <code>220</code> foo.com Simple Mail Transfer Service Ready</td></tr>
<tr><td></td><td>🟦 <b>AcceptConnection</b></td></tr>
<tr><td>Event</td><td>🟧 <b>ConnectionAccepted</b><br>&nbsp;&nbsp;<code>peer_address</code>: 203.0.113.20<br>&nbsp;&nbsp;<code>local_address</code>: 192.0.2.10:25</td></tr>
</table>

---

## D. Raw HTML table with `valign="top"`, still no blank lines

If C renders and D also renders, `valign` is the answer and the cost is only the lost monospace.

<table>
<tr><th align="left" valign="top">🟦 C · Step 1</th><th align="left" valign="top"><code>AcceptConnection</code></th></tr>
<tr><td align="left" valign="top">MTA Client</td><td align="left" valign="top">⬛ <code>220</code> foo.com Simple Mail Transfer Service Ready</td></tr>
<tr><td align="left" valign="top"></td><td align="left" valign="top">🟦 <b>AcceptConnection</b></td></tr>
<tr><td align="left" valign="top">Event</td><td align="left" valign="top">🟧 <b>ConnectionAccepted</b><br>&nbsp;&nbsp;<code>peer_address</code>: 203.0.113.20<br>&nbsp;&nbsp;<code>local_address</code>: 192.0.2.10:25</td></tr>
</table>

---

## E. Raw HTML table, `valign="top"`, blank lines re-enabling markdown

**This is the variant that broke the chat message.** The blank line inside each cell makes GitHub
parse the contents as markdown again, so backticks and `**` work and the monospace survives. It is
the only variant that gets both top alignment *and* the styling convention — and it is the most
likely to fail, because a blank line is exactly what tells a markdown parser a block has ended.

<table>
<tr><th align="left" valign="top">

🟦 C · Step 1

</th><th align="left" valign="top">

`AcceptConnection`

</th></tr>
<tr><td align="left" valign="top">

MTA Client

</td><td align="left" valign="top">

⬛ `220` foo.com Simple Mail Transfer Service Ready

</td></tr>
<tr><td align="left" valign="top"></td><td align="left" valign="top">

🟦 **AcceptConnection**

</td></tr>
<tr><td align="left" valign="top">

Event

</td><td align="left" valign="top">

🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25

</td></tr>
</table>

---

## Verdicts

Fill in from both renderers. A technique needs **two ✅** to be usable.

Three columns, because there are three renderers. A technique needs all three.

| # | Technique | GitHub | Claude Android | Claude Code desktop | Usable? |
|:--|:--|:--|:--|:--|:--|
| A | markdown, no technique | ✅ table | ✅ table | ✅ table | **control** — no top alignment |
| B | markdown + padding | ✅ | | | |
| C | raw HTML, minimal | ✅ table | ❌ **tags stripped, cells fused** | ❌ **tags printed literally** | ❌ |
| D | raw HTML + `valign` | ✅ table, `valign` kept | ❌ *unreachable* | ❌ *unreachable* | ❌ |
| E | raw HTML + `valign` + blank lines | ✅ table, markdown live | ❌ *unreachable* | ❌ *unreachable* | ❌ |

**C settles D and E without their being run.** All three use the same table elements, and both Claude
renderers refuse those elements — so no attribute or blank-line arrangement inside them can be
reached. Marked unreachable rather than untested, because running them would only re-demonstrate C.

### What C actually did, 2026-08-07

**Android — the tags are consumed and the cell contents run together.** No grid, no rows. Every cell
boundary closed up with **no separator at all**, fusing words across it: *Step 1AcceptConnection*,
*MTA Client*⬛, *ReadyAcceptConnection*, *AcceptConnectionEvent*. This is the silent-fusion failure
mode already on record for the lone `<br>`, and it is the worst kind — the output is plausible prose,
so nothing announces that a table was lost.

**Claude Code desktop — the opposite.** The markup is not interpreted and not stripped; it is
**printed**, tags and all, so the reader sees `<tr><td>` in the body text. That fails loudly, which
is much the better failure, but it fails.

Two incidental results from the Android render, both worth keeping:

- **`<b>` survives and `<code>` does not.** *AcceptConnection* and *ConnectionAccepted* came through
  bold while `220` came through in the body font. That is consistent with the `<code>`-stripping
  already recorded in rule 5, and it locates the behavior in an element allowlist rather than in
  anything about tables.
- **A bare `<br>` produced real line breaks here**, putting `peer_address` and `local_address` on
  their own lines — inside a raw HTML block, with no `&#10;`. Rule 5 records the opposite for a
  **markdown table cell**, and that rule is not overturned by this: the two are different contexts
  and the rule was tested in its own. ⚠️ But it is now an open question rather than a settled one,
  and it matters, because if the app honored `<br>` in both places then `&#10;<br>` would be
  producing *two* breaks in every step table. It does not appear to. **Test both contexts directly
  before touching rule 5.**

---

## Prediction, written before the app was looked at

Kept unedited afterward, per the practice in
[`EXPERIMENT-text-color.md`](EXPERIMENT-text-color.md).

**C decides everything.** If a bare `<table>` does not render in the app, D and E are unreachable
regardless of their merits, and B is the only surviving candidate.

**E is the one worth wanting and the one least likely to survive.** It is the only variant that
keeps the mono-fixed/standard-variable convention *and* aligns to the top. It is also asking a
markdown parser to resume parsing inside a raw HTML block, which is the least standardized corner
of the whole format — CommonMark and GFM disagree about it, and an app with its own parser has no
reason to match either.

**B is the likely survivor and is still probably not adopted**, for the reason recorded against
box-drawing characters in rule 5: it is correct at one viewport width and wrong at others, and it
fails *on a phone* while looking right on a desktop. That is the same silent, device-dependent
failure mode as the lone `<br>`, and this project has already decided once that a technique which
looks right where it is authored and wrong where it is read is worse than no technique.

**So the likely outcome is that nothing is adopted and the middle alignment stays.** Recording that
in advance is the point: if the result comes back the other way, it is a real finding rather than a
rationalization.
