# Step 1 Sub-step 6 — Analysis Document (RFC 5321): Business Processes, Domain Expert Questions, two recorded decisions, and the assembly-only quality checklist run

# Step 1 · Sub-step 6 — Analysis Document (RFC 5321)

This is the assembly half of Brain Storming: the parts of the kit's analysis document
(`SKILL.md` around lines 459-510) that no individual sweep could produce, plus the quality
checklist items (around lines 573-587) that only run once the whole roster exists.

Chapter short names used throughout: Session, Transaction, Content, Trace, Handoff, Resolution,
Queueing, Relaying, Gatewaying, Expansion, Forwarding, Notification, Diagnostics, Faults,
Provisioning, Abuse.

All line numbers are into `docs/rfc/rfc5321.txt` and were read, not remembered.

---

## Recorded decisions

### Decision — the Interview Phase was skipped

The kit's Interview Phase is conditional (`SKILL.md` around lines 46-56): it is skipped when
written requirements, documented business rules, and named domain experts are all present.

**Justification accepted.** RFC 5321 is arguably all three at once. It is the written
requirement; §2.4, §4.1.4, §4.5.3 and §4.5.4 are documented business rules stated as `MUST` /
`SHOULD` / `MAY`; and the named domain expert is the document's own editor, whose name stands on
every page footer of the archived text. Questions 1 and 2 of the interview would have returned
(A) and (A) with no follow-up triggers.

**What was lost, and it is not nothing.** Question 3, *Known Complexity Areas*, exists to aim the
brainstorm before it runs — "Complex areas often have hidden events; identifying them upfront
ensures they're covered." Had it run, three areas would have been named, and all three turn out to
be exactly where the roster is now weakest:

| Area question 3 would have named | Where it bites | What the sweeps did instead |
|---|---|---|
| The `250`-after-DATA handoff | §4.1.1.4 lines 2036-2049, §4.2.5 lines 2943-2987, §6.1 lines 3962-3968 | Produced three names for one fact — `MessageAcceptedForProcessing`, `DeliveryResponsibilityAccepted`, `ResponsibilityForMessageAccepted` — across two chapters |
| Partial-recipient failure | §4.1.1.4 lines 2042-2045 against §3.3 lines 1144-1150 | Produced `PartialFailureSignaledInDataReply`, `RecipientFailureReportedAfterContentAccepted` and `PartialDeliveryFailureDetected` without settling whether responsibility is per message or per recipient |
| Null-reverse-path loop suppression | §6.1 lines 3983-3987 against §4.5.5 lines 3796-3801 | Produced `NullReversePathSet` in two chapters and left the postmaster-redirect branch unreconciled with the `MUST NOT` |

The skip is recorded as a decision rather than a silence because the three weakest clusters in the
checklist run below are the three the skipped question was designed to prevent. The cost was real
and it is measurable in the output.

### Decision — the eleven sweeps were truncated by a schema cap, and the re-sweep only partly lifted it

Every one of the eleven parallel sweeps returned **exactly 32 events**, which is the schema's array
cap, not a natural stopping point. Eleven independent readers agreeing to the digit is a cap
signature, not a finding. The raw set of 352 candidates is therefore **truncated, not exhausted**,
and the set-aside piles in `extras.md` show each sweep still had material in hand when it stopped.

A per-chapter re-sweep was run to recover what the cap cut off. It worked, and it is measurably
incomplete. Counting the roster:

| Chapter count | Chapters at that count | Reading |
|---|---|---|
| 40 | Session, Transaction, Content, Resolution, Relaying, Faults, Provisioning, Abuse | Eight chapters land on the identical round number — the re-sweep hit a second cap at 40 |
| 32 | Gatewaying, Expansion, Diagnostics | Three chapters still carry the **original** 32-item cap; the re-sweep did not extend them at all |
| 38, 35, 34, 26, 18 | Trace, Queueing, Handoff, Notification, Forwarding | Below every cap, so these five are the only chapters that plausibly ran to exhaustion |

**Recorded as a known limit on completeness.** Eleven of sixteen chapters terminate on a cap value
rather than on running out of material. Any later step that treats this roster as the complete
event set for RFC 5321 is treating a truncation artifact as a boundary. The five chapters that
stopped below a cap are the only ones where "we found everything" is even arguable.

---

## Business Processes

Written in the kit's Actor / Steps / Outcomes shape (`SKILL.md` around lines 494-498). Examples use
real values per repository rule 8 — foo.com, bar.com, 192.0.2.10, 203.0.113.20,
\<Smith@bar.com>, \<Jones@foo.com>, message id f2C8D14, timestamp "Tue, 19 May 1998 09:14:07 -0700".

**1. Session establishment and identification** — a client and a server bring a transmission
channel into a state where mail commands are legal.

- Actor: SMTP client opens; SMTP server answers. §3.1 lines 961-962, §3.2 lines 987-998.
- Steps: client opens the channel to 192.0.2.10 → server answers `220` naming itself foo.com, or
  answers `554` and formally refuses the session while holding the connection open (§3.1 lines
  973-979) → client sends `EHLO` bar.com, falling back to `HELO` bar.com if the server errors
  (§3.2 lines 996-998), or substituting the address literal for the domain when it has no usable
  name (§4.1.1.1 lines 1787-1800) → server may check the claimed name against 203.0.113.20 but
  `MUST NOT` refuse mail on the result (§4.1.4 lines 2481-2488) → server answers `250` with its
  extension list.
- Outcomes: both ends are in the initial state — no transaction in progress, all state tables and
  buffers cleared (§4.1.1.1 lines 1822-1825). The session now holds a claimed client identity and a
  fixed extension vocabulary. A later `EHLO` mid-session re-runs this and clears everything as if
  `RSET` had arrived (§4.1.4 lines 2453-2459).

**2. Mail transaction** — one envelope and one content are accumulated and committed.

- Actor: SMTP client drives; SMTP server buffers and judges. §3.3 lines 1017-1023.
- Steps: `MAIL FROM:<Smith@bar.com>` → server answers `250`, or refuses permanently or temporarily,
  or accepts provisionally and defers judgment until forward-paths can be examined (§3.3 lines
  1034-1045) → one `RCPT TO:<Jones@foo.com>` per recipient, appending to the forward-path buffer and
  touching nothing else (§4.1.1.3 lines 1936-1938) → `DATA` → `354` → content lines →
  `<CRLF>.<CRLF>`.
- Outcomes: the terminator both ends the content and confirms the transaction (§3.3 lines
  1116-1118). The server processes the stored reverse-path, forward-paths and mail data, then clears
  all three buffers (§4.1.1.4 lines 2036-2040). The session returns to no-transaction-in-progress
  and a further `MAIL` becomes legal (§4.1.4 lines 2502-2512).

**3. Responsibility handoff** — custody of a message moves from client to server, or does not.

- Actor: SMTP server decides; SMTP client is bound by the decision. §4.2.5 lines 2943-2987.
- Steps: server receives `<CRLF>.<CRLF>` → server processes → server returns exactly one of `2yz`,
  `4yz`, `5yz`.
- Outcomes: on `2yz` the server owes delivery, or retry at §4.5.4 intervals, or notification to the
  address in the `MAIL` command (§4.2.5 lines 2943-2957), and `MUST NOT` lose the message for
  frivolous reasons (§6.1 lines 3962-3968). On `4yz` the server `MUST NOT` attempt delivery later
  and the client retains the message (§4.2.5 lines 2959-2964). On `5yz` the server is barred
  permanently and the client `SHOULD not` retry the same server without user review (§4.2.5 lines
  2981-2987). This is the single most consequential state change in the protocol and the RFC gives
  it no name.

**4. Next-hop resolution** — a delivery domain becomes an IP address to connect to.

- Actor: SMTP client, on answers from a DNS resolver. §5.1 lines 3826-3935.
- Steps: lexically identify foo.com from the forward-path → MX lookup → follow a CNAME and restart
  → on non-existent-domain report an error; on temporary error queue and retry; on an empty MX list
  assume an implicit MX at preference 0 → resolve each exchanger name to an address record (§5.1
  lines 3854-3862) → sort by preference, randomize ties, keep the resolver's order for a multihomed
  host (§5.1 lines 3883-3896) → if relaying, discard the relay's own entry and everything at or
  below its preference (§5.1 lines 3917-3935).
- Outcomes: an ordered candidate list, at least two of which `SHOULD` be tried (§5.1 lines
  3874-3878); or a reported error in one of three distinct branches; or, after a self-match with
  nothing left, the message returned as undeliverable.

**5. Queueing and retry** — a message that could not go now acquires a lifetime of its own.

- Actor: sending SMTP client and its queue runner. §4.5.4.1 lines 3687-3772.
- Steps: message that cannot be transmitted immediately is queued **with its envelope** (lines
  3691-3693) → attempt fails → mandatory delay before retrying that destination, `SHOULD` be at
  least 30 minutes (lines 3703-3707) → cadence widens after the first hour (lines 3720-3723) →
  retries continue to a give-up time generally of at least 4-5 days (lines 3709-3714).
- Outcomes: transmitted and discharged, or given up and converted to a permanent failure. Along the
  way: unreachable hosts get their own list (lines 3716-3718), receiving mail from foo.com may flush
  the queue for foo.com (lines 3725-3730), recipients sharing a destination are consolidated into
  one transaction (lines 3759-3767), and `5yz` responses to `MAIL` `MUST NOT` be cached (line 3750).

**6. Relaying** — a server accepts the task of carrying mail onward and becomes a client.

- Actor: relay SMTP server, then the same host acting as SMTP client. §3.6.3 lines 1494-1529.
- Steps: accept the relaying task for \<Jones@foo.com> the way it would accept mail for a local user
  → assume the client role → run *Next-hop resolution* and *Mail transaction* against the next hop
  → on failure, run *Non-delivery notification*.
- Outcomes: the message moves, with the header section and body untouched except for the relay's own
  `Received:` line — a relay `MUST NOT` inspect the content beyond adding that line and optionally
  detecting loops (§3.6.3 lines 1524-1529, §2.3.10 lines 810-814). A relay may instead decline for
  policy with `550` (§7.9 lines 4375-4381).

**7. Gatewaying** — a message crosses between transport service environments.

- Actor: gateway SMTP. §3.7 lines 1533-1546, §3.7.4 lines 1606-1617, §3.7.5 lines 1633-1638.
- Steps: receive from one environment → apply the transformations a relay is forbidden to make →
  transform `From:`, `To:` and `Cc:` addresses to RFC 5322 syntax, fully-qualified names, and
  reply-usable form (§3.7.4 lines 1606-1612) → prepend a `Received:` line naming the environment it
  forwarded from → set the envelope return path from the foreign environment's error-return address,
  defaulting to the originator's address only as a last resort (§3.7.5 lines 1633-1638).
- Outcomes: the message is acceptable on the far side. Layering semantics `SHOULD` survive the
  crossing and some information loss is "almost inevitable" (Appendix E lines 5130-5139). Building
  an SMTP envelope from RFC 822 header fields alone `MUST NOT` be used to gateway inbound
  (Appendix B lines 4773-4777).

**8. Non-delivery notification** — a failure after acceptance is reported to the sender.

- Actor: whichever host first determines delivery is impossible. §3.6.3 lines 1494-1509, §6.1 lines
  3970-3989.
- Steps: determine the message cannot be delivered → construct the notification → address it to the
  envelope return path, stripping any explicit source route down to its final hop → send it with
  `MAIL FROM:<>`.
- Outcomes: the originator learns. Three suppressions apply: a null return path means the receiver
  `MUST NOT` notify at all (§6.1 lines 3983-3984), a notification about a notification is forbidden
  outright (§3.6.3 lines 1505-1506), and the queueing strategy `MUST NOT` answer an error message
  with an error message under any circumstances (§4.5.4 lines 3682-3683). Local logging of null
  address events remains permitted (§6.1 lines 3984-3987).

**9. Final delivery** — the message leaves the SMTP environment.

- Actor: delivery SMTP server, then a message store or mail user agent. §4.4 lines 3204-3212.
- Steps: insert a `Return-Path` line at the head of the mail data, preserving the reverse-path from
  the `MAIL` command → optionally remove any pre-existing `Return-Path` header fields first (§4.4
  lines 3237-3238) → deposit in the store or hand to the user agent (§2.3.10 lines 807-810).
- Outcomes: exactly one return path `SHOULD` be present in the delivered message (§4.4 lines
  3240-3243). Any local storage transformation is expected to be reversible.

**10. Alias and list expansion** — one accepted recipient becomes many.

- Actor: the RFC's "recipient mailer" — an alias resolver or a mailing list expander. §3.9 lines
  1689-1708.
- Steps (alias): replace the pseudo-mailbox address in the envelope with each expanded address in
  turn, leaving the rest of the envelope and the body unchanged (§3.9.1 lines 1712-1716).
- Steps (list): the same replacement, **plus** retarget the backward-pointing address to the list
  administrator — that change is the whole difference between the two models (§3.9.2 lines
  1720-1729).
- Outcomes: N copies with their own delivery fates. The header section `MUST` be left unchanged, the
  `From` field in particular (§3.9 lines 1694-1696). Dropping addresses by heuristic is strongly
  discouraged (§3.9 lines 1703-1706). A list that modifies further stops being a hop in transit and
  becomes a full MUA that accepts a delivery and posts a new message (§3.9.2 lines 1743-1746).

**11. Forwarding for address correction** — mail addressed to an old mailbox reaches a new one.

- Actor: receiving SMTP server acting as forwarder; SMTP client or user agent on the other side.
  §3.4 lines 1164-1211.
- Steps: server is aware of an address change → it either forwards and discloses the corrected
  address with `251`, or forwards silently with `250`, or rejects with `551` and the replacement
  address, or rejects with `550` and no address information at all.
- Outcomes: the mail moves or does not; separately, the client's stored address for the
  correspondent may or may not be updated — the server `MUST NOT` assume it will be, or even that
  the information reaches the user (§3.4 lines 1194-1196 and 1204-1206). A client that acts on
  `251` or `551` without establishing the server's authenticity is exposed to a man in the middle
  (§7.4 lines 4286-4290).

**12. Address diagnostics** — a requester asks about an address without sending mail.

- Actor: receiving SMTP server. §3.5 lines 1215-1418, §4.1.1.6 through §4.1.1.9.
- Steps: `VRFY` \<Smith@bar.com> or `EXPN` for a list → server verifies, or declines with `252`, or
  refuses → reply carries the mailbox in `<local-part@domain>` form with no source route (§3.5.2
  lines 1337-1357).
- Outcomes: no protocol state changes — these commands have no effect on the reverse-path,
  forward-path or mail data buffers and may be issued at any time (§4.1.1.7 lines 2143-2145,
  §4.1.1.8 lines 2157-2159, §4.1.1.9 lines 2197-2199). What changes is what the requester now knows,
  which is why this process appears in the roster almost entirely as disclosure events. A server
  `MUST NOT` return `250` unless it actually verified (§3.5.3 lines 1373-1374), and a site that
  disables the commands `MUST` return `252` rather than anything readable as a verdict (§7.3 lines
  4243-4245).

**13. Protocol fault handling** — a command the server cannot honor gets a reply and the session
survives.

- Actor: receiving SMTP server. §4.1.4 lines 2514-2528, §4.2.4 lines 2933-2939,
  §3.8 lines 1660-1664.
- Steps: bad argument → `501`, state unchanged → wrong order → `503`, state unchanged →
  unrecognized verb → `500` → recognized but unimplemented → `502` → limits exceeded → `500`, `501`,
  `452` or `552` per §4.5.3.1.9 lines 3554-3569.
- Outcomes: the session stays open. A server that closes the connection on a command it did not
  understand is in violation (§3.8 lines 1660-1664). An extended server `MUST NOT` advertise a
  capability it will answer with `502` or `500` (§4.2.4 lines 2937-2939).

**14. Session termination** — the channel closes, by agreement or otherwise.

- Actor: SMTP client normally; SMTP server on shutdown or timeout. §3.8 lines 1642-1676,
  §4.1.1.10 lines 2207-2219.
- Steps: client sends `QUIT` → server answers `221` → server closes.
- Outcomes: the server `MUST NOT` intentionally close except after `QUIT` and `221`, after
  detecting the need to shut down and returning `421`, or after a timeout (§3.8 lines 1646-1658). A
  premature close obliges the server to cancel the pending transaction but **not** to undo any
  previously completed one, and to act as if a `4yz` had been received (§4.1.1.10 lines 2215-2219);
  the client `SHOULD` treat the same loss as a `451` (§3.8 lines 1671-1676).

**15. Server provisioning** — an operator or implementer fixes what the server will do before any
session exists.

- Actor: implementer, administrator, site operator. §4.5.1 lines 3380-3409, §4.5.3, §7.5, §7.9.
- Steps: implement the nine required commands → provision postmaster as a case-insensitive local
  name at every domain served, including the domainless `RCPT TO:<Postmaster>` form → set size
  ceilings, recipient limits, timeouts, retry parameters → set relay policy and the sources it
  applies to → decide what the greeting discloses.
- Outcomes: every runtime event above is bounded by a value set here. Several of these settings
  place the server in violation the moment they take effect — a recipient limit below 100 (§4.5.3.1.8
  lines 3537-3539), a missing required command (§4.5.1 lines 3380-3392), an advertised capability
  that answers `502` (§4.2.4 lines 2937-2939).

**16. Registration** — a keyword becomes legal for everyone.

- Actor: IANA, IESG, registrants. §8 lines 4383 onward, §2.2.2 lines 532-585.
- Steps: a Standards-Track or Experimental document is approved → IANA enters the extension,
  address literal tag, transmission type or trace header field in the relevant registry.
- Outcomes: the vocabulary every conforming implementation may use changes. Two systems may instead
  agree bilaterally on an `X`-prefixed verb, legal between those two and no others
  (§4.1.5 lines 2536-2541).

---

## Questions for Domain Expert

These are the places where RFC 5321 is genuinely ambiguous. They are recorded here rather than
resolved in the roster, because a sweep that resolves one of these silently hides the decision.

**Q1 — At the `250` after `<CRLF>.<CRLF>`, is responsibility taken for the message or for each
recipient?**
§4.1.1.4 lines 2042-2045 is categorical: "The SMTP model does not allow for partial failures at this
point: either the message is accepted by the server for delivery and a positive response is
returned or it is not accepted and a failure reply is returned." But §3.3 lines 1144-1150 concedes
that "some servers do not perform recipient verification until after the message text is received"
and tells them to treat a per-recipient failure as a subsequent failure. So the reply is per
message and the failure is per recipient. Which one carries the identity that a later notification
is about? The roster currently answers both ways at once.

**Q2 — When some accepted recipients deliver and some do not, is that one notification or several?**
§6.1 lines 3970-3972 says the receiver "MUST formulate and mail a notification message" — singular,
for a failure, with no plural form given. §3.3 lines 1148-1150 warns only that a `550` after the
data are accepted "makes it difficult or impossible for the client to determine which recipients
failed." Nothing states whether one notification lists every failed recipient or each failure gets
its own. The roster carries `NonDeliveryNotificationComposed` with a meaning that hedges across both.

**Q3 — Does the postmaster redirect of a failed notification violate the null-reverse-path
prohibition?**
§6.1 lines 3983-3984: if the return path is null the receiver `MUST NOT` send a notification.
§4.5.5 lines 3796-3801: when delivery of a notification fails, "at some hosts the MTA is set up to
forward such failed notification messages to someone who is able to fix problems with the mail
system, e.g., via the postmaster alias." A forward is not a notification — but it is a message
generated because a message failed, addressed to a party the original sender did not name. The RFC
never reconciles these. `FailedNotificationRedirectedToPostmaster` and
`NotificationSuppressedForNullReturnPath` currently coexist in the roster with no rule for which
fires.

**Q4 — What does "reported as an error" mean in §5.1?**
Three separate branches of §5.1 end in that phrase — non-existent domain (line 3840), no usable MX
(line 3846), MX records present but none usable (line 3852). The RFC never says to whom, by what
mechanism, or at what point. A session reply, a bounce to \<Smith@bar.com>, and a log line are all
consistent with the text, and they are three different events with three different actors. The
roster's `RoutingErrorReported` is the union of all three and therefore names none of them.

**Q5 — Is a relay that rewrites the envelope still a relay?**
§2.3.10 lines 810-814 defines a relay as one that transmits mail "without modification to the
message data other than adding trace information" — message *data*, silent on the envelope. §5.1
lines 3909-3911 lets a designated mail exchanger relay "potentially after having rewritten the
MAIL FROM and/or RCPT TO addresses." So envelope rewriting is permitted to a relay by §5.1 and
unaddressed by the definition it is supposed to satisfy. `ReversePathRewrittenBeforeRelay` and
`ForwardPathRewrittenBeforeRelay` sit on that gap.

**Q6 — What identifies a message across a hop?**
The trace line's `ID` clause "MAY contain an @ as suggested in RFC 822, but this is not required"
(§4.4 lines 3170-3171) — so it is local and optional. No field in RFC 5321 identifies the same
message at two hops. When an alias expands one accepted recipient into three (§3.9.1 lines
1712-1716), are those one message or three? When a client transmits one copy for many recipients
(§4.5.4.1 lines 3759-3763) and the server delivers to each, how many messages existed? Every
Message-stream event in the roster presumes an answer.

**Q7 — Does an automated reply to a null-reverse-path message include delivery and disposition
notifications?**
§4.5.5 lines 3815-3818 tells automated email processors they "SHOULD NOT reply to messages with a
null reverse-path." The same section (lines 3789-3793) lists DSNs and MDNs as things *required* to
carry a null reverse-path. An MDN generated for a DSN would be a reply to a null-reverse-path
message that a standard requires to exist. Which rule wins is unstated.
`AutoReplySentToNullReversePathMessage` records the behavior without deciding whether it is always
a violation.

**Q8 — After a site-policy recipient limit, what state is the transaction in?**
§4.5.3.1.10 lines 3601-3608 permits a server to "simply return the 503 after DATA without
returning any previous negative response" — that is, to answer every `RCPT` in the transaction with
`250` and then refuse `DATA` with `503` for a reason the client was never told. The client holds a
buffer of accepted forward-paths and a sequence error. Are those recipients accepted, refused, or
in neither state? `DataCommandRefusedAfterRecipientOverflow` names the refusal and leaves the
recipients undefined.

**Q9 — Is `TURN` in scope at all?**
The only appearance of the role reversal in RFC 5321 is Appendix F.1 lines 5166-5171, a
deprecated-features note: the command "raises important security issues," "can easily be used to
divert mail from its correct destination," and "Its use is deprecated." The state change is named
only inside the description of why not to do it. `ClientServerRolesSwitched` is the roster's only
event whose sole grounding is a deprecation note, and whether Step 1 should carry deprecated
mechanisms is a scope question the RFC cannot answer.

**Q10 — Who is the actor for a loss?**
§6.1 lines 3964-3968 forbids the receiver to lose an accepted message "for frivolous reasons, such
as because the host later crashes." A crash initiates nothing. `AcceptedMessageLost` is a fact the
model must carry and has no actor that could be listed in a Role Catalog; the same shape recurs for
`ConnectionLostWithoutQuit`. Whether a stream may hold actorless events is a method question, not
an SMTP one.

---

## Quality checklist — the assembly-only items

Three of the kit's checklist items cannot be evaluated by any single sweep. Run against the roster
of 567 entries.

### "No overlapping event semantics — two events don't mean the same thing" — **FAIL**

**567 entries, 491 distinct names, 76 redundant entries.**

**(a) Exact-name collisions — 72 names, 4 of them in three chapters each.** These are the same name
carrying the same meaning in more than one chapter, which is duplication rather than reuse, since
each chapter states the meaning again in its own words.

| Event name | Chapters |
|---|---|
| `AddressUpdateDisclosureRestricted` | Forwarding, Provisioning, Abuse |
| `DataCommandRefusedAfterRecipientOverflow` | Transaction, Content, Faults |
| `MailAcceptedDespiteMalformedTraceHeader` | Trace, Gatewaying, Faults |
| `RecipientRejectedWithForwardingAddress` | Transaction, Forwarding, Abuse |
| `AcceptedMessageLost` | Handoff, Notification |
| `AddressBookEntryUpdated` | Forwarding, Abuse |
| `BilateralAgreementEstablished` | Provisioning, Abuse |
| `BlindCopySentAsSeparateTransaction` | Expansion, Abuse |
| `CommandRejectedAsOutOfSequence` | Transaction, Faults |
| `ConnectionClosedDefensively` | Session, Abuse |
| `ConnectionClosedInViolation` | Session, Faults |
| `ConnectionRefusalConfigured` | Session, Provisioning |
| `CorrectedDestinationDisclosed` | Forwarding, Abuse |
| `DataCommandRejectedOutOfSequence` | Transaction, Content |
| `DataTerminationTimeoutExpired` | Session, Content |
| `DeliveryAttemptFailed` | Queueing, Relaying |
| `DeliveryFailedAfterAcceptance` | Handoff, Notification |
| `DeliveryResponsibilityDeclinedPermanently` | Content, Handoff |
| `DeliveryResponsibilityDeclinedTemporarily` | Content, Handoff |
| `DuplicateMessageReceived` | Content, Handoff |
| `ExpnSupportDisabled` | Diagnostics, Provisioning |
| `HeaderSectionAlteredFromDeducedEnvelopeRelationship` | Expansion, Abuse |
| `MailDataTransformedForLocalStorage` | Content, Handoff |
| `MailRejectedForPolicyReason` | Relaying, Abuse |
| `MailRejectedOnTraceHeaderFormat` | Trace, Gatewaying |
| `MailTransactionConfirmed` | Transaction, Content |
| `MessageContentStored` | Content, Handoff |
| `MessageForwardedSilently` | Forwarding, Abuse |
| `MessageRepairRefusedByRelay` | Relaying, Faults |
| `NextHopChannelEstablished` | Resolution, Relaying |
| `NotificationSuppressedForNullReturnPath` | Handoff, Notification |
| `NotificationSuppressedToPreventLoop` | Relaying, Notification |
| `NullAddressEventLoggedLocally` | Handoff, Notification |
| `NullReversePathSet` | Relaying, Notification |
| `OverflowReplyReclassifiedAsTransient` | Transaction, Faults |
| `OversizedMessageDataRejected` | Content, Faults |
| `PartialDeliveryFailureDetected` | Handoff, Notification |
| `ProtocolNameRegistered` | Trace, Abuse |
| `ReceivedLinePrependedByGateway` | Trace, Gatewaying |
| `RecipientBufferLimitReached` | Transaction, Faults |
| `RecipientLimitConfigured` | Faults, Provisioning |
| `RecipientLimitEnforcedBySitePolicy` | Transaction, Faults |
| `RecipientListCopiedIntoHeaderSection` | Expansion, Abuse |
| `RecipientRejectedBelowMandatoryMinimum` | Transaction, Faults |
| `RecipientRejectedWithoutAddressInformation` | Transaction, Forwarding |
| `RecipientValidationDeferred` | Transaction, Handoff |
| `RelayAccessRestrictedToKnownSources` | Relaying, Abuse |
| `RelayPolicyConfigured` | Relaying, Provisioning |
| `RelaySourceAuthorized` | Relaying, Provisioning |
| `RelayingDeclined` | Transaction, Abuse |
| `ReplyTextCustomized` | Faults, Provisioning |
| `RetryParametersConfigured` | Queueing, Provisioning |
| `ReturnPathDeletedByInboundGateway` | Trace, Gatewaying |
| `ReturnPathHeaderStrippedBeforeDelivery` | Trace, Handoff |
| `ReturnPathInsertedByOutboundGateway` | Trace, Gatewaying |
| `ReturnPathLineInserted` | Trace, Handoff |
| `ReversePathConstructedFromForeignEnvelope` | Trace, Gatewaying |
| `ServiceExtensionRegistered` | Provisioning, Abuse |
| `SizeLimitConfigured` | Faults, Provisioning |
| `SoftwareVersionAnnouncementDisabled` | Session, Provisioning |
| `SourceRouteStripped` | Transaction, Relaying |
| `SourceRouteStrippedFromReturnPath` | Handoff, Notification |
| `SourceRoutedAddressRefused` | Transaction, Relaying |
| `TextLineLengthExceeded` | Content, Faults |
| `TraceClauseRegistered` | Trace, Abuse |
| `TraceHeaderFieldRegistered` | Provisioning, Abuse |
| `UndeliverableMailNotificationSent` | Handoff, Relaying |
| `UnknownCommandRejected` | Session, Faults |
| `UserNameRecognitionRuleConfigured` | Diagnostics, Provisioning |
| `VerificationRestrictedToAuthenticatedRequestors` | Provisioning, Abuse |
| `ViaClauseRecorded` | Trace, Gatewaying |
| `VrfySupportDisabled` | Diagnostics, Provisioning |

**(b) Distinct names, same state change.** These are the harder failures, because deduplicating by
name does not catch them.

| Colliding events | Why they are one fact | RFC |
|---|---|---|
| `MessageAcceptedForProcessing`, `DeliveryResponsibilityAccepted`, `ResponsibilityForMessageAccepted` | All three are the server answering `2yz` to `<CRLF>.<CRLF>`. Three names, one reply | §4.1.1.4 lines 2045-2047, §4.2.5 lines 2943-2945, §6.1 lines 3962-3964 |
| `EndOfMailDataIndicated`, `MailTransactionConfirmed` | The RFC states outright that the same indicator does both | §3.3 lines 1116-1118 |
| `BufferCleared`, `StateTableReset` | One `RSET`, split in two because the RFC's sentence names two nouns | §3.3 lines 1029-1031 |
| `TransactionBuffersCleared`, `MailDataBuffersCleared` | Same three buffers, same clearing, different chapter | §4.1.1.4 lines 2038-2040 |
| `MailLoopDetected`, `MessageLoopDetected` | Same conclusion from the same `Received:` count | §6.3 lines 4067-4072 |
| `ReturnPathRemovedInTransit`, `ReturnPathRemoved` | Meanings are word-for-word identical in the roster | §4.4 lines 3226-3231 |
| `ReversePathRebuiltForOnwardHop`, `MailCommandRebuiltAfterReturnPathRemoval` | One sentence, one act: "remove the return path and rebuild the MAIL command" | §4.4 lines 3229-3231 |
| `MultipleReturnPathsPresentAtDelivery`, `MessageDeliveredWithAmbiguousReturnPath` | Same discovery at delivery | §4.4 lines 3240-3243 |
| `ReturnPathSubmittedByOriginatingSystem`, `MessageOriginatedWithReturnPathHeaderPresent` | Same `SHOULD NOT`, same violation | §4.4 lines 3233-3234 |
| `NotificationSentWithNonNullReturnPath`, `NotificationSentWithNonNullReversePath` | Differ only in ReturnPath versus ReversePath | §3.6.3 lines 1505-1509 |
| `UndeliverableMailNotificationSent`, `NonDeliveryNotificationSent` | The RFC states the obligation twice; the sweeps named it twice | §3.6.3 lines 1494-1499, §6.1 lines 3970-3972 |
| `NotificationSentAboutNotificationFailure`, `NotificationLoopGenerated`, `ErrorMessageSentInResponseToErrorMessage` | Three names for the one forbidden act | §3.6.3 lines 1505-1506, §4.5.4 lines 3682-3683 |
| `BlindCopyRecipientDisclosedInTraceField`, `BlindCopyRecipientDisclosedInForClause` | Same sentence, same disclosure | §7.6 lines 4327-4330 |
| `InternalHostNameDisclosedInTraceField`, `InternalHostNamesDisclosedInTraceField` | Differ by a plural | §7.6 lines 4321-4326 |
| `PreviouslyAcceptedRecipientsSilentlyDiscarded`, `PreviouslyAcceptedRecipientsDiscarded` | Differ by one word; worse than an exact duplicate, because deduplication misses it | §4.5.3.1.8 lines 3544-3547 |
| `TraceLinePrepended`, `ReceivedHeaderFieldAddedByRelay`, `ReceivedLinePrependedByGateway` | One obligation on every receiving server, split three ways by actor | §4.4 lines 3158-3161 |
| `MessageRelayedByDesignatedExchanger`, `MailRelayedOnward` | Same onward transfer | §5.1 lines 3909-3911, §3.6.3 lines 1494-1495 |
| `MessageDeliveredFinallyByDesignatedExchanger`, `FinalDeliveryMade` | Same final delivery | §5.1 lines 3911-3912, §4.4 lines 3204-3209 |
| `MessageHandedOffOutsideSmtpEnvironment`, `MessageForwardedToAnotherMailSystem` | Same hand-off out of the transport environment | §5.1 lines 3912-3913, §4.4 lines 3209-3212 |
| `MessageQueuedForDelivery`, `MessageQueuedForSubsequentDelivery` | Same queue entry, two chapters, two names | §4.5.4.1 lines 3689-3693 |
| `MessageRequeued`, `MessageQueuedAfterTemporaryDnsError` | The second is the first with a cause attached; a cause is not a different state change | §5.1 lines 3841-3842 |
| `SmtpServiceStarted`, `SmtpListenerStarted` | One pending listen on port 25 | §4.5.4.2 lines 3776-3778 |
| `TimeoutPolicyConfigured`, `TimeoutValuesReconfigured` | Same configuration act | §4.5.3.2 lines 3612-3615 |
| `SelectiveAcceptancePolicyAdopted`, `MailAcceptancePolicyConfigured` | Same standing decision | §7.9 lines 4357-4365 |
| `VerificationFalselyClaimed`, `VerificationFakedAfterSyntaxOnlyCheck` | Both are the `250`-after-syntax-check violation | §7.3 lines 4247-4248 |
| `VerificationDeniedWithoutChecking`, `VerificationAlwaysRefusedWith550` | Both are the always-`550` non-conformance | §7.3 lines 4249-4251 |
| `VerificationDeclinedAsDisabled`, `VerificationDeclinedWithoutDisclosure` | Both are the `252` a disabling site `MUST` return | §7.3 lines 4243-4245 |
| `VrfySupportDisabled`, `ExpnSupportDisabled`, `VerificationCommandDisabled` | The third is a third name spanning the first two | §3.5.2 lines 1359-1362 |
| `AddressDiagnosticsRestrictedToAuthenticatedRequestors`, `VerificationRestrictedToAuthenticatedRequestors` | Same sentence | §7.3 lines 4270-4272 |
| `ServerIdentityDisclosurePolicySet`, `ServerIdentityDisclosureConfigured`, `SoftwareVersionAnnouncementDisabled` | Three names for one disclosure setting | §3.1 lines 964-969, §7.5 lines 4294-4306 |
| `AddressesHarvestedViaExpn`, `AddressesHarvestedFromMailingList` | Same harvest | §7.3 lines 4253-4264 |
| `MailingListAccessRestricted`, `MailingListProtectionInstalled` | Same protection | §7.3 lines 4263-4265 |
| `MailingListExpandedAtSource`, `DuplicateEliminationBySourceExpansionAttempted`, `DuplicateRecipientEliminatedBySourceExpansion` | One discouraged strategy, three names, two chapters | §3.5.4 lines 1410-1417 |
| `PseudoMailboxExpanded`, `AliasExpanded`, `ListExpanded` | The generic plus its two specializations; every real expansion emits the generic and one specific | §3.9 lines 1698-1708, §3.9.1, §3.9.2 |
| `ListExpanded`, `ReturnPathRewrittenToListAdministrator` | The roster's own meaning says the retarget happens "in the same act" | §3.9.2 lines 1721-1726 |
| `ListReclassifiedAsFullMua`, `DeliveryAcceptedByList`, `MessagePostedByList` | The reclassification *is* the conjunction of the other two | §3.9.2 lines 1743-1746 |
| `HeaderSectionModifiedByExpander`, `MessageModifiedByList` | Same prohibited modification | §3.9 lines 1694-1696 |
| `ExistingTraceLinePreserved`, `MessageContentPassedThroughUnmodified` | The second contains the first | §4.4 lines 3178-3182, §3.6.3 lines 1524-1529 |
| `ReceivedLineChainAltered`, `ExistingReceivedLineAlteredByGateway`, `MessageDataModifiedByRelay` | One `MUST NOT`, three actors, three names | §4.4 lines 3178-3180 |
| `PartialFailureSignaledInDataReply`, `RecipientFailureReportedAfterContentAccepted` | Same reply on the same wire | §3.3 lines 1144-1150 |
| `RecipientRejectedAsUndeliverable`, `RecipientRejectedWithoutAddressInformation` | Same `550` at `RCPT` | §3.3 lines 1071-1074, §3.4 lines 1200-1204 |
| `TransactionCommandArgumentRejected`, `CommandRejectedAsMalformed` | Both are the `501` with state unchanged | §4.1.4 lines 2514-2516 |
| `CommandRejectedAsOutOfSequence` versus `RecipientCommandRejectedOutOfSequence`, `DataCommandRejectedOutOfSequence`, `DiagnosticCommandRejectedAsOutOfSequence`, `NestedMailCommandRejected`, `MailCommandRejectedForUninitializedSession`, `InterveningCommandRefusedAfterSessionRefusal` | A generic `503` event plus six specializations of it — the largest overlap cluster in the roster | §4.1.4 lines 2516-2528, §3.3 lines 1088-1089, §3.1 lines 977-979 |
| `UnknownCommandRejected`, `RequiredCommandRejectedAsUnrecognized` | Both are the `500`; the second adds only that the verb was one of the nine | §4.2.4 lines 2936-2937, §4.5.1 lines 3380-3392 |
| `MessageRejectedForHeaderDefects`, `MalformedMessageRejected` | Same refusal | §3.3 lines 1154-1158, §6.4 lines 4079-4085 |
| `MessageRejectedAtEndOfData`, `MessageRejectedForIncompleteTransaction`, `MessageRejectedByPolicy`, `DeliveryResponsibilityDeclinedPermanently` | Four names for the `5yz` at end-of-data, distinguished only by the reason | §3.3 lines 1137-1142, §4.2.5 lines 2981-2983 |
| `NullReversePathDeclared`, `NullReversePathSet` | `MAIL FROM:<>` on the wire, named once from the client side and once from the notifier's | §3.6.3 lines 1507-1522 |
| `UnsentMessageDiscardedWithoutQueuing`, `UndeliverableMailSilentlyDiscarded`, `MessageSilentlyDropped`, `BounceSuppressedForHostileContent` | Four overlapping silent-drop events | §4.5.4.1 lines 3689-3691, §6.2 lines 4015-4019 |
| `DeliveryGivenUp`, `DeliveryDeterminedImpossible`, `MessageReturnedAsNonDeliverable` | Three names around one give-up | §4.5.4.1 lines 3709-3711, §3.6.3 lines 1494-1496 |
| `MessageIntroducedIntoTransportEnvironment`, `MessageHandedFromUserAgentToMta`, `MessageSubmittedForDistribution` | Three names for the first hand-off | §2.3.10 lines 805-807, §3.6.3 lines 1481-1483 |
| `ForwardingAliasProvisioned`, `MailboxRelocationRecorded`, `MailForwardingRuleCreated`, `ReplacementAddressAssociatedWithMailbox` | One provisioning act; the RFC names two *motives* for it, not two mechanisms | §3.4 lines 1164-1167 |
| `MessageForwardedSilently`, `MessageRelayedToCorrectedAddress` | Same onward send after a `250` | §3.4 lines 1191-1194 |
| `SizeRestrictionImposedWithoutSizeExtension`, `SizeServiceExtensionImplemented`, `SizeLimitConfigured` | The first is the negation of the second combined with the third | §4.5.3.1.7 lines 3523-3526 |
| `EqualPreferenceRandomizationSkipped`, `CandidateAddressesTriedOutOfOrder` | The second's meaning explicitly includes the first | §5.1 lines 3883-3896 |
| `RoutingErrorReported` versus `TargetDomainFoundNonExistent`, `NoUsableMailExchangerFound`, `MailExchangerTargetFoundUnresolvable` | The roster's own meaning says it covers "three separate branches" — it is their union | §5.1 lines 3839-3852 |
| `ReplyIssued` versus roughly 120 reply-bearing events | A generic event whose semantics contain most of the roster. The most severe overlap present | §4.2 lines 2548-2551 |

**(c) One misclassification found while checking (b).** `ExpnServiceExtensionRegistered` is placed
in the Diagnostics chapter on the Extension Registry stream with actor "Receiving SMTP Server," and
its meaning is "EXPN entered the set of extensions this server is obliged to advertise." That is
not a registration — §3.5.2 lines 1364-1366 says only that "if EXPN is supported, it MUST be listed
as a service extension in an EHLO response." The event is `ExpnSupportEnabled` (Provisioning) under
a registry name, with an actor no registry could have. Both a duplicate and a naming defect.

### "Each event can be traced back to a specific actor in the Role Catalog" — **FAIL, and partly unrunnable**

The Role Catalog is a separate sub-step's artifact and is not in hand here, so the traceability
direction cannot be run in full. What *can* be run is the converse: which roster events carry an
actor label that no Role Catalog could legitimately hold, because the label is not a role. Those
fail regardless of what the catalog turns out to contain.

| Event | Actor as labeled | Why it fails |
|---|---|---|
| `ConnectionLostWithoutQuit` | network | A network initiates nothing. There is no role here, and §4.1.1.10 lines 2215-2219 treats the loss as a condition, not an act |
| `ExtensionFallbackTimeoutElapsed` | Queue timer | A clock is not a role. The comparable timeout events — `GreetingTimeoutExpired`, `CommandTimeoutExpired`, `ServerInactivityTimeoutExpired`, `QueueActivityTimedOut` — correctly attribute the expiry to the party that gave up |
| `MessageBouncedBeforeFallbackAttempted` | Queue timer | Same defect as above |
| `AcceptedMessageLost` | Receiving host | A host is not a role, and a loss has no initiator by construction. §6.1 lines 3964-3968 names the crash, not an actor |
| `BlindCopyRecipientDisclosed` | Message recipient | The recipient reads; the disclosure was caused by whoever wrote the address where it could be read. Also fails the queries-are-not-events test |
| `SlowRelayDetected` | Operator or postmaster reading trace data | A reader, and the act is a comparison over `Received:` timestamps. §4.4 lines 3184-3186 is about comparability, not about a state change |
| `SpoofedOriginDetected` | Expert examiner (human or downstream analyzer) | Textually grounded — §7.1 lines 4163-4166 does say "cannot be detected by an expert" — but the actor sits outside every system boundary the catalog will draw |
| `AddressBookPoisonedByForgedReply` | Man in the middle, through the client | "Through the client" concedes the point: the attacker is the cause, the client is the actor. §7.4 lines 4286-4290 |
| `BlindCopyRecipientDisclosedInTraceField` | Receiving SMTP server (as the unwitting cause) | The parenthetical states that the labeled party did not initiate it |
| `MultipleReturnPathsPresentAtDelivery` | Delivery SMTP server (as the party that discovers it) | A discoverer, not an actor. Same shape in `PartialDeliveryFailureDetected` |
| `HostileContentDetected` | Receiving site (mechanism unspecified by this RFC) | A site is not a role, and the label admits the mechanism is undefined |
| `ServiceExtensionNegotiated` | SMTP client and server jointly | Joint actor. The catalog form assigns one actor per event |
| `TransportLevelAuthenticationCompleted` | SMTP client and SMTP server together | Joint actor |
| `BilateralAgreementEstablished` | Operators of both systems, out of band | Joint actor, and out of band besides |
| `ConnectionClosedInViolation` | remote SMTP server (peer) | Not obviously a distinct role from SMTP server; the catalog must rule on whether peer-side roles are separate entries or the same role observed from the other end |

Two further findings on this item, neither of them a per-event failure:

- **The catalog must reach well past the SMTP session or more than twenty events lose their actor.**
  IANA, IANA registrant, IESG, domain administrator / DNS zone operator, implementer, site
  operator, mailbox owner, and message store all appear as actors. They are legitimate system actors
  under the kit's definition, but none of them is a party to any SMTP connection.
- **The two-ends split is systematic and needs a ruling, not case-by-case judgment.**
  `ServiceUnavailableAnnounced` / `ShutdownNoticeReceived`, `EndOfMailDataIndicated` /
  `EndOfMailIndicatorRecognized`, and `TransmissionChannelOpened` / `NextHopChannelEstablished` are
  each one wire fact recorded twice because two actors touched it. Whether that is legitimate
  two-actor modeling or duplication decides about a dozen roster entries at once.

### "All known error and boundary conditions have corresponding events" — **FAIL**

Named error and boundary conditions in RFC 5321 with no event anywhere in the roster. Each was
found by reading the section, not by pattern-matching the roster.

| Condition | RFC | Missing event |
|---|---|---|
| Local-part longer than 64 octets | §4.5.3.1.1 lines 3486-3487 | Five of the seven size minimums have events; local-part has none |
| Domain name longer than 255 octets | §4.5.3.1.2 line 3491 | None |
| Reply line longer than 512 octets | §4.5.3.1.5 lines 3506-3508 | None, and this is the only ceiling whose victim is the client. `OversizedCommandLineRejected` has no counterpart |
| A server that supports no service extensions at all, "in violation of this specification" | §4.1.1.1 lines 1815-1817 | None. The RFC names the violation explicitly, and every comparable named violation does have an event — `RequiredCommandLeftUnimplemented`, `NonConformingRecipientLimitConfigured`, `ShortCommandLineRejectedAsTooLong` |
| An `EHLO` response missing keywords for implemented non-required commands | §4.1.1.1 lines 1874-1877 | None. The exact mirror, `UnimplementedCapabilityAdvertised`, is present |
| A client transmitting `MAIL` or `RCPT` parameters the server never offered | §4.1.1.3 lines 1979-1981 | None. `MailParametersRejected` is the server's `555` / `455`; the client-side `MUST NOT` has no violation event |
| A system failing to preserve the case of a mailbox local-part | §2.4 lines 869-871 | None. The `MUST` is explicit and the roster records no violation |
| A server requiring command verbs in upper case, "in violation of this specification" | §2.4 lines 877-880 | None |
| A client transmitting high-bit octets with no negotiated extension | §2.4 lines 903-906 | None. `HighOrderBitCleared` is the server's permitted reaction; the client's `MUST NOT` has no event |
| Delivery of a severely garbled message, or its rejection, after mislabeled content crosses a path that cannot carry it | §2.4 lines 908-914 | None |
| Non-ASCII envelope commands sent by the client | §2.4 lines 916-919 | Partly covered by `CommandRejectedAsMalformed` on the server side; the sending act has no event |
| `8BITMIME` requested for non-MIME high-bit material | §2.4 lines 927-929 | None. Extension-specific, so arguably out of scope, but it is a named `MUST NOT` in the base document |
| The RCPT timeout that must be lengthened when list and alias processing is not deferred | §4.5.3.2.3 lines 3635-3636 | None. `ExpansionDeferredUntilAfterAcceptance` records the configuration; the timeout consequence is folded into the generic `CommandTimeoutExpired` |
| The six per-command timeouts as distinct boundaries | §4.5.3.2.2 through §4.5.3.2.7, lines 3631-3669 | `CommandTimeoutExpired` conflates `MAIL`, `RCPT`, DATA-initiation and data-block; only the `220` and DATA-termination timeouts have their own events |
| User review and intervention after a `5yz` at end-of-data | §4.2.5 lines 2985-2987 | None. `QueuedMessageReturnedToUser` covers the `4yz` branch only |

Two conditions that look like gaps and are not, checked and cleared: the premature-close obligation
to cancel the pending transaction while leaving completed ones intact (§4.1.1.10 lines 2216-2218)
is carried by `PendingTransactionCanceled`; the relay prohibition on applying submission repairs
(§6.4 lines 4140-4141) is carried by `MessageRepairRefusedByRelay` with `MessageDataModifiedByRelay`
as its violation.

---

## Correction recorded in this session

⚠️ **A finding was drafted and then withdrawn on reading the source.** While checking chapter
Gatewaying, I noted that `MessageGatewayedFromHeaderFieldsAlone` claims "which the specification
forbids," and that §7.2 lines 4214-4235 — the only "Blind" Copies discussion in the body — contains
no prohibition and no mention of a BCC header field. The draft finding was that three Expansion
events (`BlindCopyRecipientCopiedFromBccHeaderField`, `BccHeaderFieldRemoved`,
`EmptyBccHeaderFieldInserted`) and one Gatewaying event were ungrounded imports from RFC 5322.

That was wrong, and grepping the file rather than trusting the section index is what caught it. All
four are grounded in the appendices: Appendix B lines 4737-4745 specifies the BCC handling verbatim,
including inserting an empty BCC header field when stripping leaves none; and Appendix B lines
4773-4777 states the prohibition outright — "A submission protocol based on Standard RFC 822
information alone MUST NOT be used to gateway a message from a foreign (non-SMTP) mail system into
an SMTP environment." `MessageRemailedIntoOriginatingEnvironment` is likewise grounded, at
Appendix B lines 4784-4786.

The lesson is repository rule 3 in its own idiom: a section-heading index is not the document. Four
roster events came very close to being reported as fabrications because the grounding sits in an
appendix that the heading grep used to navigate the file does not surface as a numbered section.
