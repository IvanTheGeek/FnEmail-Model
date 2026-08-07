# The step form

What one step of a walked path looks like. Every path in this directory uses it, so a step reads
the same whichever path you are in.

A **path** is the model instantiated with real data. A **step** is one slice of that model, at one
moment, with the actual bytes that crossed the wire and the actual field values that resulted.

---

## The form — a Command Slice

Everything lives in one table. There is no heading above it and no contract below it; the header row
carries the identity and the last row carries the dependency.

| 🟦 C · Step 2 | `Helo` |
|:--|:--|
| MTA Client | ⬛ `HELO` bar.com&#10;<br>`250` foo.com |
| | 🟦 **Helo**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Event | 🟧 **ClientIdentified**&#10;<br>&nbsp;&nbsp;`claimed_domain`: bar.com&#10;<br>&nbsp;&nbsp;`protocol`: SMTP |
| Given | 🟧 **ConnectionAccepted** |

**The header row** is `🟦 C · Step N` and the slice name. **N** is the position in *this walk* —
unlike a slice number it is not arbitrary, because a walk is one concrete conversation and step 4
genuinely happened after step 3. The name matches `../event-model.md` exactly. See `../HANDOFF.md`
convention 8 for why slices are named rather than numbered.

**The left column names the participant, not the element type.** The wire row is the actor, the
event row is `Event`, the dependency row is `Given` — and **the command row's label is blank**. That
blank is deliberate: the middle row *is the slice itself*, which the header named one line above, so
labeling it would state the name a third time.

⚠️ **The command carries the fields it takes from the actor.** A bare command name breaks the
completeness check — every field needs an origin, and the command is the origin for everything the
client supplies. Where the command and event rows look nearly identical, that is the check
**passing**: a value arriving and being stored unchanged. Where they diverge, something is derived,
generated or dropped, and that is visible at a glance.

⚠️ **`Given` is last, and it is the only row that can be wrong.** The command and event rows are
*observations* of what the walk did. A `Given` is a **claim** about what the step depends on.

## The `Given` row

**One event per row.** The label is carried by the first row only; continuation rows leave it blank,
the same idiom the command row uses.

| Degree | Means | Written as |
|:--|:--|:--|
| **nothing** | this step depends on no previous event | ⚪ |
| **the event exists** | it must have happened; nothing about its contents | 🟧 **EventName** |
| **the event and its data** | specific fields must match | **the Event row's own layout** |

**It is the minimal dependency, not the accumulated history.** Dymitruk's convention is that a given
is every previous row, so a test runner *"can always just use an accumulator to add events"*. That
exists for a runner, which has no walk to read. Here the walk is on the page, so restating it would
be redundant — `RcptTo` has three events above it and depends on one. **A deliberate departure.**

**A dependency with no event of ours is still written down**, marked 🟨 and stated as external. That
is what keeps a translation boundary visible at the step where it bites rather than hidden behind a
read-model name.

⚠️ **The degree-0 marker is provisional.** ⚪ is in use; alternatives are under test in
[`EXPLORE-gwt-form.md`](EXPLORE-gwt-form.md). It also marks a command that supplies no fields —
`BeginData` and `Quit` are bare verbs — so one marker covers *nothing required* and *nothing
supplied*.

---

## A View Slice is different, and is being reworked

**A View Slice has no wire.** Nothing crosses the network to build a read model. Its top row names
the **consumer**, its bottom row names the **source events**, and it reads bottom to top.

| 🟩 V · Step 3 | `SessionState` |
|:--|:--|
| Consumed by | ⬜ `MailFrom` · `RcptTo` · `BeginData` |
| | 🟩 **SessionState**&#10;<br>&nbsp;&nbsp;`identified`: true&#10;<br>&nbsp;&nbsp;`transaction_open`: false |
| Sources | 🟧 **ConnectionAccepted** · **ClientIdentified** |

⚠️ **This form predates the Command Slice rework and has not been brought onto it.** It has no
`Given` row and its rows are still ordered consumer-first. Deliberately left alone — see
[`EXPLORE-view-slice.md`](EXPLORE-view-slice.md). **Do not pattern it on the Command Slice without
that work being done**, because the corpus disagrees with itself about whether a view even has a
`When`.

---

## Rules

**Uppercase verbs are the RFC's example convention, not a requirement.** Every path here writes
`HELO`, `MAIL FROM`, `RCPT TO` in caps because RFC 5321 Appendix D does, and rule 2 keeps a quoted
dialogue in its own form. But §2.4 says a verb *"MAY be encoded in upper case, lower case, or any
mixture … with no impact on its meaning"*. Do not read the caps in these paths as normative, and
never use case to carry meaning — see `../event-model.md` → *Case sensitivity*.

**The wire row holds the wire, verbatim.** Reply codes are exact, including the text after them.
This is the row a reader follows to reconstruct the conversation — which is why this project needs
no sequence diagram (see `../diagrams/README.md` §7). The `C:` and `S:` prefixes are **not** used:
in SMTP a verb is always the client and a three-digit code is always the server, so direction is
recoverable from the content. Step 9 is the exception, where the DATA payload is neither, and there
the RFC 5322 field names carry the monospace instead.

**Monospace marks what the protocol fixes; standard font marks what varies.** On a wire line the
reply code and the verb are monospace and the rest is not — `220` foo.com Ready. In a payload the
field name is monospace and the value is not — `peer_address`: 198.51.100.40. **Values carry no
quotation marks.** One axis only, font family; never bold or italic. Settled in
[`../diagrams/EXPERIMENT-inline-styling.md`](../diagrams/EXPERIMENT-inline-styling.md).

**Every field carries a real value.** Not `<address>` but 198.51.100.40. The value is the point
— walking with placeholders finds nothing, and both payload defects this project has found came
from instantiating a field and seeing that nothing consumed it, or that its value could not survive
replay.

**A step carries only its own walk's scenarios.** Not the 5–20 a command handler may have. Those
accumulate at the *slice*, which is the union of what every walk contributed — see `../HANDOFF.md`
§1, *Paths are the source; slices are derived*.

**Hotspots travel with the step.** A slice under an open question carries its marker inline —
`🟥 H1` — so a reader walking the path meets the doubt at the point where it matters rather than in
an appendix.

⚠️ **🟥 is currently overloaded**, meaning both *hotspot* and *error outcome*. Unresolved; see
[`EXPLORE-gwt-form.md`](EXPLORE-gwt-form.md).

---

## Why the wire is a code span and not prose

Reply codes and the `<>` around a reverse-path are protocol syntax; losing a bracket or a space
changes their meaning. Code spans render monospace on GitHub and in the Android app, and they
survive copy-paste into a terminal. `MAIL FROM:<Smith@bar.com>` is testable; *"a MAIL FROM command
naming Smith"* is not.

⚠️ **An address outside a code span loses its angle brackets** — GFM autolinks it and eats them, and
`<CRLF>` in running text vanishes entirely. Escape the opening bracket: `\<Smith@bar.com>`. The
address still autolinks, which is accepted; the brackets are RFC 5321 path syntax and are not.

---

## What a path needs besides steps

The walk is the middle of the document, not all of it:

| Section | Does |
|:--|:--|
| **Scene** | the actors, addresses and time, as a table. Anything deliberate about the setup is called out here |
| **The walk** | the steps |
| **Accounting** | steps vs distinct slices touched, and what remains uncovered across all paths |
| **Completeness, instantiated** | the origin-and-destination check run against real values — usually the `Received:` header, since it is where the model's fields become output |
| **What this walk tested** | what it proved. Negative results count and are often the valuable kind |
| **What it did not test** | stated plainly. A clean path confirms; it does not discover |

The last two matter more than they look. A path that found nothing should say so — that is a fact
about the path, and it is how this project learned to walk the messy path first.
