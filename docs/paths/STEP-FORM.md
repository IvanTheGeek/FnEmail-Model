# The step form — the SMTP layer

What a walked step in this repository adds to the generic form.

The generic form is the method repo's:
[`path-and-step-form.md`](https://github.com/IvanTheGeek/EventModeling/blob/main/docs/path-and-step-form.md)
— the step table and its rows, the `Given` block and its three degrees, what a view is, what a path
document contains besides steps. This file does not restate any of it (rule 11). Rendering
conventions are [`../../AGENTS.md`](../../AGENTS.md) rule 5; the chip legend is
[`../diagrams/README.md`](../diagrams/README.md).

The worked instance is
[`WORKING-helo-direct-single-recipient-v2.md`](WORKING-helo-direct-single-recipient-v2.md).
Pattern new steps on it — and note that it still carries its `WORKING-` prefix, so its details may
move (rule 13).

---

## Every server reply is a step of its own

A reply is a **rendered view**: the wire line is what the renderer produces from the view's fields.
So each of the seven replies in a walk gets its own view step, and none of them appear as text
inside the wire row of the command that provoked them. A view read only internally —
`RecipientDirectory`, which `RcptTo` consults and which is rendered to nobody — has no top row at
all, and its readers declare it in their own `Given`.

The eighth output is not a reply and still gets a step: the `Received:` trace line, drawn into the
stored message rather than the socket, which is why its top row carries no wire chip.

## Uppercase verbs are the RFC's example convention, not a requirement

Every path here writes `HELO`, `MAIL FROM`, `RCPT TO` in caps because RFC 5321 Appendix D does, and
rule 2 keeps a quoted dialogue in its own form. But §2.4 says a verb *"MAY be encoded in upper
case, lower case, or any mixture … with no impact on its meaning"*. Do not read the caps in these
paths as normative, and never use case to carry meaning — see
[`../event-model.md`](../event-model.md) → *Case sensitivity*.

## The wire row holds the wire, verbatim

Reply codes are exact, including the text after them. This is the row a reader follows to
reconstruct the conversation, which is why this project needs no sequence diagram
([`../diagrams/README.md`](../diagrams/README.md) §7).

**The `C:` and `S:` prefixes are not used.** In SMTP a verb is always the client and a three-digit
code is always the server, so direction is recoverable from the content itself.

The wire row also corroborates an emptiness that nothing else can. `BeginData` and `Quit` are bare
verbs and carry no field lines, and they need no marker saying so, because `QUIT` visibly takes no
argument on the line directly above. A `Given` has no such corroboration anywhere in the table,
which is why it is the one place 🟤 is written — the reasoning, and the candidates rejected, are in
[`EXPLORE-gwt-form.md`](EXPLORE-gwt-form.md).

## The DATA payload is neither a verb nor a code

One wire row in every walk breaks the monospace rule's usual reading, because what crosses is
message content rather than protocol. There the RFC 5322 field names carry the monospace and the
values do not:

| 🟦 C · Step 12 | `SubmitContent` |
|:--|:--|
| MTA Client | ⬛ `Date:` Tue, 19 May 1998 09:14:02 -0700&#10;<br>`From:` Smith \<Smith@bar.com>&#10;<br>`To:` Jones@foo.com&#10;<br>`Subject:` Tuesday&#10;<br>(blank)&#10;<br>Blah blah blah...&#10;<br>`.` |
| | 🟦 **SubmitContent**&#10;<br>&nbsp;&nbsp;`content`: 194 octets, dot-unstuffed |

The terminating lone `.` is protocol and stays monospace. The step number is this walk's position,
not a handle — slices are referred to by name (rule 5).

## The wire is a code span because SMTP syntax is load-bearing

Reply codes and the `<>` around a reverse-path are protocol syntax; losing a bracket or a space
changes their meaning. `MAIL FROM:<Smith@bar.com>` is testable, *"a MAIL FROM command naming
Smith"* is not. Code spans render monospace in every renderer this project targets and survive
copy-paste into a terminal.

⚠️ **An address outside a code span loses its angle brackets** — GFM autolinks it and eats them,
and `<CRLF>` in running text vanishes entirely. Escape the opening bracket: `\<Smith@bar.com>`. The
address still autolinks, which is accepted; the brackets are RFC 5321 path syntax and are not. The
general rule is in `AGENTS.md` rule 5; what is SMTP's is that the brackets are *syntax* —
`MAIL FROM:<>` is not `MAIL FROM:`.

## 🟥 is overloaded — open

The chip means both *hotspot* and *error outcome*. Unresolved; see
[`EXPLORE-gwt-form.md`](EXPLORE-gwt-form.md).
