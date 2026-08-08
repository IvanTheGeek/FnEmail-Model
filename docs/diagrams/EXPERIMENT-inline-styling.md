# Experiment — inline styling to carry meaning inside a string

> ## ✅ CLOSED 2026-08-06 — adopted: **mono for fixed, standard for variable**
>
> | | | |
> |:--|:--|:--|
> | **Wire line** | **W2** | `S: 220` foo.com Simple Mail Transfer Service Ready |
> | **Payload field** | **P2** | `claimed_domain`: bar.com |
>
> **Monospace marks what the protocol fixes** — reply codes, verbs, field names. **Standard font
> marks what varies** — operator config, client arguments, instance values.
>
> Three reasons it won, over twenty-nine other combinations:
>
> - **The required parts stay visible and distinguishable**, which is what monospace is for.
> - **The variable parts become easier to read**, because a proportional font is easier to read, and
>   those are the parts a reader actually scans.
> - **Values are easy to select.** A value in standard font outside a code span is ordinary text to
>   a reader and to a mouse.
>
> It uses **one axis** — font family — and never touches weight or slant, so it sits clear of the
> two techniques that failed: bold does not apply to monospace in the app, and italic was unwanted
> on values. **Values carry no quotation marks.**
>
> Everything below is the working that led here, kept per rule 4.


Third in the series, after `EXPERIMENT-block-labels.md` (Mermaid label syntax, superseded and
deleted 2026-08-08 with the rest of the Mermaid era — in git history, last present at commit
d22315e) and [`EXPERIMENT-text-color.md`](EXPERIMENT-text-color.md) (color, closed — chips won).
Same method: try every candidate, open it in **both** renderers, keep only what works in both.

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

> 🛑 **Bigger finding, 2026-08-06 — the Claude app does not apply bold to monospace at all.**
>
> The proof is above and was missed on the first reading. **Samples 3 and 4 render identically** in
> the app. Sample 3 is ``*`code`*``, italic only. Sample 4 is ``***`code`***``, bold *and* italic.
> If bold applied, they would differ. It is dropped. Sample 2, ``**`code`**``, is likewise the same
> weight as plain sample 1.
>
> | | GitHub | Claude app |
> |:--|:--|:--|
> | bold on a code span | ✅ | ❌ **ignored** |
> | italic on a code span | ✅ | ✅ |
> | bold on standard text | ✅ | ✅ |
>
> **Both routes to bold-inside-monospace are now closed.** `<code><b>` fails because the app strips
> `<code>`; ``**`code`**`` fails because the app drops the weight. There is no third route.
>
> ⚠️ **This was under-called once already.** An earlier note read L's rendering as a *weak* signal —
> "nearly invisible in the app". It is not weak, it is **absent**, and the difference matters: a
> weak signal is a design trade-off, an absent one is a dead technique. Reported from desktop as
> well as phone, so this is the app's renderer generally, not the mobile font fallback that caused
> the box-drawing failure.
>
> **Three surviving axes**, and only three: **font family** (mono vs standard), **italic** (on
> either), and **bold on standard text only**.

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
> Scheme **B** splits the line into three code spans, which reads as one line where spans are
> unstyled and as three chips where they are pilled.
>
> ⚠️ **Downgraded on review 2026-08-06 — this is a note, not a disqualifier.** An earlier version of
> this block treated the pill divergence as fatal to B. The reviewer's position is that the
> background difference is an acceptable cost, so **B stays live** and the pill is recorded as a
> characteristic rather than a failure. Scheme **C** still has the property that it never relies on
> adjacent spans looking continuous, which is worth knowing — it just is not the deciding factor it
> was written up as.
>
> **Incidental:** in scheme K, with the HTML stripped, `<Smith@bar.com>` was **auto-linked as a
> mailto address** in the app. Losing markup does not merely lose formatting; it can hand the
> content to a different parser.

---

## 5. The full matrix

The schemes above were picked before the option space was mapped. This is the map.

**Three orthogonal axes**, plus modifiers that ride on top of any cell:

| | upright | **bold** | *italic* | ***bold italic*** |
|:--|:--|:--|:--|:--|
| **standard** | 220 foo.com Ready | **220 foo.com Ready** | *220 foo.com Ready* | ***220 foo.com Ready*** |
| **mono** | `220 foo.com Ready` | **`220 foo.com Ready`** | *`220 foo.com Ready`* | ***`220 foo.com Ready`*** |

Eight base styles. Source, in order:

| | source |
|:--|:--|
| standard | `220 foo.com Ready` — no markup |
| standard bold | `**220 foo.com Ready**` |
| standard italic | `*220 foo.com Ready*` |
| standard bold italic | `***220 foo.com Ready***` |
| mono | ``` `220 foo.com Ready` ``` |
| mono bold | ``` **`220 foo.com Ready`** ``` |
| mono italic | ``` *`220 foo.com Ready`* ``` |
| mono bold italic | ``` ***`220 foo.com Ready`*** ``` |

### Modifiers that combine with any of the eight

| Modifier | Sample | Source | Note |
|:--|:--|:--|:--|
| **ALL CAPS** | `220 FOO.COM READY` | content change, not markup | Free, survives copy-paste. ⚠️ **Corrected:** an earlier note here said "unsafe for domains and paths". Wrong — RFC 5321 §2.4 puts verbs *and domains* under case-insensitive rules, so caps is safe for both. It is **mailbox local-parts** that MUST preserve case, so `<Smith@bar.com>` may never be capitalized as a notation choice. See `../event-model.md` → *Case sensitivity* |
| **~~strikethrough~~** | ~~`220 foo.com Ready`~~ | `~~...~~` | Available and unused. Reads as *removed* or *superseded*, so it carries a meaning already — hard to repurpose |
| **quotes as delimiter** | `"bar.com"` vs `bar.com` | content | Already in use for values. Zero markup cost, and the quotes are themselves the signal |
| **angle brackets** | `<Smith@bar.com>` | content | RFC 5321's own delimiter for a path. Free, and *already meaningful* to the domain |
| **separator choice** | `field: value` · `field = value` · `field · value` | content | Distinguishes without styling anything |

### Two more the list did not include

**Links are colored in both renderers**, and are the only remaining route to color that is not
markup a sanitizer strips: [220 foo.com Ready](#). ⚠️ **Do not adopt** — it is semantic abuse, the
target has to go somewhere, and a reader will tap it. Recorded so the option is closed rather than
rediscovered.

**Emoji chips can sit inline**, not only in a row label: 🟥 `550 No such user here`. Already the
project's color mechanism, already proven in both renderers, and it adds a *distinct* channel
rather than competing with the font axes. Currently unused inside a wire line.

---

## 6. Schemes without italic on values

Constraint from review: *italic is a strong differentiator but tiring in quantity, and unwanted on
the values a reader looks at most.* That rules out **B**, **C**, **H** and **I** as written, since
all four put the value in italic.

The remaining lever is **bold**, and it inverts cleanly. Rather than marking the *variable* part,
mark the *fixed* part — the thing the model defines:

**L — bold names the schema, plain is the instance.** One font, one axis, no italic.

| | |
|:--|:--|
| Event title | **ClientIdentified** |
| Field and value | **`claimed_domain`**`: "bar.com"` |
| Wire line | **`220`**` foo.com Simple Mail Transfer Service Ready` |

**M — as L, but the value drops to standard font.** Monospace then means *literal protocol bytes*,
so the value stops claiming to be one.

| | |
|:--|:--|
| Field and value | **`claimed_domain`**: "bar.com" |
| Wire line | **`220`** foo.com Simple Mail Transfer Service Ready |

**N — ⚠️ HALF DEAD. ALL CAPS carries the protocol, bold carries the schema.**

| | |
|:--|:--|
| Wire line | `C: HELO bar.com` — ~~verb already caps by protocol~~ |
| Reply | `S: 220 foo.com Simple Mail Transfer Service Ready` |
| Field and value | **`claimed_domain`**`: "bar.com"` |

> ⚠️ **Corrected 2026-08-06 — the premise was half wrong, and the wrong half was the dangerous
> kind.**
>
> This scheme was justified as *"nearly free, because SMTP verbs are already uppercase and reply
> codes are already digits — the protocol has done the differentiating for us."* Only the second
> clause is true.
>
> **Reply codes are genuinely self-differentiating.** RFC 5321 §4.2 requires a three-digit numeric
> code, so `220` is structurally distinct from the free-form text after it with no styling at all.
> That half stands.
>
> **Verbs are not uppercase by the protocol.** §2.4 says a command verb *"MAY be encoded in upper
> case, lower case, or any mixture … with no impact on its meaning"*, and names servers that demand
> uppercase as being **in violation**. `helo bar.com` is a valid greeting. See `../event-model.md`
> → *Case sensitivity*.
>
> So the uppercase in every path here is the RFC's **example convention**, faithfully reproduced
> per rule 2, and not a rule the protocol enforces.
>
> **This makes case worse than unavailable as notation — it makes it misleading.** Styling by case
> would encode a distinction the specification explicitly says does not exist, and a reader who
> learned the notation would have learned something false about SMTP. A scheme that is merely
> unusable costs nothing; one that teaches a falsehood is a defect.
>
> **What survives:** the reply-code half. The digits already separate themselves from the text, so
> a wire line may need *less* notation than assumed — but the argument now covers replies only, not
> commands.

### Noise budget

Ranked by how many style changes appear in one line, since the stated goal is the fewest that still
distinguish:

| Scheme | Style changes per line | Value in italic? |
|:--|--:|:--|
| **D** control | 0 | no |
| **N** caps + bold field | 1 | no |
| **M** bold mono + standard | 2 | no |
| **L** bold mono + plain mono | 2 | no |
| **C** / **I** | 2 | **yes** |
| **B** / **H** | 3 | **yes** |

---

---

## 7. Every working variant, side by side

Rebuilt 2026-08-06 after two findings: **bold does not apply to monospace in the Claude app**, and
**values are not quoted** — see the note below. Everything here renders in both renderers. Nothing
is ruled out on taste; pick by eye.

**Three surviving axes:** font family (mono vs standard), *italic* on either, and **bold on
standard text only.**

> ⚠️ **Values carry no quotation marks.** An earlier version wrote `bar.com` as `"bar.com"`, and the
> quotes were even listed in §5 as a free delimiter that distinguishes without styling. Rejected on
> review: quotes are not how this project writes a value, and there is no current case that needs
> them. That removes a channel §5 was counting on, so the **styling has to do the work the quotes
> were doing** — which raises the stakes on this section rather than lowering them.
>
> Angle brackets stay where the RFC uses them: `<Smith@bar.com>` is a reverse-path in RFC 5321's own
> syntax, not a quoting convention of ours.

### The complete cross — every live combination

Both parts vary independently, so the map is a grid rather than a list. **Six live styles per part**
— mono-bold is absent because the app drops it, which is the only exclusion here.

Short example so the grid fits: fixed `220`, variable `Ready`. **Rows are the fixed part, columns
the variable part.** The diagonal is both parts in one style — a single continuous run, which is
what the control looks like, so it is shown rather than blanked.

| fixed \ variable → | mono | mono *ital* | std | std **bold** | std *ital* | std ***b+i*** |
|:--|:--|:--|:--|:--|:--|:--|
| **mono** | `220 Ready` | `220` *`Ready`* | `220` Ready | `220` **Ready** | `220` *Ready* | `220` ***Ready*** |
| **mono *ital*** | *`220`* `Ready` | *`220 Ready`* | *`220`* Ready | *`220`* **Ready** | *`220`* *Ready* | *`220`* ***Ready*** |
| **std** | 220 `Ready` | 220 *`Ready`* | 220 Ready | 220 **Ready** | 220 *Ready* | 220 ***Ready*** |
| **std **bold**** | **220** `Ready` | **220** *`Ready`* | **220** Ready | **220 Ready** | **220** *Ready* | **220** ***Ready*** |
| **std *ital*** | *220* `Ready` | *220* *`Ready`* | *220* Ready | *220* **Ready** | *220 Ready* | *220* ***Ready*** |
| **std ***b+i***** | ***220*** `Ready` | ***220*** *`Ready`* | ***220*** Ready | ***220*** **Ready** | ***220*** *Ready* | ***220 Ready*** |

**Thirty distinguishing combinations, plus six controls on the diagonal.** The earlier W-list sampled only the first two rows, which is
what prompted this grid — the fixed part had never been varied outside monospace except in W7.

⚠️ **The bottom four rows are the ones nobody had looked at.** Putting the *fixed* part in standard
font and the *variable* part in monospace inverts the usual reading, and it is not obviously wrong:
the value that came off the wire is the literal byte-string, and `220` is arguably a label for a
meaning rather than bytes to be reproduced.

### Wire line — the shortlist, at real length

The grid above is exhaustive; these are the ones worth reading in full-length form.

### Payload field — the complete cross

Same grid, applied to a field. **Rows are the field name, columns the value.** No quotes.

| name \ value → | mono | mono *ital* | std | std **bold** | std *ital* | std ***b+i*** |
|:--|:--|:--|:--|:--|:--|:--|
| **mono** | `claimed_domain: bar.com` | `claimed_domain`: *`bar.com`* | `claimed_domain`: bar.com | `claimed_domain`: **bar.com** | `claimed_domain`: *bar.com* | `claimed_domain`: ***bar.com*** |
| **mono *ital*** | *`claimed_domain`*: `bar.com` | *`claimed_domain: bar.com`* | *`claimed_domain`*: bar.com | *`claimed_domain`*: **bar.com** | *`claimed_domain`*: *bar.com* | *`claimed_domain`*: ***bar.com*** |
| **std** | claimed_domain: `bar.com` | claimed_domain: *`bar.com`* | claimed_domain: bar.com | claimed_domain: **bar.com** | claimed_domain: *bar.com* | claimed_domain: ***bar.com*** |
| **std **bold**** | **claimed_domain**: `bar.com` | **claimed_domain**: *`bar.com`* | **claimed_domain**: bar.com | **claimed_domain: bar.com** | **claimed_domain**: *bar.com* | **claimed_domain**: ***bar.com*** |
| **std *ital*** | *claimed_domain*: `bar.com` | *claimed_domain*: *`bar.com`* | *claimed_domain*: bar.com | *claimed_domain*: **bar.com** | *claimed_domain: bar.com* | *claimed_domain*: ***bar.com*** |
| **std ***b+i***** | ***claimed_domain***: `bar.com` | ***claimed_domain***: *`bar.com`* | ***claimed_domain***: bar.com | ***claimed_domain***: **bar.com** | ***claimed_domain***: *bar.com* | ***claimed_domain: bar.com*** |

⚠️ **The lower-left quadrant is the inversion.** Standard-font name with a monospace value says
*the value is the literal thing and the name is our label* — which with quotes gone is also the only
way left to show where a value begins and ends.

### Payload field — the shortlist, in prose form

### The realistic case — three fields, no quotes

Noise is only judgeable at length.

**P2** — `queue_id`: f2C8D14 · `reverse_path`: <Smith@bar.com> · `received_at`: 1998-05-19T09:14:07-07:00

**P4** — `queue_id`: **f2C8D14** · `reverse_path`: **<Smith@bar.com>** · `received_at`: **1998-05-19T09:14:07-07:00**

**P5** — `queue_id`: *`f2C8D14`* · `reverse_path`: *`<Smith@bar.com>`* · `received_at`: *`1998-05-19T09:14:07-07:00`*

**P7** — **queue_id**: `f2C8D14` · **reverse_path**: `<Smith@bar.com>` · **received_at**: `1998-05-19T09:14:07-07:00`

**P1 control** — `queue_id: f2C8D14` · `reverse_path: <Smith@bar.com>` · `received_at: 1998-05-19T09:14:07-07:00`

⚠️ **At three fields the control gets genuinely hard to scan**, which is the argument for doing
anything at all. It is one undifferentiated monospace run where every boundary — between fields,
and between each name and its value — is carried by punctuation alone.

### In a cell

| | **`Helo`** — C |
|:--|:--|
| ⬛ **Actor** | `220 foo.com Ready` — W1&#10;<br>`220` foo.com Ready — W2&#10;<br>`220` *foo.com Ready* — W3&#10;<br>**220** `foo.com Ready` — W7 |
| 🟧 **Event** | `claimed_domain: bar.com` — P1&#10;<br>`claimed_domain`: bar.com — P2&#10;<br>`claimed_domain`: *`bar.com`* — P5&#10;<br>**claimed_domain**: `bar.com` — P7 |

---

## Verdicts

Fill in from both. A scheme needs **two ✅** and must survive a table cell.

| # | Scheme | GitHub | Claude app | In a cell | Note |
|:--|:--|:--|:--|:--|:--|
| 1 | md nesting into code | ❌ | ❌ | — | mechanically impossible in both |
| 6 | `<code><b>` | ✅ | ❌ | — | **DEAD** — app strips `<code>` |
| A · E · F · G · K | all HTML schemes | ✅ | ❌ | — | **DEAD** with 6 |
| 2 | ``**`code`**`` | ✅ | ✅ | ✅ | live primitive |
| 3 | ``*`code`*`` | ✅ | ✅ | ✅ | live primitive |
| 4 | ``***`code`***`` | ✅ | ✅ | ✅ | live primitive |
| 5 | adjacent code spans | ✅ | ✅ | ✅ | live primitive · GitHub pills each one |
| ~~**B**~~ | mono-bold + mono-italic | ✅ | ❌ | — | 🛑 **DEAD** — app drops bold on mono |
| **C** | mono + standard-italic | ✅ | ✅ | ✅ | **LIVE** · same in both |
| **D** | control, all mono | ✅ | ✅ | ✅ | **LIVE** · baseline |
| **H** | mono field + mono-italic value | ✅ | ✅ | ✅ | **LIVE** · 2 pills on GitHub |
| **I** | mono field + standard-italic value | ✅ | ✅ | ✅ | **LIVE** · same in both |
| **J** | control, fields all mono | ✅ | ✅ | ✅ | **LIVE** · baseline |
| ~~**L**~~ | mono-bold schema + mono instance | ✅ | ❌ | — | 🛑 **DEAD** — the bold is not weak, it is absent |
| ~~**M**~~ | mono-bold schema + standard instance | ⚠️ | ⚠️ | — | 🛑 **DEAD as written** — the bold is lost in the app. The *font-family* half survives as **V6/F6** |
| **N** | case carries the protocol | — | — | — | ❌ **DEAD on correctness**, not rendering — verbs are not uppercase by protocol |
| — | §4 cell with `&#10;<br>` | ✅ | ✅ | ✅ | breaks survive alongside styling |

**Four things are dead now**, and none died on taste:

1. **HTML** — the app strips `<code>`
2. **Markdown nesting into a code span** — mechanically impossible
3. **N** — its premise about SMTP verbs was factually false
4. 🛑 **Bold on monospace** — the app ignores the weight. This one arrived last and took **B, L and
   M** with it, which were the three leading candidates.

What is left is a smaller and clearer field: **font family, italic, and bold on standard text
only.** Every surviving scheme uses one or two of those three.

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
