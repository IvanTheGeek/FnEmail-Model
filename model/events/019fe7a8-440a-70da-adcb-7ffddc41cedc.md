# `CommandRejectedAsOutOfSequence`

The server refused a syntactically valid transaction command because the session was not in a state that allows it, and its own state stayed exactly as it was.

| Grouping | Value | Fixed at |
|:--|:--|:--|
| Workflow | 2. Mail Transaction | Step 1 — timeline discovery |
| Stream root | Session | Step 1 — stream roots |
| Ownership swimlane | — | Step 6 — events section, per team or system |
| Originating actor | SMTP server (SMTP-receiver) | Step 3 — actor swimlane; provenance, not this event's row |

🟧 Event · confidence *certain* · recorded 2026-08-09, Step 1 Brain Storming, whole-spec sweep of RFC 5321 · `019fe7a8-440a-70da-adcb-7ffddc41cedc`

## Data

| Field | Type | Cardinality | Example | Example source |
|:--|:--|:--|:--|:--|
| `command_verb` | CommandVerb | Single | DATA | rfc5321.txt:1992 |

## Where RFC 5321 says so

| Section | Lines | Text |
|:--|:--|:--|
| §4.1.4 | 2516-2528 | *"If the commands in a transaction are out of order to the degree that they cannot be processed by the server, a 503 failure reply MUST be returned and the SMTP server MUST stay in the same state."* |
| §4.2.2 | 2787 | *"503 Bad sequence of commands"* |
| §3.3 | 1160 | *"Mail transaction commands MUST be used in the order discussed above."* |
| §3.3 | 1087-1089 | *"If a RCPT command appears without a previous MAIL command, the server MUST return a 503 "Bad sequence of commands" response."* |
| §4.1.4 | 2516-2518 | *"If the commands in a transaction are out of order to the degree that they cannot be processed by the server, a 503 failure"* |
| §3.3 | 1130-1132 | *"If there was no MAIL, or no RCPT, command, or all such commands were rejected, the server MAY return a "command out of sequence" (503) or "no valid recipients" (554) reply in response to the DATA command."* |
| §4.3.2 | 3119 | *"E: 503, 554"* |

Line numbers are into [`docs/rfc/rfc5321.txt`](../../docs/rfc/rfc5321.txt), the archived copy. Quotations keep the specification's own words and spellings (AGENTS.md rule 2).

## Notes

The UMBRELLA event that RecipientCommandRejectedOutOfSequence, DataCommandRejectedOutOfSequence, NestedMailCommandRejected and MailCommandRejectedForUninitializedSession are named instances of. Kept alongside them rather than instead of them, because each of those four carries its own citation, its own strength and its own trigger, and because this one carries a fact none of them do: the second MUST at 2527-2528, that the server stays in the same state. Note the citation spans a page break in the source — the sentence starts at 2516 and finishes at 2527-2528. 503 is the only reply that makes the session's accumulated state observable from outside; it is the sequencing rules rendered as a wire fact, which is exactly the argument the condemned status/phase register would deny. Rejected alternative names: BadSequenceReported, CommandRejectedWithStateUnchanged (the §4.2-4.3 sweep's name, which merged five unrelated reply codes with sharply different strengths — that merge is undone here). §4.3.2 also lists 554 alongside 503 for a pre-data DATA rejection.

FIELD HYGIENE PASS: struck two fields. session_id (f2C8D14) goes because RFC 5321 supplies no session identifier and f2C8D14 appears nowhere in the document — it is this project's own minted queue-id example from commit 6acfc68; the channel is the session's identity while it lives, which is doubly apt here because this event is rooted in Session. reply_code (503) struck as the wire rendering of a decision. ⚠️ That is the sharpest strike in the chapter, because the note above rests this event's distinctness on the code: '503 is the only reply that makes the session's accumulated state observable from outside'. What survives the strike is the fact the code was carrying, and it is stronger without it: the SECOND MUST at 2527-2528, that the server stays in the same state, which is a statement about state and not about rendering, and which the note already names as the thing none of the four named instances carry. The code and its gloss are preserved in the citations at 2516-2528 and 2787. command_verb SURVIVES here — unlike offending_verb on RecipientCommandRejectedOutOfSequence, which was a constant fixed by that event's name, this one genuinely varies across MAIL, RCPT and DATA, which is what makes this the umbrella.

COLLISION RESOLVED — merged from 2 independently brainstormed records. Both copies are the same umbrella over the same four instances, citing 4.1.4 2516-2528 and 4.2.2 2787 identically, with the same actor and the same stream root. One reached it from the transaction text at 3.3, the other from the sequencing text at 4.1.4, which is a union of citations, not a difference of events. Field lists already agreed after hygiene — command_verb alone, kept in both because it genuinely varies across MAIL, RCPT and DATA, which is what makes this the umbrella rather than an instance.

From the duplicate in 14. Protocol Fault Handling: Merged from candidates 236, 149 (RecipientCommandRejectedOutOfSequence), 155 (DataCommandRejectedOutOfSequence) and the 503 arm of 173; every citation re-checked in the file and correct. Rejected alternative names, all real state changes that describe the same fault at different verbs: RecipientCommandRejectedOutOfSequence, DataCommandRejectedOutOfSequence, CommandRejectedWithStateUnchanged, BadSequenceRefused. The strengths differ by verb and a merged event must not flatten them: RCPT without MAIL is a MUST 503 (1087-1089); out-of-order to the degree it cannot be processed is a MUST 503 with the state preserved (2516-2518 continuing to 2527-2528); DATA with no usable envelope is a MAY, and a MAY offering TWO codes with different meanings — 503 says "wrong order", 554 says "no valid recipients", and a diagnosing operator reads them differently, so a facilitator may reasonably split the DATA case back out. 503 is the only reply that makes the session's accumulated state observable from outside — it is §4.3.2's sequencing rules rendered as a wire fact, which is a strong argument that session state is real and worth recording, exactly what the condemned status/phase register denies. Also folded from 149: the same 503 for commands arriving between a 554 greeting and QUIT (§3.1 lines 978-979, a SHOULD). Client-side sibling from 155, handed to the transaction chapter: MessageDataWithheldByClient, from the MUST NOT at 1133-1135, "message data MUST NOT be sent unless a 354 reply is received".

FIELD HYGIENE PASS (chapter 14): two fields struck plus the session key. session_id struck — RFC 5321 names no session, transaction or connection identifier, and f2C8D14 is nowhere in the RFC; identity here is position in the session, which is the very thing the 503 reports. reply_code struck: 503 is fixed by the event's meaning and is the decision rendered. commands_already_accepted struck as a derived fold over the session's accepted commands — it is the read model the inherited note was really describing when it argued that 503 makes session state observable, and that argument is untouched by removing the copy from this event. command_verb KEPT as the only record of the refused command.
