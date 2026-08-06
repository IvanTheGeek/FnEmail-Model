# Experiment — inline styling to carry meaning inside a string

Third in the series, after [`EXPERIMENT-block-labels.md`](EXPERIMENT-block-labels.md) (Mermaid
label syntax, superseded) and [`EXPERIMENT-text-color.md`](EXPERIMENT-text-color.md) (color, closed
— chips won). Same method: try every candidate, open it in **both** renderers, keep only what
works in both.

**Renderers under test.** A technique must work in both.

| Renderer | Version |
|:--|:--|
| **Claude Android app** | **1.260721.20** (build 26072120) |
| **GitHub** | web renderer, viewed in Chrome on Android and on desktop; predictions cross-checked against the `POST /markdown` API |

Record the app version with any future result — the app updates independently of this repository,
and a verdict without a version cannot be re-checked.

---

## The problem

A wire line is not one thing. It has parts, and they come from different places:

```
220 foo.com Simple Mail Transfer Service Ready
|   |
|   +-- ours. An operator sets this in config and could set it to
|       anything. RFC 5321 section 4.2 fixes the code, not the text.
|
+-- the protocol's. RFC 5321 mandates 220 for a service greeting.
    Change it and the server stops being conformant.
```

> ⚠️ **The diagram above was redrawn in pure ASCII on 2026-08-06, and the reason is a finding.**
>
> It originally used box-drawing characters — `└ ┬ ┘ ─ │`. Those **are not in most monospace
> fonts**, so they are substituted from a fallback font at a different advance width. The Latin
> text stays monospace, the box glyphs do not match it, and the alignment collapses.
>
> | | box-drawing | pure ASCII |
> |:--|:--|:--|
> | GitHub, desktop | ✅ aligned | ✅ |
> | GitHub, Android browser | ❌ **misaligned** | ✅ |
> | Claude Android app | ❌ **misaligned** | ✅ |
>
> **This is a font problem, not a markdown one**, which is why it is device-dependent rather than
> renderer-dependent — the same page aligns on a laptop and breaks on a phone in *both* renderers.
> A markup fix cannot help; only changing the characters can.
>
> **Rule: ASCII art uses only `| - + / \ ^ _` and the printable ASCII range.** Nothing above U+007F
> inside a block where columns have to line up. The same caution applies to `…`, `—` and `·` — fine
> in prose, unsafe where alignment matters.

Rendered as one undifferentiated `code` span, those two facts look identical — and they are not.
**This is the domain/product split from `../model-altitude.md` §2.3 appearing inside a single
string**, which is exactly the distinction the model works hardest to keep visible everywhere else.

The same problem in a payload: in `claimed_domain: "bar.com"` the field name is **schema** and the
value is **an instance**. One is part of the model; the other is example data chosen to walk a path.

**Question: can inline styling carry that distinction, in both renderers, without becoming noise?**

---

## 1. What nests — the mechanical limits

Tested against GitHub's renderer before writing this page.

| # | Source | GitHub result |
|:--|:--|:--|
| 1 | `` `**bold** inside code` `` | ❌ **markdown does not nest into a code span** — asterisks render literally |
| 2 | ``**`code inside bold`**`` | ✅ `<strong><code>` |
| 3 | ``*`code inside italic`*`` | ✅ `<em><code>` |
| 4 | ``***`code in both`***`` | ✅ `<em><strong><code>` |
| 5 | `` `220` `foo.com Ready` `` | ✅ two separate spans, plain space between |
| 6 | `<code><b>220</b> foo.com</code>` | ✅ **`<b>` survives inside `<code>` as live HTML, unescaped** |

**Rule 1 is the constraint everything else works around.** Markdown cannot style *part* of a code
span. Either you break the span into pieces, or you use HTML.

**Rule 6 is the escape hatch.** GitHub keeps `<b>` inside `<code>` — verified, zero escaping. That
is the only way to bold one word inside an otherwise unbroken monospace run.

Live samples, for the second renderer:

1. `**bold** inside code`
2. **`code inside bold`**
3. *`code inside italic`*
4. ***`code in both`***
5. `220` `foo.com Ready`
6. <code><b>220</b> foo.com Simple Mail Transfer Service Ready</code>

> ⚠️ **Tested 2026-08-06 — sample 6 fails, and it takes the HTML approach with it.**
>
> | sample | GitHub | Claude app |
> |:--|:--|:--|
> | 1 `` `**bold** in code` `` | asterisks literal ✅ | asterisks literal ✅ |
> | 2 ``**`code`**`` | ✅ | ✅ |
> | 3 ``*`code`*`` | ✅ italic mono | ✅ italic mono |
> | 4 ``***`code`***`` | ✅ | ✅ |
> | 5 adjacent spans | ✅ two pills | ✅ |
> | **6 `<code><b>`** | ✅ **monospace, bold inside** | ❌ **plain serif body text — `<code>` not honored at all** |
>
> **The app does not render raw `<code>` tags.** It strips them and prints the contents as prose,
> so `<code><b>220</b> …</code>` gives monospace-with-bold on GitHub and undifferentiated body text
> on a phone. Same asymmetry class as `\text{}` in the color experiment: correct where you author,
> degraded where you read.
>
> **Consequence: you can distinguish parts of a wire line only by breaking the code span, never by
> styling inside it.** Rule 6's escape hatch is closed, and every scheme below that used it is
> dead.

---

## 2. Candidate schemes — wire lines

Each renders the same greeting. The question is which reads as *one line with two parts* rather
than as two things stuck together.

**A — ❌ DEAD. HTML bold inside one code span.** Renders as plain prose in the app.

<code>S: <b>220</b> foo.com Simple Mail Transfer Service Ready</code>

**B — broken span, bold code then italic code.**

`S:` **`220`** *`foo.com Simple Mail Transfer Service Ready`*

**C — broken span, code then plain italic.** Only the mandated part is monospace.

`S: 220` *foo.com Simple Mail Transfer Service Ready*

**D — control, what the paths use today.** No distinction at all.

`S: 220 foo.com Simple Mail Transfer Service Ready`

**E — ❌ DEAD, same reason.** A client line, verb mandated, argument the client's.

<code>C: <b>HELO</b> bar.com</code>

**F — ❌ DEAD as written**, but the *question* it poses survives and still has to be answered by whatever scheme wins.

<code>S: <b>354</b> Start mail input; end with &lt;CRLF&gt;.&lt;CRLF&gt;</code>

⚠️ **F is the case that tests the scheme rather than the rendering.** RFC 5321 §4.2 fixes `354` but
the text after it is *also* only a suggestion — so is the text config, or is it protocol? If the
scheme cannot answer that, it is decoration rather than notation.

---

## 3. Candidate schemes — payload fields

`claimed_domain: "bar.com"` — schema name versus instance value.

**G — ❌ DEAD. Bold field, plain value, one span.**

<code><b>claimed_domain</b>: "bar.com"</code>

**H — broken span, code field then italic code value.**

`claimed_domain`: *`"bar.com"`*

**I — code field, plain italic value.** Value is not literal bytes, so not monospace.

`claimed_domain`: *"bar.com"*

**J — control, what the paths use today.**

`claimed_domain: "bar.com"`

**K — ❌ DEAD. Multiple fields, the realistic case.**

<code><b>queue_id</b>: "f2C8D14" · <b>reverse_path</b>: "&lt;Smith@bar.com&gt;" · <b>received_at</b>: "1998-05-19T09:14:07-07:00"</code>

---

## 4. In a table cell, where it has to live

❌ The HTML version that stood here is removed — it renders as prose in the app. Rebuilt on the
surviving markdown-only syntax, scheme **B** for the wire and **H** for the payload:

| | **`Helo`** — C |
|:--|:--|
| ⬛ **Actor** | `C:` **`HELO`** *`bar.com`*&#10;<br>`S:` **`250`** *`foo.com`* |
| 🟦 **Command** | **Helo** |
| 🟧 **Event** | **ClientIdentified**&#10;<br>`claimed_domain`: *`"bar.com"`* |

And scheme **C**, which drops monospace for the non-mandated part entirely:

| | **`Helo`** — C |
|:--|:--|
| ⬛ **Actor** | `C: HELO` *bar.com*&#10;<br>`S: 250` *foo.com* |
| 🟦 **Command** | **Helo** |
| 🟧 **Event** | **ClientIdentified**&#10;<br>`claimed_domain`: *"bar.com"* |

Compare against the current form:

| | **`Helo`** — C |
|:--|:--|
| ⬛ **Actor** | `C: HELO bar.com`&#10;<br>`S: 250 foo.com` |
| 🟦 **Command** | **Helo** |
| 🟧 **Event** | **ClientIdentified**&#10;<br>`claimed_domain: "bar.com"` |

> ✅ **`&#10;<br>` survives alongside inline styling.** Breaks work in all three cell variants in
> both renderers. That hazard is cleared.
>
> ⚠️ **But the renderers disagree about how a broken span *reads*, and this decides the experiment.**
>
> **GitHub draws a grey background pill around every code span. The app draws none.**
>
> | | GitHub | Claude app |
> |:--|:--|:--|
> | **B** — `C:` **`HELO`** *`bar.com`* | **three separate pills** — reads as three fragments | no pills — reads as **one continuous line** |
> | **C** — `C: HELO` *bar.com* | one pill, then italic text — **two units** | mono, then italic serif — **two units** |
> | **D** — control | one pill — one unit | one run — one unit |
>
> Scheme **B** is the one that looked most promising and it is the one that diverges. Splitting a
> line into three code spans reads as one line where spans are unstyled and as three chips where
> they are pilled. The technique that best expresses *one line with two parts* in the app is the
> technique that most destroys it on GitHub.
>
> Scheme **C** degrades identically in both, because it never relies on adjacent spans looking
> continuous — it uses a font change rather than a span boundary to mark the split.
>
> **Incidental:** in scheme K, with the HTML stripped, `<Smith@bar.com>` was **auto-linked as a
> mailto address** in the app. Losing markup does not merely lose formatting; it can hand the
> content to a different parser.

---

## Verdicts

Fill in from both. A scheme needs **two ✅** and must survive a table cell.

| # | Scheme | GitHub | Claude Android | Reads as one line? | Notes |
|:--|:--|:--|:--|:--|:--|
| 1 | md nesting into code | ❌ | ❌ | — | mechanically impossible in both |
| 2 | ``**`code`**`` | ✅ | ✅ | | **live** |
| 3 | ``*`code`*`` | ✅ | ✅ | | **live** |
| 4 | ``***`code`***`` | ✅ | ✅ | | **live** |
| 5 | adjacent code spans | ✅ | ✅ | | **live** |
| 6 | `<code><b>` | ✅ | ❌ | — | **DEAD — app strips `<code>`** |
| A | HTML bold in one span | ✅ | ❌ | — | dead with 6 |
| E | HTML, client line | ✅ | ❌ | — | dead with 6 |
| F | HTML, all-mandated line | ✅ | ❌ | — | dead with 6 |
| G | HTML bold field | ✅ | ❌ | — | dead with 6 |
| K | HTML, many fields | ✅ | ❌ | — | dead with 6 |
| **B** | bold code + italic code | ⚠️ **3 pills** | ✅ one line | ❌ **diverges** | best in the app, worst on GitHub |
| **C** | code + plain italic | ✅ | ✅ | ✅ **same in both** | font change, not span boundary |
| **H** | code field + italic code value | ⚠️ 2 pills | ✅ | ⚠️ | same divergence, milder |
| **I** | code field + plain italic | ✅ | ✅ | ✅ **same in both** | C's equivalent for payloads |
| — | §4 table cell, with `&#10;<br>` | ✅ | ✅ | — | **breaks survive — hazard cleared** |

---

## What the styling would have to mean

Deciding this **before** looking at the results, so the rendering does not drive the semantics.

If a scheme is adopted, the styles must map to something the model already distinguishes —
otherwise it is two vocabularies to keep in sync:

| Style | Would mean | Tier in §2.3 |
|:--|:--|:--|
| **bold** inside monospace | fixed by the RFC — change it and conformance breaks | domain / protocol |
| plain monospace | literal bytes, but ours to choose | product |
| *italic* | example data, chosen to walk a path, not part of the model | neither — instance |

That third row is the one that earns its keep. A reader of
[`../paths/`](../paths/) has to know which values are **the model** and which are **this walk** —
`claimed_domain` is the model, `"bar.com"` is Tuesday. The paths currently rely on the reader
inferring it.

## The case against, written in advance

Same discipline as the color experiment: predict, then test.

**Three styles are near the limit of what a reader tracks.** The chips already carry element type;
adding bold-versus-plain-versus-italic inside cells means two encodings running at once.

**It is fragile in a way chips are not.** A chip is one character and either renders or does not.
`<code><b>220</b> …</code>` is nested HTML inside a table cell inside a markdown document, with
`&#10;<br>` also in play — more places to break, and the `<div>` finding from the color experiment
showed the renderers already disagree about block-level HTML.

**And the distinction may not be load-bearing.** Nothing downstream consumes it. Compare
`peer_address`, which earned its place in the model because `Received:` requires it — if no reader
decision changes based on knowing `220` is mandated and `foo.com Ready` is not, this is decoration
by the model's own **G1** test.

**The counter-argument, which is real:** the same could have been said of colored chips, and those
turned out to carry element type usefully. The test is whether a reader *does something different*
having seen the distinction — and for `354` in case F, that is genuinely unclear.
