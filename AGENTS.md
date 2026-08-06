# Working rules for agents in this repository

Operating instructions. For what the project *is* — scope, model state, open hotspots, reading
order — see [`docs/HANDOFF.md`](docs/HANDOFF.md). This file is only the rules that produce bad work
when broken.

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
grep -rniE '\b(colour|behaviour|honour|centre|modelling|labell|maths|whilst)\w*|\b(organis|sanitis|recognis|generalis|prioritis|summaris|apologis)(e|es|ed|ing|ation|er)\b|\b(analyse|analysed|analysing)\b' --include='*.md' .

# commit messages, before you write the next one
git log -n 20 --format='%B' | grep -niE '\b(colour|behaviour|honour|centre|modelling|labell|maths|whilst)\w*|\b(organis|sanitis|recognis|generalis|prioritis|summaris|apologis)(e|es|ed|ing|ation|er)\b|\b(analyse|analysed|analysing)\b'
```

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

**Existing commit messages are left as they are.** Rewriting thirty commits to fix spelling would
destroy the history that records how the project actually went, for a cosmetic gain. Recorded here
instead, per rule 4.

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
see rule 7 — but the discipline did not.

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

**Line breaks inside a table cell are `&#10;<br>` — both halves, always.** GitHub uses the `<br>`
and collapses the entity; the Claude Android app strips the `<br>` and uses the entity. A lone
`<br>` fails **silently and only on a phone**, fusing two words. Tested in
[`docs/diagrams/EXPERIMENT-text-color.md`](docs/diagrams/EXPERIMENT-text-color.md).

**Color is carried by emoji chips**, not by markup. Every text-coloring alternative was tested and
fails or diverges between renderers. 🟧 Event · 🟦 Command · 🟩 Read Model · ⬜ rendered UI ·
⬛ wire · 🟨 external · 🟥 hotspot.

**No Mermaid.** Diagrams are markdown tables — it needs a diagram engine the Android app lacks.

**ASCII art uses printable ASCII only — `| - + / \ ^ _`.** Box-drawing characters (**U+2500 to U+257F**)
are **not in most monospace fonts**, so they fall back to a substitute at a different advance width
and the alignment collapses. It aligns on a desktop and breaks on a phone in **both** renderers,
because it is a font problem rather than a markup one — no markup change can fix it. The same
caution applies to the ellipsis, em dash and middot inside a block where columns must line up; they are fine in
prose. Tested in [`docs/diagrams/EXPERIMENT-inline-styling.md`](docs/diagrams/EXPERIMENT-inline-styling.md).

**Monospace marks what the protocol fixes; standard font marks what varies.** `S: 220` foo.com
Simple Mail Transfer Service Ready — the reply code is monospace, the operator-configurable text is
not. `claimed_domain`: bar.com — the field name is monospace, the instance value is not. One axis,
font family, never weight or slant. **Values carry no quotation marks.** Settled empirically in
[`docs/diagrams/EXPERIMENT-inline-styling.md`](docs/diagrams/EXPERIMENT-inline-styling.md) against
thirty alternatives.

**Bold does not apply to monospace in the Claude app.** ``**`code`**`` renders at normal weight
there and bold on GitHub; `<code><b>` fails too, because the app strips `<code>`. There is no third
route, so **monospace text cannot be bolded** for a reader on the app. Italic on a code span *does*
work. The three usable axes are font family, italic, and bold on standard text only. Tested in
[`docs/diagrams/EXPERIMENT-inline-styling.md`](docs/diagrams/EXPERIMENT-inline-styling.md).

**To show a backtick inside a code span, use double backticks as the delimiter.** Backslash escapes
**do not work inside a code span** — the backslash renders literally and the escaped backtick still
closes the span, splitting the content in half. Write ``` ``**`code`**`` ``` and never
`` `**\`code\`**` ``.

**Refer to slices by name, never by number.** Numbering is an artifact of the current altitude and
scope; change either and every cross-reference breaks silently. Counts are fine — "twelve slices"
is a fact, "slice 4" is a handle that moves.

## 6. Commit messages carry the reasoning

They are long here on purpose. State what changed, **why**, and what was rejected. A commit that
records a correction should say what the old claim was — the message is often the only place the
superseded reasoning survives.

End with:

```
Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>
```

## 7. Nothing third-party enters this repository

The Event Modeling research lives in a **separate private repo**,
`git@github.com:IvanTheGeek/event-modeling-research.git`. It holds two commercial books, a mostly
unlicensed mirrored corpus, and machine transcripts of copyrighted speech.

**FnEmail cites, quotes briefly with attribution, and links. It never reproduces.** This is what
keeps FnEmail releasable, and it is not negotiable.

## 8. Walk paths with real data

Placeholders find nothing. Both payload defects this project has found, the orphan field, and H3's
resolution all came from instantiating a field with a real value and seeing that nothing consumed
it. `<address>` teaches nothing; `"198.51.100.40"` does.

A clean path confirms; a messy path discovers. Walk the messy one first.

## 9. Do not re-litigate settled decisions

`docs/HANDOFF.md` §4 lists them. If evidence genuinely overturns one, that is a ⚠️ correction under
rule 4 with the evidence attached — not a quiet change.

## 10. Extensions stay parked

[`docs/event-model-extensions.md`](docs/event-model-extensions.md) is deliberately not applied, so
the orthodox model stays measurable against it. Do not fold extension ideas into the model without
being asked.
