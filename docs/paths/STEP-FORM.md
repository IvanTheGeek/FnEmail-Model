# The step form

What one step of a walked path looks like. Every path in this directory uses it, so a step reads
the same whichever path you are in.

A **path** is the model instantiated with real data. A **step** is one slice of that model, at one
moment, with the actual bytes that crossed the wire and the actual field values that resulted.

---

## The form

A step is a heading, a slice table, and a contract.

```
### Step N · `SliceName` — C
```

- **N** is the position in *this walk*. Unlike slice numbers it is not arbitrary — a walk is one
  concrete conversation, and step 4 genuinely happened after step 3. See `../HANDOFF.md`
  convention 8 for why slices themselves are named rather than numbered.
- **`SliceName`** matches the heading in `../event-model.md` exactly.
- **C** or **V** — Command Slice or View Slice. Same marker the model uses.

Then the slice, in the convention from [`../diagrams/README.md`](../diagrams/README.md): rows are
element types, chips carry the type, `&#10;<br>` breaks a line.

| | **`Helo`** — C |
|:--|:--|
| ⬜ **Screen** | `C: HELO bar.com`&#10;<br>`S: 250 foo.com` |
| 🟦 **Command** | **Helo** |
| 🟧 **Event** | **ClientIdentified**&#10;<br>`claimed_domain: "bar.com"` · `protocol: "SMTP"` |

Then the contract, which is what makes it a *walk* rather than a picture:

> **Pre** — `ConnectionAccepted` exists ✓
> **Post** — `ClientIdentified` emitted ✓

---

## Rules

**The Screen row holds the wire, verbatim.** `C:` is the client, `S:` is us. Reply codes are exact,
including the text after them. This is the row a reader follows to reconstruct the conversation —
which is why this project needs no sequence diagram (see `../diagrams/README.md` §7).

**A View Slice has no wire.** Nothing crosses the network to build a read model. Its Screen row
names the **consumer** instead, and its bottom row names the **source events** rather than an
emitted one. A View Slice reads bottom to top.

| | **`SessionState`** — V |
|:--|:--|
| ⬜ **Consumed by** | `MailFrom` · `RcptTo` · `BeginData` |
| 🟩 **Read Model** | **SessionState**&#10;<br>`identified: true` · `transaction_open: false` |
| 🟧 **Sources** | **ConnectionAccepted** · **ClientIdentified** |

**Every field carries a real value.** Not `<address>` but `"198.51.100.40"`. The value is the point
— walking with placeholders finds nothing, and both payload defects this project has found came
from instantiating a field and seeing that nothing consumed it, or that its value could not survive
replay.

**Pre and Post are checked, not asserted.** A ✓ means the walk verified it against the slice
contract in `../event-model.md`. If a precondition cannot be satisfied, the walk stops there and
that is a finding.

**Hotspots travel with the step.** A slice under an open question carries its marker inline —
`🟥 H1` — so a reader walking the path meets the doubt at the point where it matters rather than in
an appendix.

---

## Why the wire is a code span and not prose

Reply codes and the `<>` around a reverse-path are protocol syntax; losing a bracket or a space
changes their meaning. Code spans render monospace on GitHub and in the Android app, and they
survive copy-paste into a terminal. `MAIL FROM:<Smith@bar.com>` is testable; *"a MAIL FROM command
naming Smith"* is not.

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
