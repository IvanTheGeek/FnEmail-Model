# Exploring — the View Slice form

**Open. Nothing here is adopted.** The Command Slice form was reworked on 2026-08-07 and the View
Slice was deliberately left behind, because bringing it along would have prejudged a question the
corpus does not settle. This document is the starting point: every View Slice as it stands today,
unchanged, so the rework has something to argue against.

Command Slices now read: wire, command **with its actor-supplied fields**, event, and a `Given`
carrying the minimal dependency. See [`STEP-FORM.md`](STEP-FORM.md).

---

## The four View Slices, exactly as they are now

Three are exercised by [`helo-direct-single-recipient.md`](helo-direct-single-recipient.md). The
fourth is not exercised by any path.

### `SessionState` — walked, step 3

| 🟩 V · Step 3 | `SessionState` |
|:--|:--|
| Consumed by | ⬜ `MailFrom` · `RcptTo` · `BeginData` |
| | 🟩 **SessionState**&#10;<br>&nbsp;&nbsp;`identified`: true&#10;<br>&nbsp;&nbsp;`transaction_open`: false |
| Sources | 🟧 **ConnectionAccepted** · **ClientIdentified** |

### `RecipientDirectory` — walked, step 5 · the translation boundary

| 🟩 V · Step 5 | `RecipientDirectory` &nbsp;*(translation boundary — H3)* |
|:--|:--|
| Consumed by | ⬜ `RcptTo` |
| | 🟩 **RecipientDirectory**&#10;<br>&nbsp;&nbsp;`is_local`: true |
| Sources | 🟨 translated from the **Directory context** — external, deferred |

### `TransactionState` — walked, step 7

| 🟩 V · Step 7 | `TransactionState` |
|:--|:--|
| Consumed by | ⬜ `RcptTo` · `BeginData` · `SubmitContent` |
| | 🟩 **TransactionState**&#10;<br>&nbsp;&nbsp;`open`: true&#10;<br>&nbsp;&nbsp;`reverse_path`: \<Smith@bar.com>&#10;<br>&nbsp;&nbsp;`recipient_count`: 1 |
| Sources | 🟧 **MailTransactionStarted** · **RecipientAccepted** |

### `SessionTranscript` — **never walked by any path**

No step number, because no walk reaches it. Reconstructed from `../event-model.md` and shown in the
same form for comparison; its values are therefore **not instantiated from a real walk**, which
breaks rule 8 and is exactly why it is the weakest of the four.

| 🟩 V | `SessionTranscript` |
|:--|:--|
| Consumed by | ⬜ an operator reading the session |
| | 🟩 **SessionTranscript**&#10;<br>&nbsp;&nbsp;*no walked values — never exercised* |
| Sources | 🟧 every event in the session |

---

## What is already known to be wrong with this form

Recorded so the rework does not have to rediscover it.

**It has no `Given`.** Command Slices now carry the minimal dependency explicitly. A View Slice's
`Sources` row is *arguably* the same thing under another name — it lists the events the read model
folds — but `Sources` and `Given` are not obviously the same claim, and the difference has never
been stated.

**It reads bottom to top while its neighbors read top to bottom.** The current rule says so
outright. Ten stacked steps that change direction three times is a real cost that was accepted when
View Slices were the only exception; the Command Slice rework has now moved the baseline.

**The rows may already be a Given/When/Then in disguise.** Mapping them onto the corpus vocabulary:

| current row | plausibly is |
|:--|:--|
| `Sources` | **Given** — the events |
| `Consumed by` | **When** — the query |
| the read model | **Then** — the result |

If that holds, the form is GWT written in non-corpus words in non-corpus order — and relabeling
would be a rename rather than a redesign.

**`SessionTranscript` has no real values.** It is in the model and in no walk, so it is the one
slice with nothing instantiated behind it. Under *paths are the source, slices are derived*, that
makes it a slice with no evidence.

---

## ⚠️ The corpus disagrees with itself here — this is the whole reason for a separate pass

Detail and citations in `research/GWT-FINDINGS.md`. In brief:

**Dymitruk specified a View as Given/Then with no `When` for six years**, from 2019 through YOW! 2023
and into a 2025 talk, giving a different reason each time — no command, no action, *"you can't reject
state it is what it is"*.

**Then he began adding a `When`, in three podcast episodes** — 2025-03-28, 2025-11-26, 2026-03-29 —
where the `When` is a **query with a key**, not event filtering. His stated motive is a uniform test
signature. It has **not** propagated to his talks or to eventmodeling.org, and episode 43 puts it as
*"the standard I'd like to push forward"*, which is an aspiration.

**Dilger's own artifacts contradict each other**: his 2026 blog writes a view scenario with a query
in the `When`, while his tooling still emits `"when": []` and generates `.when([])`.

**And his book draws it two ways in one chapter** — Fig. 13.5 omits the `When` row entirely,
Fig. 13.6 shows it present and empty.

So *"does a View Slice have a When"* has no settled answer to copy. **Whatever this project picks is
a choice, and it needs to be labeled as one.**

⚠️ Note also that `../event-model.md` **already ships** `WHEN Query(session)` on a state view — a
form whose only support is those three podcast episodes, and which names no key, which is precisely
the *"implicit requirement"* episode 24 complains about.

---

## Questions the rework has to answer

1. **Does a View Slice get a `When`?** And if so, is it a query with a key — `Query(session_id=…)`
   rather than `Query(session)`?
2. **Is `Sources` just `Given` renamed?** If yes, the fix is a relabel and the row order.
3. **Which direction does it read?** Keeping bottom-to-top preserves the existing documents;
   flipping it makes every step in a path read the same way.
4. **What is a View's dependency on an external context?** `RecipientDirectory` has no event of ours
   at all. On a Command Slice that now shows as a 🟨 `Given`; a View has nowhere to put it except
   `Sources`, where it currently sits.
5. **What about a read model nothing has walked?** `SessionTranscript` has no instantiated values,
   and no form will fix that — only a path that reaches it will.
