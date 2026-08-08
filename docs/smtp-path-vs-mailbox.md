# Path and Mailbox — SMTP's two address types

What looks like one thing on the wire — an email address, sometimes wearing angle brackets — is
two different **types** in RFC 5321's grammar. A **Mailbox** is an address: `local-part@domain`. A
**Path** is an address *in routing position*: a Mailbox wrapped in `<` and `>`. The brackets are
not decoration around an address; they are the type constructor — and the proof is the **null
path**, `<>`, a legal Path containing no Mailbox at all. A Mailbox cannot be empty; a Path can.
The brackets are what make *nothing* sendable.

Written up 2026-08-08 from the [`WORKING` walk-through](paths/WORKING-helo-direct-single-recipient-v2.md)
of step 5, where the question was whether `MAIL FROM:` carries `<Smith@bar.com>` or a bare
Smith@bar.com. Citations give the archived spec ([`rfc/rfc5321.txt`](rfc/rfc5321.txt), line-linked)
and the canonical text at rfc-editor.org.

⚠️ **Naming collision, harmless but worth a sentence:** SMTP's *Path* has nothing to do with this
repository's *paths* (walked timelines in [`paths/`](paths/)). Same word, unrelated ideas; this
document is about SMTP's.

---

## The grammar

§4.1.2 — [archived ll. 2260–2268](rfc/rfc5321.txt#L2260) ·
[rfc-editor §4.1.2](https://www.rfc-editor.org/rfc/rfc5321#section-4.1.2):

```
Reverse-path   = Path / "<>"

Forward-path   = Path

Path           = "<" [ A-d-l ":" ] Mailbox ">"

A-d-l          = At-domain *( "," At-domain )
               ; Note that this form, the so-called "source
               ; route", MUST BE accepted, SHOULD NOT be
               ; generated, and SHOULD be ignored.
```

and the inner type, [archived l. 2314](rfc/rfc5321.txt#L2314):

```
Mailbox        = Local-part "@" ( Domain / address-literal )
```

Read bottom-up: a Mailbox is the address itself. A Path is a bracketed Mailbox, optionally
prefixed by the deprecated source-route remnant. A Forward-path is exactly a Path. A Reverse-path
is a Path **or the two literal characters** `<>`.

The terminology section behind the inner type is §2.3.11, *Mailbox and Address*
([archived l. 826](rfc/rfc5321.txt#L826) ·
[rfc-editor §2.3.11](https://www.rfc-editor.org/rfc/rfc5321#section-2.3.11)): *"an "address" is a
character string that identifies a user to whom mail will be sent or a location into which mail
will be deposited. The term "mailbox" refers to that depository."*

---

## Where each type appears on the wire

| Wire position | Grammar type | Citation |
|:--|:--|:--|
| `MAIL FROM:` argument | Reverse-path | `mail = "MAIL FROM:" Reverse-path` — §4.1.1.2, [l. 1913](rfc/rfc5321.txt#L1913); walkthrough form at §3.3, [l. 1027](rfc/rfc5321.txt#L1027) |
| `RCPT TO:` argument | Forward-path | §4.1.1.3, [l. 1985](rfc/rfc5321.txt#L1985) — even the special Postmaster forms are bracketed: `"<Postmaster@" Domain ">" / "<Postmaster>" / Forward-path` |
| `HELO` argument | Domain — no address type at all | `helo = "HELO" SP Domain CRLF` — §4.1.1.1, [l. 1831](rfc/rfc5321.txt#L1831) |
| `Received:` `FOR` clause | **Path or Mailbox** — the one place the grammar accepts either | `For = CFWS "FOR" FWS ( Path / Mailbox )` — §4.4, [l. 3339](rfc/rfc5321.txt#L3339) |
| `Return-Path:` header, written at final delivery | Reverse-path | `Return-path-line = "Return-Path:" FWS Reverse-path <CRLF>` — §4.4, [l. 3300](rfc/rfc5321.txt#L3300) |

So a conformant client sends `MAIL FROM:<Smith@bar.com>` — brackets on the wire — and a bare
`MAIL FROM:Smith@bar.com` does not parse. RFC 5321 carries no tolerance for the bracket-less form;
the old lore about accepting it was RFC 1123-era receiver robustness, and it did not survive into
this grammar.

---

## The null path

The two characters `<>` are literally transmitted. §3.6.3 requires it for undeliverable-mail
notifications, so that error reports can never loop
([archived ll. 1494–1522](rfc/rfc5321.txt#L1494) ·
[rfc-editor §3.6.3](https://www.rfc-editor.org/rfc/rfc5321#section-3.6.3)):

> *"One way to prevent loops in error reporting is to specify a null reverse-path in the MAIL
> command of a notification message. When such a message is transmitted, the reverse-path MUST be
> set to null"*

followed by the spec's own example of the wire line, verbatim at
[l. 1522](rfc/rfc5321.txt#L1522):

```
MAIL FROM:<>
```

§4.5.5, *Messages with a Null Reverse-Path*
([archived ll. 3787–3818](rfc/rfc5321.txt#L3787) ·
[rfc-editor §4.5.5](https://www.rfc-editor.org/rfc/rfc5321#section-4.5.5)), completes the rules:
everything not required to be null *"SHOULD be sent with a valid, non-null reverse-path"*, and
automated processors *"SHOULD NOT reply to messages with a null reverse-path, and they SHOULD NOT
add a non-null reverse-path, or change a null reverse-path to a non-null one, to such messages
when forwarding"*.

**This is why the types cannot be merged.** A bounce needs a sender field that says *no sender,
and do not bounce me*. There is no way to write that as a Mailbox — `Local-part "@" Domain` has no
empty form. The Path type is what makes the null case a first-class value instead of a sentinel.

---

## Consequences

**The brackets travel — they are part of the fact, not the renderer.** The bracketed form is not
private to the `MAIL` line: the `FOR` clause renders it, and the `Return-Path:` header written at
final delivery *"preserves the information in the \<reverse-path> from the MAIL command"* —
§4.4, [ll. 3205–3208](rfc/rfc5321.txt#L3205). Under the walk's view/renderer split (the renderer
owns the constants, the view owns the facts), the brackets land on the fact side: they appear at
every destination the value reaches.

**Storing Paths costs nothing and buys the null case.** The walk's fields `reverse_path`:
\<Smith@bar.com> and `forward_path`: \<Jones@foo.com> store the Path, brackets included, matching
their names. A future bounce-shaped walk writes `reverse_path`: `<>` as an ordinary value of the
same type. A bare-Mailbox field would need an invented no-value marker — exactly the kind of
manufactured apparatus the walk-through has been culling.

**One address, two case rules, split at the `@` — inside the Mailbox, inside the Path.** §2.4,
[l. 868](rfc/rfc5321.txt#L868): *"The local-part of a mailbox MUST BE treated as case sensitive"* —
while the Domain half is not. Any code that folds case on a whole Path corrupts the half the
specification protects.

**The source-route remnant is accept-only.** The optional `A-d-l` prefix inside a Path — the form
`<@relay1,@relay2:Smith@bar.com>` — is deprecated: the grammar's own comment says it *"MUST BE
accepted, SHOULD NOT be generated, and SHOULD be ignored"* ([ll. 2266–2268](rfc/rfc5321.txt#L2266)).
A parser must expect it; nothing here ever emits it.

**And a documentation hazard this repository already guards against:** outside a code span,
markdown eats angle brackets — an unescaped address autolinks and silently loses them. The rule
and its history are in [`AGENTS.md`](../AGENTS.md) §5: *"`MAIL FROM:<>` is not `MAIL FROM:`"*.
