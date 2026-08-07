# Experiment — what breaks a line inside a table cell?

Rule 5 says `&#10;<br>`, **both halves, always**. That was settled 2026-08-06 in
[`EXPERIMENT-text-color.md`](EXPERIMENT-text-color.md) and its verdict reads *"identical in both
renderers"*.

**Both.** There are three. The Claude Code desktop app was not discovered until 2026-08-07, during
[`EXPERIMENT-vertical-align.md`](EXPERIMENT-vertical-align.md), and it is now visibly **adding an
extra gap** wherever a step table stacks payload fields. The old rule is not wrong about what it
tested; it was tested against a renderer set that was one short.

⚠️ **For the avoidance of doubt: this was never a Mermaid question.** The 2026-08-06 test was run on
a markdown table and that file mentions Mermaid zero times. It has always been about markdown.

| Renderer | Version |
|:--|:--|
| **Claude Android app** | fill in at time of test |
| **Claude Code desktop app** | fill in at time of test |
| **GitHub** | web renderer. Column measured through the `POST /markdown` API **and confirmed by eye** 2026-08-07 — the API predicted all seven correctly |

---

## What GitHub does, already measured

| Token | GitHub emits | Visible break? |
|:--|:--|:--|
| `<br>` `<br/>` `<br />` | a `<br>` tag | ✅ **yes** |
| `&#10;` `&#xA;` `&NewLine;` `&#13;&#10;` | a literal newline character | ❌ **no** — HTML collapses it to a space |
| `&#10;<br>` | newline **and** tag | ✅ yes, from the tag only |
| `\` | a literal backslash | ❌ no |

**On GitHub only a `<br>` tag ever breaks a line.** Every entity form is inert there. So the entity
half of the current rule does nothing on GitHub — it is carried purely for the phone.

**Which makes the whole question one thing: does the phone break on `<br>` by itself?** If it does,
`<br>` alone works in all three and the entity should go. If it does not, no single token works
everywhere and the answer has to be structural — see the last test.

---

## The tests

Each cell contains the word **ONE**, a candidate token, then the word **TWO**. For each, report one
of: **same line** · **one break** · **two breaks or an extra gap**.

### A · `<br>` alone — *the deciding test*

| test | cell |
|:--|:--|
| A | ONE<br>TWO |

### B · `&#10;` alone — *the other half, on its own*

| test | cell |
|:--|:--|
| B | ONE&#10;TWO |

### C · `&#10;<br>` — the current rule

| test | cell |
|:--|:--|
| C | ONE&#10;<br>TWO |

### D · `<br/>` self-closing

| test | cell |
|:--|:--|
| D | ONE<br/>TWO |

### E · `<br>&#10;` — same two halves, reversed

| test | cell |
|:--|:--|
| E | ONE<br>&#10;TWO |

### F · `&#xA;` hex entity

| test | cell |
|:--|:--|
| F | ONE&#xA;TWO |

### G · `&NewLine;` named entity

| test | cell |
|:--|:--|
| G | ONE&NewLine;TWO |

---

## H · Three lines, the way a real payload looks

The failure only matters at scale, so this is the shape it actually takes in a step. Compare against
test J below.

| 🟦 C · Step 1 | `AcceptConnection` |
|:--|:--|
| Event | 🟧 **ConnectionAccepted**&#10;<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20&#10;<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |

## I · The same, with `<br>` only

| 🟦 C · Step 1 | `AcceptConnection` |
|:--|:--|
| Event | 🟧 **ConnectionAccepted**<br>&nbsp;&nbsp;`peer_address`: 203.0.113.20<br>&nbsp;&nbsp;`local_address`: 192.0.2.10:25 |

## J · The structural answer — no line breaks at all

One row per line. **No token, so nothing can render it differently.** This cannot fail in any
renderer, present or future.

| 🟦 C · Step 1 | `AcceptConnection` |
|:--|:--|
| Event | 🟧 **ConnectionAccepted** |
| | &nbsp;&nbsp;`peer_address`: 203.0.113.20 |
| | &nbsp;&nbsp;`local_address`: 192.0.2.10:25 |

---

## Verdicts

Fill in from all three. A token needs **one break in every column** to be usable.

| | Token | GitHub | Claude Android | Claude Code desktop | Usable |
|:--|:--|:--|:--|:--|:--|
| A | `<br>` | ✅ one | ❌ **none — words FUSE** | ✅ one | ❌ |
| B | `&#10;` | ❌ none | ✅ one | ✅ one | ❌ |
| C | `&#10;<br>` | ✅ one | ✅ one | ⚠️ **two — extra gap** | ❌ |
| D | `<br/>` | ✅ one | ❌ **none — words FUSE** | ✅ one | ❌ |
| E | `<br>&#10;` | ✅ one | ✅ one | ⚠️ **two — extra gap** | ❌ |
| F | `&#xA;` | ❌ none | ✅ one | ✅ one | ❌ |
| G | `&NewLine;` | ❌ none | ✅ one | ✅ one | ❌ |
| J | **separate rows** | ✅ | ✅ | ✅ | ✅ **the only one** |

---

# ✅ CLOSED 2026-08-07 — **no token works. Use rows.**

**Every single-token candidate fails somewhere, and the failure is structural rather than a matter of
picking a better token.** The three renderers demand mutually exclusive things:

| Renderer | Needs | Ignores |
|:--|:--|:--|
| **GitHub** | a **tag** | entities — they emit a literal newline, which HTML collapses to a space |
| **Claude Android** | an **entity** | tags — stripped, and the words **fuse silently** |
| **Claude Code desktop** | **either** | nothing — it honors **both**, so any pairing renders **twice** |

GitHub and the phone need opposite things, so a pairing is forced. The desktop honors both halves of
that pairing, so the pairing is guaranteed to double there. **There is no arrangement of these two
tokens that yields one break in all three** — and reversing the order does not help, because E gaps
exactly as C does.

⚠️ **The 2026-08-06 rule was not wrong. It was optimal for the two renderers it knew about**, and it
remains the best *token* available. It is retired because a token is the wrong instrument, not
because a better token was found.

**The answer is J: one row per line.** It uses no token, so there is nothing for a renderer to
interpret differently — this cannot break in any renderer, present or future, which no token-based
answer can promise.

### What this costs

- **Every stacked payload becomes rows.** A three-field event is four rows, not one.
- **Multi-line wire rows become rows.** `helo-direct-single-recipient.md` Step 9 has eight wire lines.
- **Step tables stop being a fixed height**, which was V16's whole argument.
- ⚠️ **V16 in [`../paths/EXPLORE-gwt-form.md`](../paths/EXPLORE-gwt-form.md) is dead.** It stacks given
  events inside one cell. **V17 survives unchanged** — it was already one row per event, and it is now
  the only one of the pair that can be rendered.

The indent (`&nbsp;&nbsp;`) still works and still distinguishes a field line from its event name.

## Prediction, written before the phone was looked at — ❌ **WRONG**

Kept unedited below, per the practice in [`EXPERIMENT-text-color.md`](EXPERIMENT-text-color.md).

⚠️ **It predicted A would pass. A failed, and fused the words.** The evidence cited was real — the
Android app *was* seen honoring a bare `<br>` on 2026-08-07 — but that sighting was inside a **raw
HTML block**, and this test is a **markdown table cell**. The prediction names that difference in its
own second sentence, calls it *"a different context, so it does not settle this"*, and then predicts
as though it did.

**Flagging a caveat is not the same as applying it.** The wrong half of the reasoning was written
down, correctly labeled, and used anyway.

**A will pass on the phone, and `<br>` alone will be the answer.** The reasoning is that the Android
app was seen honoring a bare `<br>` on 2026-08-07, inside the raw HTML block of variant C in the
vertical-align experiment — `peer_address` and `local_address` landed on separate lines there with no
entity present. That was in a different context, so it does not settle this, but it is evidence the
app understands the tag. If it does, the entity has been redundant on the phone as well as inert on
GitHub, and it is doing nothing anywhere except adding a gap on the desktop.

**If A fails on the phone, no token works everywhere** and J is the only correct answer. That would
be a larger change than it looks: every step's payload fields become rows, every wire line becomes a
row, and the step table stops being four rows tall. It would also settle the open V16-against-V17
question in [`../paths/EXPLORE-gwt-form.md`](../paths/EXPLORE-gwt-form.md) by force, since V16
depends on stacking inside a cell.

**The uncomfortable possibility is that A passes and C also looks fine on the phone**, because the
phone may be collapsing whatever it does not use. Then the rule was never wrong on the phone and the
desktop is simply stricter — and the fix is still to drop the entity, but the old verdict stands as
written for the renderers it named.
