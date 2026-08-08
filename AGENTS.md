# Working rules for agents in this repository

Operating instructions. For what the project *is* — scope, setup, reading order, and where status
lives — see the [README](README.md). For decisions already settled, see
[`docs/DECISIONS.md`](docs/DECISIONS.md). Status itself is tracked at its sources, never in a
parallel ledger. This file is only the rules that produce bad work when broken.

---

## 1. US spelling. Always.

**color, modeling, behavior, organize, analyze, center, labeled, sanitize, honor, recognize.**

Not colour, modelling, behaviour, organise, analyse, centre, labelled, sanitise, honour, recognise.

This has been corrected more than three times and keeps recurring, which is why it is rule one.

**It applies to every word you write, not only to files** — chat replies and **commit messages**
included. Commit messages here are long by design (rule 6), which makes them thousands of words of
prose that a `--include='*.md'` grep never sees. On 2026-08-06 the files were clean and the last
thirty commit messages held **27** British spellings, because the check had been run only against
markdown.

Check both:

```bash
# files
grep -rniE '\b(colour|behaviour|honour|centre|modelling|labell|maths|whilst|artefact)\w*|\b(organis|sanitis|recognis|generalis|prioritis|summaris|apologis|authoris|standardis)(e|es|ed|ing|ation|er)\b|\b(analyse|analysed|analysing)\b' --include='*.md' .

# commit messages, before you write the next one
git log -n 20 --format='%B' | grep -niE '\b(colour|behaviour|honour|centre|modelling|labell|maths|whilst|artefact)\w*|\b(organis|sanitis|recognis|generalis|prioritis|summaris|apologis|authoris|standardis)(e|es|ed|ing|ation|er)\b|\b(analyse|analysed|analysing)\b'
```

*(The `artefact`, `authoris` and `standardis` stems were added 2026-08-08, after three live hits —
two *artefact*s and a *relay-authorisation* in `model-altitude.md`, plus a *standardise* in
`HANDOFF.md` — escaped every earlier sweep because the stems were missing. Tested both directions
on addition: the pattern catches *artefact*, *authorisation*, *standardise* and does not fire on
*artifact*, *authorization*, *standardize*.)*

⚠️ **The pattern is tested by running it, not by reading it.** Three earlier versions were wrong,
each in a different way, and the third was invisible: a script wrote `\b` through a non-raw Python
string, so **literal backspace bytes (0x08)** landed in the file where the word boundaries should
have been. The pattern then matched *nothing at all* — which reads as "no false positives" unless
you also check that it still catches the true ones. `cat -A` was needed to see it.

**Test both directions, always.** A pattern that catches nothing looks exactly like a pattern that
is working, if you only test the negatives.

The other two were wrong in opposite directions:
bare stems matched **analysis** and **analyses**, which are correct US spellings, and the fix then
missed **generalises** because it did not allow the third-person `-es`. It also must not fire on
*advertise, surprise, exercise, compromise* — US English uses `-ise` for those. Verify any change
against a list of both kinds before trusting it. **A check that cries wolf gets ignored, which is
worse than no check.**

⚠️ **Run the check as a gate, not as company.** Later on 2026-08-06 the commit-message grep was put
in the *same* shell invocation as the `git commit` that followed it. It fired correctly, on
*labelling*, and the commit went out anyway — the statements were sequential, so nothing was
conditional on the result. A check whose output nobody waits for is not a check. Run it, read it,
*then* commit.

**Existing commit messages are left as they are.** Rewriting thirty commits to fix spelling would
destroy the history that records how the project actually went, for a cosmetic gain. That applies to
`d8036c2` and its *labelling* too: it is one commit old and force-pushing over it would rewrite a
branch already pushed, which is a worse trade than the typo. Recorded here instead, per rule 4.

**The one exception is a quotation** — see rule 2. The corpus uses the British spelling of *color*,
and a quote keeps it. Prefer naming that fact over reproducing the word.

## 2. Quotations keep their own words

Never restyle a quote to match project spelling, punctuation or capitalization. The sources write
*state-change*, *write column*, *colour*; machine transcripts are lowercase, unpunctuated and
contain transcription errors. **Copy them exactly, errors included.** Rewriting a quote silently
corrupts the sourcing, and the whole method-reference apparatus depends on quotes being checkable.

Machine transcripts must be labeled as such wherever quoted, so a reader knows the irregularities
belong to the source.

## 3. Verify citations. Do not trust a remembered one

Every method claim traces to a file and a line. `docs/research/` moved out to a separate repo —
see rule 7 — but the discipline did not. Method conventions now also trace to the method repo's
documents (`EventModeling/docs/` — see rule 11); a claim about what this project settled traces to
[`docs/DECISIONS.md`](docs/DECISIONS.md).

**When citing an RFC, open the RFC.** This session produced two citation failures that were caught
only by reading the text: a claim attributed to §3.6 that is not there, and a "MUST" that is a
"SHOULD" and changed a tier classification when corrected. Both were plausible and both were wrong.

The archived specs are in [`docs/rfc/`](docs/rfc/).

## 4. Record corrections. Do not quietly fix them

When something turns out wrong, **leave the original and add a ⚠️ block** saying what was claimed,
why it was wrong, and what replaced it. Several documents carry these. They are the point, not
clutter — the reasoning is worth more than a clean surface, and a silent fix destroys the evidence
that the process works.

This applies to your own work in the same session.

## 5. Markdown conventions

**The canonical rule set is the method repo's
[`docs/rendering.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/rendering.md)** —
every convention there was measured, not assumed, and the experiments that measured them live in
this repo's [`docs/diagrams/`](docs/diagrams/), which that document names as its evidence base.
This rule keeps what the method repo deliberately does not repeat: the short form of each
convention, the ⚠️ incident records behind them, and the SMTP-specific rules.

**Line breaks inside a table cell are `&#10;<br>` — both halves, always.** GitHub uses the `<br>`
and collapses the entity; the Claude Android app strips the `<br>` and uses the entity. A lone
`<br>` fails **silently and only on a phone**, fusing two words. Confirmed across all three
renderers 2026-08-07 in
[`docs/diagrams/EXPERIMENT-line-break.md`](docs/diagrams/EXPERIMENT-line-break.md).

⚠️ **Known cost, accepted deliberately: the Claude Code desktop app renders an extra gap.** It
honors *both* halves, so it breaks twice. **Do not "fix" this by dropping a half.** Drop the `<br>`
and GitHub stops breaking at all — the entity emits a newline character, which HTML collapses to a
space. Drop the entity and the phone **fuses the words**. Both of those lose information; the gap
only looks untidy, and it looks untidy in the one renderer where nothing is being read for the
first time. One row per line is the only technique that cannot fail, and it is rejected on
row-count cost — written up in the experiment with the evidence attached, ready to adopt if the
cost ever stops mattering.

**Color is carried by emoji chips**, not by markup — every text-coloring alternative was tested and
fails or diverges between renderers. 🟧 Event · 🟦 Command · 🟩 Read Model · ⬜ rendered UI ·
⬛ wire · 🟨 external / required-first · 🟥 hotspot · 🟤 nothing, and only ever in a `Given` row.

**No Mermaid.** Diagrams are markdown tables — it needs a diagram engine the Android app lacks.

**ASCII art uses printable ASCII only — `| - + / \ ^ _`.** Box-drawing characters (**U+2500 to
U+257F**) fall back to a substitute at a different advance width on a phone, and the alignment
collapses — a font problem rather than a markup one, so no markup change can fix it. The same
caution applies to the ellipsis, em dash and middot inside a block where columns must line up;
they are fine in prose. Tested in
[`docs/diagrams/EXPERIMENT-inline-styling.md`](docs/diagrams/EXPERIMENT-inline-styling.md).

**Monospace marks what the protocol fixes; standard font marks what varies.** `S: 220` foo.com
Simple Mail Transfer Service Ready — the reply code is monospace, the operator-configurable text is
not. `claimed_domain`: bar.com — the field name is monospace, the instance value is not. One axis,
font family, never weight or slant; **values carry no quotation marks**. And **bold does not apply
to monospace in the Claude app** — ``**`code`**`` silently loses its weight there and `<code><b>`
fails too — so monospace text is never bolded; italic on a code span does work. Settled
empirically in
[`docs/diagrams/EXPERIMENT-inline-styling.md`](docs/diagrams/EXPERIMENT-inline-styling.md) against
thirty alternatives.

**No raw HTML tables, and no padding cells with trailing line breaks.** Both Claude renderers
refuse a raw `<table>` — the Android app strips the tags and runs the cell contents together, the
desktop app prints the tags as literal text — and padding buys nothing, because **table cells
already top-align in both Claude renderers**.

⚠️ **The claim this block used to make — that a short label floats to the middle of a tall row — was
never verified in any renderer.** It was taken from the HTML default for a table cell and asserted
from memory, which is rule 3's failure in a place rule 3 did not think to look: the habit was built
for RFCs and citations, and a remembered CSS default is the same kind of claim. Tested 2026-08-07,
the control renders **top-aligned** in both Claude apps with no technique applied. The whole
vertical-alignment question was therefore partly manufactured, and the fix for a symptom that was
never confirmed cost two false verdicts before the control was finally looked at. **Check that the
problem exists before testing solutions to it.** Settled in
[`docs/diagrams/EXPERIMENT-vertical-align.md`](docs/diagrams/EXPERIMENT-vertical-align.md).

**Angle brackets outside a code span are eaten.** `<CRLF>` in running text renders as *nothing at
all* — the parser takes it for an unknown HTML tag and drops it silently. An address fares slightly
better and still loses information: `<Smith@bar.com>` becomes a mailto link with **the angle
brackets stripped**, and in SMTP those brackets are path syntax — `MAIL FROM:<>` is not
`MAIL FROM:`. Escaping the opening bracket (`\<`) restores both, but GFM still autolinks a bare
address; only a code span suppresses that. This bit on 2026-08-06, when moving values out of
monospace to apply the mono-fixed/standard-variable rule silently deleted six angle-bracket pairs
that had been safe inside their old code spans.

**Re-render, never just re-read.** Any convention change that moves text out of a code span, and
any table surgery — adding, removing or reordering rows — is verified in the actual renderers
before it is called done. Widened from convention changes to all table surgery on 2026-08-08,
after a row removal fused two tables silently (commit a1dc975).

**To show a backtick inside a code span, use double backticks as the delimiter.** Backslash escapes
**do not work inside a code span** — the backslash renders literally and the escaped backtick still
closes the span, splitting the content in half. Write ``` ``**`code`**`` ``` and never
`` `**\`code\`**` ``.

**Refer to slices by name, never by number.** Numbering is an artifact of the current altitude and
scope; change either and every cross-reference breaks silently. Counts are fine — "twelve slices"
is a fact, "slice 4" is a handle that moves. The method repo states the same discipline in
[`docs/path-and-step-form.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/path-and-step-form.md).

## 6. Commit messages carry the reasoning

They are long here on purpose. State what changed, **why**, and what was rejected. A commit that
records a correction should say what the old claim was — the message is often the only place the
superseded reasoning survives.

End with a `Co-Authored-By` trailer naming **the agent and model that actually wrote the commit**.
The point of the rule is disclosure — that AI helped, and which agent and model it was — not any
particular name. So the line varies with the author: Claude Opus 5, Claude Sonnet 5, Claude Fable 5,
Codex, or any other agent, each under its own attribution address. For a Claude model:

```
Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>
```

Do not copy the trailer from an earlier commit — earlier commits name earlier models. State what is
true for *this* commit.

## 7. Nothing third-party enters this repository

The Event Modeling research lives in a **separate private repo**,
`git@github.com:IvanTheGeek/event-modeling-research.git`. It holds two commercial books, a mostly
unlicensed mirrored corpus, and machine transcripts of copyrighted speech.

**FnEmail-Model cites, quotes briefly with attribution, and links. It never reproduces.** This is
what keeps this repository releasable, and it is not negotiable. *(The bare name `FnEmail` now
designates the future code repo — see rule 11.)*

## 8. Walk paths with real data

Placeholders find nothing. Both payload defects this project has found, the orphan field, and the
resolution of H3, the Directory-context hotspot (settled — see
[`docs/DECISIONS.md`](docs/DECISIONS.md)) all came from instantiating a field with a real value and
seeing that nothing consumed it. `<address>` teaches nothing; `"198.51.100.40"` does.

A clean path confirms; a messy path discovers. Walk the messy one first.

## 9. Do not re-litigate settled decisions

[`docs/DECISIONS.md`](docs/DECISIONS.md) lists them. If evidence genuinely overturns one, that is
a ⚠️ correction under rule 4 with the evidence attached — not a quiet change.

## 10. Extensions stay parked

The author's extension ideas live in the method repo —
[`EventModeling/docs/extensions.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/extensions.md),
moved there 2026-08-08 from this repo's `docs/event-model-extensions.md` (history here) — and are
deliberately not applied, so the orthodox model stays measurable against them — learn the method
before extending it. Do not fold extension ideas into the model without being asked.

## 11. Four repositories, one direction of flow

The repos mirror the modeling layers — method (`EventModeling`, public), model (this repo), code
(`FnEmail`, future), sources (`event-modeling-research`, private). **Method-generic insight flows
to `EventModeling`, with FnEmail demoted to worked example; SMTP specifics stay here and cite the
method repo rather than restating it.** Research material is cited, never reproduced, into either
public repo. The ruling and the full map are in [`docs/DECISIONS.md`](docs/DECISIONS.md).

## 12. Unknowns become hotspots

On the model, not in an appendix. An open question gets a 🟥 marker at the point where it bites,
and hotspots travel with the steps that meet them. One caveat applies: hotspots are Dilger's
device, not Dymitruk's, so this practice is legitimate but not method-neutral — see the caveat in
[`docs/DECISIONS.md`](docs/DECISIONS.md).

## 13. EXPLORE- and WORKING- prefixes mark the working stance

An `EXPLORE-` document is an exploration in progress: while one is open, other documents
deliberately lag it — **cite the active EXPLORE file, not `event-model.md`**, for anything the
exploration governs. A `WORKING-` prefix marks a path still being walked through; its contents may
move, and nothing else is reconciled to it until the prefix drops. Defined in commit 9aa2578;
the [README](README.md) names the currently active files.
