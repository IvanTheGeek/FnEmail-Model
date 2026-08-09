# `RelayingDeclined`

The server refused a recipient because delivering it would mean carrying the mail onward to another host.

| Grouping | Value | Fixed at |
|:--|:--|:--|
| Workflow | 2. Mail Transaction | Step 1 — timeline discovery |
| Stream root | Recipient | Step 1 — stream roots |
| Ownership swimlane | — | Step 6 — events section, per team or system |
| Originating actor | SMTP server (SMTP-receiver) | Step 3 — actor swimlane; provenance, not this event's row |

🟧 Event · confidence *certain* · recorded 2026-08-09, Step 1 Brain Storming, whole-spec sweep of RFC 5321 · `019fe7a8-440a-70cd-adae-bfcab7c1b6a6`

## Data

| Field | Type | Cardinality | Example | Example source |
|:--|:--|:--|:--|:--|
| `forward_path` | ForwardPath | Single | \<Smith@bar.com> | walk scene |
| `refused_path` | Mailbox | Single | \<Smith@bar.com> | walk scene |
| `refusing_host` | Domain | Single | foo.com | walk scene |

## Where RFC 5321 says so

| Section | Lines | Text |
|:--|:--|:--|
| §3.3 | 1082-1083 | *"Similarly, servers MAY decline to accept mail that is destined for other hosts or systems."* |
| §3.3 | 1080-1082 | *"Servers MUST be prepared to encounter a list of source routes in the forward-path, but they SHOULD ignore the routes or MAY decline to support the relaying they imply."* |
| §3.3 | 1084-1087 | *"These restrictions make a server useless as a relay for clients that do not support full SMTP functionality. Consequently, restricted-capability clients MUST NOT assume that any SMTP server on the Internet can be used as their mail processing (relaying) site."* |
| §3.6.2 | 1454,1463-1464 | *"If it declines to relay mail to a particular address for policy reasons, a 550 response SHOULD be returned."* |
| §4.1.1.3 | 1973-1975 | *"Since hosts are not required to relay mail at all, xyz.com MAY also reject the message entirely when the RCPT command is received, using a 550 code (since this is a "policy reason")."* |
| §7.9 | 4379-4381 | *"When mail is rejected for these or other policy reasons, a 550 code SHOULD be used in response to EHLO (or HELO), MAIL, or RCPT as appropriate."* |

Line numbers are into [`docs/rfc/rfc5321.txt`](../../docs/rfc/rfc5321.txt), the archived copy. Quotations keep the specification's own words and spellings (AGENTS.md rule 2).

## Notes

⚠️ The reply code is NOT given in this span; 550 is conventional and is a GUESS, flagged. Real data makes the case concrete in a way a placeholder could not: the server is foo.com, so RCPT TO:\<Smith@bar.com> IS a relay request and this event fires on it — and the SAME address \<Smith@bar.com> was a perfectly legitimate reverse-path one command earlier. That is the whole lesson of rule 8 in eight words. This is the single most consequential MAY in the chapter — it is what makes a receiving server not an open relay — and the RFC spends one sentence on it. Two arms of one sentence at 1080-1082: SHOULD ignore the routes (SourceRouteStripped) or MAY decline the relaying they imply (this event). Depends on a relay-policy configuration event outside this chapter. Killed today by the responsibility-boundary cutoff and the inbound-only scope; both named, neither applied. Rejected alternative names: RelayRefused, RelayTaskDeclined, OpenRelayRefusal.

FIELD HYGIENE PASS: struck three fields. reply_code (550) was already flagged above as a GUESS with no source in the span — an unsourceable value is the signal the field was invented, and a reply code is the wire rendering of a decision in any case. reason_text ('relay access denied') struck likewise: the phrase appears nowhere in RFC 5321. receiving_host (foo.com) struck as ambient — the receiving host is the server recording the event, identified by the channel it is answering on. ⚠️ That strike costs something real and it is recorded rather than hidden: the note's lesson depends on knowing the server is foo.com, because that is what makes RCPT TO:\<Smith@bar.com> a relay request while the same address was a legitimate reverse-path one command earlier. The lesson survives in this note; what it needs is the session's identity, which the channel carries, not a copy of the server's own name on every event it writes. forward_path survives and is the whole point of the event.

COLLISION RESOLVED — merged from 2 independently brainstormed records. The 3.3 quote is identical in both copies, the actor and stream root are the same, and one copy is already a three-section merge of the same state change — a host refusing to carry mail onward for an address. refusing_host was the disagreement: one copy argued it is not an actor restatement because it is what makes \<Smith@bar.com> a relay request at all; the other struck it as ambient and flagged the cost with a ⚠️. Struck, because the receiving host is the actor recording the event and the channel carries the session's identity while it lives, and because this refusal stamps nothing anywhere — unlike final delivery, where the delivering host writes itself into the message, which is why delivering_host survives on ReturnPathHeaderStrippedBeforeDelivery and refusing_host does not survive here. The lesson the field was carrying survives in the notes, which is where the copy that struck it put it.

From the duplicate in 16. Abuse, Security and Registry: MERGE of three inherited candidates naming one state change from three sections: RelayingDeclined (3.3), RelayTaskDeclined (3.6.2) and RecipientRejectedForPolicy (4.1.1.3). Names rejected: RelayTaskDeclined, RecipientRejectedForPolicy, RelayRefusedForPolicy, ForwardPathRefusedAsRelay. CITATION TRAP carried forward from the 3.6 sweep and re-verified: the 3.6.2 quote is split across a page break — the sentence begins 'If it declines to' at the end of line 1454 and resumes at 1463, the footer and running header occupying 1455-1462 — so a naive line range looks wrong; the lines field records both halves. Strength: MAY on declining (1082, 1429, 1973), SHOULD on the 550 (1464, 4380). A relay that declines for policy and returns a different failure code violates nothing; refusal for NON-policy reasons carries no specified code at all. This is the single most consequential MAY in the RFC — it is what makes a receiving server not an open relay — and 3.3 spends eight words on it. Real data chosen to make it concrete (rule 8): the server is foo.com, so RCPT TO:\<Smith@bar.com> IS a relay request, and the same address is a legitimate reverse path one command earlier and an illegitimate forward path here. Cross-referenced not merged: MailRejectedForPolicyReason, the general form, which can land at EHLO where no forward path exists. Screens that would have killed it: inbound-only scope, responsibility-boundary cutoff, twelve-slice model. FIELD HYGIENE: struck session_id (SessionId, f2C8D14) — RFC 5321 names no session, transaction or connection identifier, so while the session lives the channel is its identity, and what identifies THIS event is the forward path it refused, already a field; the value was this project's own minted queue_id, commit 6acfc68. Struck reply_code (550): 1464 and 4380 both say SHOULD on the code, which renders the refusal without being it — the note's own observation, that declining with a different code violates nothing, is the proof that the code is not the fact. refusing_host is not an actor restatement here: it is what makes \<Smith@bar.com> a relay request at all.
