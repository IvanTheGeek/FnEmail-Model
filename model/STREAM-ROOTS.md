# Step 1 sub-step 2 — Event Streams (Stream Roots) for RFC 5321

## What this is

Step 1 sub-step 2 of Dymitruk's Brain Storming: **Identify Event Streams (Stream Roots)**, run over the whole of RFC 5321. The kit's instruction for this sub-step is at
`/home/ivan/event-modeling-research/research/archive/eventmodelers-kits/agent-modeling-kit/skills/eventmodeling-brainstorming-events/SKILL.md`,
lines 424–432 — cited, not reproduced (rule 7). It asks for three things per root: a **Name** in domain language, an **Identity key**, and the commands that will affect it. It is explicit that roots are not DDD aggregates but the logical roots of events, and elsewhere the kit's strongest DON'T forbids the DDD Aggregate pattern for state design. Nothing below reaches for aggregate boundaries, invariants, or consistency scopes; roots are chosen by **what the RFC keys, what has a lifetime, and where the buffer lives**.

Every section and line number below was found in `/home/ivan/FnEmail-Model/docs/rfc/rfc5321.txt` before it was cited (rule 3). Line numbers are real lines in that file.

---

## The headline finding

**RFC 5321 supplies exactly one identity key for any entity it owns: the mailbox address.**

§2.3.11, lines 828–835: "an 'address' is a character string that identifies a user to whom mail will be sent or a location into which mail will be deposited. The term 'mailbox' refers to that depository." And it comes with an equality rule, which is what makes it a real key rather than a label — §2.4, lines 866–873: the local-part "MUST BE treated as case sensitive", mailbox domains "follow normal DNS rules and are hence not case sensitive". A key plus a comparison rule is an identity key in the kit's sense.

Nothing else has one. Not the session, not the transaction, not the message, not the queue entry, not the delivery. Each of those is a thing with a birth, a death and accumulated state — and RFC 5321 never names an identifier for any of them.

The nearest miss is worth recording precisely, because it is the kind of thing a modeler remembers wrongly. The `Received:` trace line has an `ID` clause — §4.4, line 3336: `ID = CFWS "ID" FWS ( Atom / msg-id )`. It is **optional twice over**: optional in the grammar, because line 3329 has `Opt-info = [Via] [With] [ID] [For]` with `ID` bracketed; and unconstrained in content, because it may be a bare `Atom` rather than a `msg-id`. So the one per-message-per-hop handle the protocol offers is neither required nor required to be unique.

The consequence shows up in the protocol's own workarounds. Loop detection cannot follow an identifier, so §6.3, lines 4067–4072 counts `Received:` header fields instead — "Simple counting of the number of 'Received:' header fields in a message has proven to be an effective, although rarely optimal, method of detecting loops". That is what a keyless stream forces on you.

**This does not disqualify those roots.** The kit's criterion is "the logical roots of events", not "roots the specification printed a key for". The finding is recorded per root as *implementation-supplied*, and it is the single most consequential structural fact this sub-step produced.

---

## The catalog

Twenty-one roots, in five tiers. Tier E roots are outside RFC 5321's authority and are named as such rather than dropped.

### Tier A — protocol runtime

| Root | RFC name and citation | Identity key | Lifetime |
|---|---|---|---|
| SMTP Session | "SMTP session", §3.1, line 961 | **None supplied.** §2.3.6, line 743 models the state as a virtual "buffer" and "state table" and never keys it | TransmissionChannelOpened → TransmissionChannelClosed or ConnectionLostWithoutQuit |
| Mail Transaction | "mail transaction", §3.3, line 1015; §4.1.1, lines 1765–1775 | **None supplied.** Ordinal within session only | MailTransactionStarted → MailTransactionConfirmed / MailTransactionAborted |
| Accepted Message | no noun; the act is at §6.1, lines 3962–3964 | **None supplied.** `ID` clause, §4.4 line 3336, is the near miss | The `250` to \<CRLF>.\<CRLF> → delivered, relayed, or notified |
| Delivery Obligation | "for each failed recipient", §4.4, line 3273 | **None supplied.** Compound: accepted message plus forward-path | Acceptance → deposited in mailbox, handed to MUA, or notification sent |
| Mail Object | "mail object", §2.3.1, lines 610–611 | **None supplied.** Trace chain is the de facto identity | MessageIntroducedIntoTransportEnvironment (§2.3.10, line 806) → final delivery (§4.4, line 3209) |
| Queue Entry | "mail queue entry", §4.5.4.1, line 3692 | **None supplied.** Scope is message plus destination | MessageQueuedForDelivery → QueuedMessageTransmitted or DeliveryGivenUp |
| Transmission Attempt | "attempt", §4.5.4.1, line 3699; §5.1, line 3862 | **None supplied** | DeliveryDomainLexicallyIdentified → transmitted, or CandidateAddressesExhausted |
| Outbound Queue | "areas for queuing messages in transit", §4.5.4, lines 3673–3675 | The sending host | Unbounded |
| Next-Hop Destination | "destination", "target host", §5.1, line 3826 | The delivery domain name — **but see the key conflict below** | Unbounded |
| SMTP Service Instance | no noun; §4.5.4.2, lines 3776–3778 | Host plus port 25 | SmtpServiceStarted → ServiceForciblyShutDown |
| Envelope Recipient | "forward-path", §3.3, line 1057 | **None supplied**, and the forward-path string is not even unique — nothing forbids the same one twice | Provisional; see resolved question Q1 |

**Next-Hop Destination carries two different keys.** §5.1, line 3826 keys the destination by the **domain** the client "lexically identifies". §4.5.4.1, line 3716 keys durable unreachability state by the **host**: "A client SHOULD keep a list of hosts it cannot reach and corresponding connection timeouts." A domain resolves to many hosts and a host serves many domains. The roster's single Next-Hop Destination root is two roots wearing one name, and the split falls exactly between the resolution events and the reachability events.

### Tier B — addressing

| Root | RFC name and citation | Identity key | Lifetime |
|---|---|---|---|
| Mailbox | "mailbox", §2.3.11, lines 828–835 | **local-part@domain — the only RFC-supplied key**, with its case rule at §2.4, lines 866–873 | Provisioned → retired; outlives every message |
| Mailing List | "pseudo-mailbox", §3.9, lines 1698–1709 | The pseudo-mailbox address — same key space as Mailbox | MailingListDefined → undefined |
| Served Domain | "the domains for which the SMTP server provides mail service", §4.5.1, lines 3397–3399 | The domain name | MailServiceDomainAdded → removed |

Mailbox and Mailing List share a key space and are told apart only by expansion rules — §3.9, lines 1706–1709: "We classify such a pseudo-mailbox as an 'alias' or a 'list', depending upon the expansion rules." Two roots, one key space, with the discriminator being behavior rather than identity. Recorded, not resolved.

`postmaster` is the one reserved instance: §4.5.1, lines 3395–3402 requires it as a case-insensitive local name at every served domain, plus the bare `RCPT TO:<Postmaster>` form that ordinary path syntax would not permit (also §2.3.5, lines 735–737).

### Tier C — policy and build

| Root | RFC name and citation | Identity key | Lifetime |
|---|---|---|---|
| SMTP Service Policy | "site", §7.9, line 4357; "installations", §5.1, line 3888; "configuration mechanisms", §3.4, lines 1201–1204 | **None; the RFC has no configuration entity at all.** The installation | Unbounded |
| Implementation Release | "implementation", §4.5.1, lines 3380–3392; "implementations SHOULD provide the capability", §7.9, lines 4364–4365 | Software product plus version — made a disclosable fact by §3.1, lines 964–967 | Ships → superseded |

The roster's Server Configuration root holds over sixty events and **is not a root**. It has no identity key even in principle: VrfySupportDisabled and MailboxRelocationRecorded are not facts about the same thing at any granularity, so no key could serve both. It is a junk drawer. The split above sends site and installation policy to SMTP Service Policy, build-time conformance facts to Implementation Release, and domain-, mailbox- and list-scoped facts out to Tier B.

Implementation Release is a real distinction, not tidiness. MinimumCommandSetProvisioned, RelayFilteringCapabilityImplemented and QueuingStrategyAdopted are facts about a *build*; RequiredCommandLeftUnimplemented makes a receiver "non-conformant from the moment it is deployed", which is a property of the artifact, not of any running server or any operator's choice.

### Tier D — boundary

| Root | RFC name and citation | Identity key | Lifetime |
|---|---|---|---|
| Transport Service Environment | "transport service environment", §2.3.10, lines 803–825; §3.7, lines 1533–1539 | **None supplied** | Unbounded |

### Tier E — outside RFC 5321

| Root | RFC name and citation | Identity key | Note |
|---|---|---|---|
| SMTP Service Extensions Registry | §8, lines 4390–4398; §2.2.2, lines 534–571 | EHLO keyword value | IANA's |
| Address Literal Tags Registry | §8, lines 4400–4405 | The tag | IANA's |
| Mail Transmission Types Registry | §8, lines 4407–4438 | Link or protocol name; plus Additional-registered-clauses names | IANA's |
| Trace Header Field Registry | §8, lines 4444–4447 | Header field name | **BCP 90's, not RFC 5321's** — the RFC names only three registries of its own |
| DNS Zone | §5.1, line 3828 delegates to RFC 1035 | Owner name plus RR type | No RFC 5321 citation supports its events |
| Correspondent Address Record | "the user's address book", §7.4, line 4288 | Correspondent address | Client-local; the RFC mentions it only to warn against trusting an unauthenticated `251` or `551` |

The roster's single Extension Registry root is **four registries**, and one of them belongs to a different document. That matters for the model: an event on the BCP 90 registry is not governed by RFC 5321's registration rules at all.

---

## Roots retired

| Provisional root | Verdict |
|---|---|
| Reply | Not a root. §4.2, line 2551: "Every command MUST generate exactly one reply." A reply has no life beyond its command, and its effect is on the client's state — line 2556: "The number is for use by automata to determine what state to enter next." Fold to SMTP Session |
| Address Diagnostic Request | Not a root — see Q5 |
| Relay Task | Not a root — see Q4 |
| Relay Host | Not a root. Its one event, SlowRelayDetected, is a conclusion derived by comparing timestamps across a trace chain (§4.4, lines 3184–3187). Read model |
| Notification Message / Notification | Not a root. A notification is a mail object distinguished by a field value — §4.5.5's whole title is "Messages with a Null Reverse-Path", lines 3787–3805. Fold to Mail Object |
| Server Configuration | Not a root — split as in Tier C |
| Address Book | Duplicate name for Correspondent Address Record |
| Message | Not one root — split into Mail Object and Accepted Message, see Q3 |
| Extension Registry | Not one root — four registries, see Tier E |

---

## Resolved modeling questions

### Q1 — Does a recipient's acceptance belong to a Recipient stream or the Transaction stream?

**Resolved: to the Transaction, up to the responsibility handoff. After the handoff there is a per-recipient stream, but it is a different root with a different lifetime.**

For Transaction:

- §4.1.1, lines 1771–1775: "the model for this is that distinct buffers are provided to hold the types of data objects; that is, there is a reverse-path buffer, a forward-path buffer, and a mail data buffer." The forward-path buffer is **one buffer, singular, per transaction**.
- §4.1.1.3, lines 1935–1938: `RCPT` appends to the forward-path buffer and "does not change the reverse-path buffer nor the mail data" buffer. Appending to a buffer is a state change on the thing that owns the buffer.
- §4.1.1.5, lines 2081–2083: `RSET` discards "Any stored sender, recipients, and mail data" — one act, all recipients at once. No per-recipient discard exists.
- §4.5.3.1.8, line 3537: "The minimum total number of recipients that MUST be buffered is 100." A ceiling on a collection is a property of the collection's owner.
- The RFC supplies no recipient identifier at all, and does not forbid the same forward-path appearing twice in one transaction.

For a per-recipient stream:

- §4.4, lines 3271–3273: "A single notification listing all of the failed recipients or separate notification messages MUST be sent for each failed recipient." Per-recipient fate, explicitly.
- §4.4, lines 3177–3179: "If the FOR clause appears, it MUST contain exactly one \<path> entry, even when multiple RCPT commands have been given." The trace line narrows to one recipient — the protocol itself splitting a transaction into per-recipient threads.
- §3.9.1, lines 1710–1716: alias expansion replaces one envelope recipient with several, each with its own delivery fate.
- 251 and 551 (§3.4, lines 1185–1199) are per-recipient outcomes with per-recipient consequences at the client.

The two bodies of evidence do not conflict; they describe **two different things at two different times**, and the boundary between them is the `250` to the end-of-data indicator. Before it, a recipient is a string in a buffer that is cleared wholesale (§4.1.1.4, lines 2036–2041). After it, a recipient is an obligation the server took on and must discharge or report (§6.1, lines 3968–3972).

So: Envelope Recipient is retired as a root for the in-session events, which go to Mail Transaction; and **Delivery Obligation** is introduced for the post-acceptance events, keyed by accepted message plus forward-path.

### Q2 — Is a Session a stream at all when the RFC names no session identifier?

**Resolved: yes. The missing identifier is a finding about the specification, not a disqualification.**

Against:

- No session identifier appears anywhere in RFC 5321. §2.3.6, lines 741–746 goes as far as modeling the session's state — a "buffer" and a "state table" — and still names no key for the session that holds them.
- The only available identity is the transmission channel, and §1.1, lines 236–238 deliberately holds the transport at arm's length: "SMTP is independent of the particular transmission subsystem and requires only a reliable ordered data stream channel." The one key on offer comes from a layer the specification refuses to depend on.

For:

- §3.1, lines 961–962: "An SMTP session is initiated when a client opens a connection to a server and the server responds with an opening message." A named birth.
- §3.8, lines 1642–1644: "An SMTP connection is terminated when the client sends a QUIT command." A named death.
- §4.1.4, lines 2506–2507: "There may be zero or more transactions in a session." The session demonstrably outlives the transaction, so it holds state across events of its own.
- §4.1.4, lines 2453–2456: a later EHLO "MUST clear all buffers and reset the state exactly as if a RSET command had been issued" — a rule that is only expressible if a session-scoped state exists to be reset.

The kit asks for a logical root of events with an identity key. RFC 5321 gives the first unambiguously and withholds the second. **Record the key as implementation-supplied and move on** — the same verdict, for the same reason, applies to Mail Transaction, Accepted Message, Mail Object, Queue Entry, Transmission Attempt and Delivery Obligation.

### Q3 — Where does a message's stream begin relative to the responsibility handoff?

**Resolved: the handoff does not begin a message's life — it begins a *custody* stream on a message that already existed. And on the receiving server there is no message stream before it, because the content lives in a transaction buffer that gets cleared.**

The decisive text is §4.1.1.4, lines 2036–2041: "Receipt of the end of mail data indication requires the server to process the stored mail transaction information. This processing consumes the information in the reverse-path buffer, the forward-path buffer, and the mail data buffer, and on the completion of this command these buffers are cleared." The mail data buffer is **per transaction and destroyed**. Whatever persists at the receiving server is what acceptance created, not what the buffer held.

And acceptance is a distinct, named act — §6.1, lines 3962–3964: "When the receiver-SMTP accepts a piece of mail (by sending a `250 OK` message in response to DATA), it is accepting responsibility for delivering or relaying the message"; §2.1, lines 419–424: "once the server has issued a success response at the end of the mail data, a formal handoff of responsibility for the message occurs".

There is a sharp instance that settles where the line falls. §3.3, lines 1097–1099: "When the end of text is successfully received and stored, the SMTP-receiver sends a `250 OK` reply." **Storing precedes the reply.** So MessageContentStored is a pre-handoff event and belongs to Mail Transaction, and the boundary is the reply, not the storing. That single sentence disposes of the tempting reading that the message stream begins when content lands.

Hence two roots where the roster had one:

- **Mail Object** — the thing that crosses hops, is trace-stamped, is loop-detected, gets a Return-Path at final delivery. It begins where §2.3.10, lines 805–807 says an originating system "introduces mail into the Internet or, more generally, into a transport service environment", and ends where §4.4, lines 3208–3210 says "final delivery means the message has left the SMTP environment". One instance spans every hop.
- **Accepted Message** — one instance *per hop*, born exactly at the `250` to \<CRLF>.\<CRLF>, ending when the custodian delivers, relays, or notifies.

Content-shaping events split cleanly on the same line. On the receiving side, DotUnstuffingApplied, TextLineLengthExceeded and HighOrderBitCleared act on the mail data buffer, so they belong to Mail Transaction. On the sending side, DotStuffingApplied and TerminatingLineEndingAdded act on a message that already exists in the sender's hands, so they belong to Mail Object or Queue Entry. The roster put both sides on Message.

### Q4 — Is Relay Task a root distinct from the message in custody?

**Resolved: no. The RFC equates them itself.**

§3.6.2, lines 1449–1453: "The relay server may accept or reject the task of relaying the mail **in the same way it accepts or rejects mail for a local user**. If it accepts the task, it then becomes an SMTP client, establishes a transmission channel to the next SMTP server specified in the DNS." The task's acceptance *is* the responsibility acceptance of §6.1, seen from the relay's side; its lifetime is identical. Fold to Accepted Message.

The one salvage: RelayTaskDeclined and RelayDeclinedWithNonStandardCode happen at `RCPT` time (§3.6.2, lines 1454–1456), before any acceptance exists, so they belong to Mail Transaction rather than to the custody stream.

### Q5 — Is Address Diagnostic Request a stream?

**Resolved: no. It is the kit's query case.** `VRFY` and `EXPN` are lookups: §3.5.1, lines 1217–1219 — "SMTP provides commands to verify a user name or obtain the content of a mailing list." They change nothing. The kit's "guest viewed calendar for room availability" test applies squarely, and the request has no lifetime beyond its own reply anyway.

Its thirty-odd events are **not discarded** — they are reassigned by what actually changed:

- Where information left the system, the root is the thing that leaked: MailboxExistenceDisclosedViaVrfy and AmbiguousMailboxAlternativesDisclosed to Mailbox; MailingListMembershipDisclosedViaExpn and AddressesHarvestedViaExpn to Mailing List. §7.3, lines 4258–4262 treats harvesting as the consequence that matters.
- Where the server emitted a non-conforming reply, the root is Session: VerificationFalselyClaimed and VerificationDeniedWithoutChecking are the two behaviors §7.3, lines 4243–4252 names as violations, and each is a fact about one emitted reply.
- Where a site turned something off, the root is SMTP Service Policy.

The candidates that remain pure reads with no state change and no disclosure — the plain success and failure responses — are the set-aside pile for this sub-step, with "query, not event" as the reason.

---

## Open questions

**O1 — Is the session one stream or two?** RFC 5321 describes one session but assigns timeouts separately to each end: the client's per-command timeouts at §4.5.3.2, lines 3612–3618, the server's at §4.5.3.2.7, lines 3666–3669. GreetingTimeoutExpired and ServerInactivityTimeoutExpired are facts known to only one party, at different moments. A single Session stream forces two observers into one timeline that neither of them can actually see. Left open: the alternative is a Session root per endpoint, which doubles the root and needs a correlation that the RFC does not supply either.

**O2 — Is the trace chain the Mail Object's identity, or a read model over it?** For: it is the only thing that persists across hops and the RFC uses it as a surrogate identity for loop detection (§6.3, lines 4067–4072). Against: it is derived — each hop appends, and reading it is a projection over many hops' events, which is a read model by construction. Both readings survive the evidence.

**O3 — Is client policy a separate root from server policy?** RFC 5321 separates client and server obligations throughout, and ClientRecipientsPerTransactionLimitConfigured and ConcurrentOutboundTransactionLimitConfigured (§4.5.4.1, lines 3748–3756, 3758–3761) are client-side. But one installation runs both, so one key serves both. Provisionally one SMTP Service Policy root with the side as a field; flagged because the alternative is defensible.

**O4 — Does a Queue Entry per destination or per message?** §4.5.4.1, line 3692 says an entry "will include not only the message itself but also the envelope information" — reading as one entry per message. But line 3699 says "The sender MUST delay retrying a particular destination", and line 3731 speaks of "a large queue of messages for each unavailable destination host" — reading as one entry per message-destination pair. RecipientsGroupedIntoSingleTransaction and RecipientListSplitAcrossTransactions only make sense on the second reading. Unresolved in the text.

---

## Reassignment flags — every provisional stream_root I would change

Grouped by the move, event names as they appear in the roster.

**Recipient → Mail Transaction** (Q1: pre-handoff, the forward-path buffer is the transaction's)

RecipientDeclared · RecipientAccepted · PostmasterRecipientAccepted · RecipientRejectedAsUndeliverable · RecipientRejectedWithForwardingAddress · RecipientRejectedWithoutAddressInformation · RecipientRejectedForPolicy · RelayingDeclined · RecipientValidationDeferred · SourceRouteStripped · SourceRoutedAddressRefused · SourceRoutedAddressAccepted · SourceRouteHonored · ExplicitSourceRouteGenerated · InvalidSourceRouteGenerated · SourceRouteIgnoredByGateway · ForwardPathRewrittenBeforeRelay · MessageForwardedSilently · CorrectedDestinationDisclosed

**Recipient → Delivery Obligation** (post-handoff)

DeliveryFailedAfterAcceptance · DeliveryFailedAfterSourceRouteStripped

**Message → Mail Transaction** (Q3: the mail data buffer belongs to the transaction and is cleared at end of DATA)

MailDataAppended · ProblematicControlCharacterTransmitted · TextLineLengthExceeded · BareLineTerminatorTransmitted · BareLineFeedTerminationAccepted · HighOrderBitCleared · EightBitOctetsNormalized · DotStuffingApplied · DotUnstuffingApplied · EmptyMessageContentTransmitted · ExtraLineEndingAppended · MessageContentStored · MessageRejectedForHeaderDefects · MalformedMessageRejected · MailAcceptedDespiteMalformedTraceHeader · MailRejectedOnTraceHeaderFormat

**Message → Accepted Message** (custody at one hop)

DeliveryResponsibilityAccepted · ResponsibilityForMessageAccepted · AcceptedMessageCommittedToStableStorage · AcceptedMessageLost · DuplicateMessageReceived · RelayInspectedMessageData · MessageContentInspectedByRelay · MessageRelayedByDesignatedExchanger · MessageHandedOffOutsideSmtpEnvironment · MessageRelayedToCorrectedAddress · HostileContentDetected · MessageJudgedSeriouslyFraudulent · ReturnAddressDeterminedInvalid · MessageSilentlyDropped · MessageDroppedForInvalidReturnAddress · BounceSuppressedForHostileContent · BounceSentDespiteHostileContent · PartialDeliveryFailureDetected

**Message → Mail Object** (the thing in transit across hops — the whole of group 4 and group 9 except as noted)

TraceLinePrepended · FromClauseComposed · TraceSourceNameStampedWithoutConnectionAddress · FromClauseOmittedFromTrace · TraceRecipientNarrowed · TraceForClauseOmitted · MultiplePathsStampedInForClause · BlindCopyRecipientDisclosedInTraceField · BlindCopyRecipientDisclosedInForClause · InternalHostNameDisclosedInTraceField · InternalHostNamesDisclosedInTraceField · ReceivedLineChainAltered · ExistingTraceLinePreserved · TraceLineInsertedOutOfPosition · TraceDateStampedWithZoneNameInsteadOfOffset · ObsoleteDateFormUsedInTraceLine · UnregisteredTraceNameUsed · ReceivedLinePrependedByGateway · ReceivedHeaderFieldAddedByRelay · ViaClauseRecorded · ViaClauseOmittedByGateway · ExistingReceivedLineAlteredByGateway · ReceivedLineOmittedAtBoundaryCrossing · ReturnPathLineInserted · ReturnPathRemovedInTransit · ReturnPathRemoved · ReturnPathHeaderStrippedBeforeDelivery · ReturnPathSubmittedByOriginatingSystem · MessageOriginatedWithReturnPathHeaderPresent · MultipleReturnPathsPresentAtDelivery · MessageDeliveredWithAmbiguousReturnPath · ReturnPathLineOmittedAtFinalDelivery · ReturnPathInsertedByOutboundGateway · ReturnPathDeletedByInboundGateway · ReversePathConstructedFromForeignEnvelope · MailLoopDetected · MessageLoopDetected · LoopingMessageRejected · TrivialLoopStopped · LoopedMessagePassedOnUndetected · MessageIntroducedIntoTransportEnvironment · MessageHandedFromUserAgentToMta · MessageForwardedToAnotherMailSystem · MessageDeliveredAfterResponsibilityDeclined · MessageContentPassedThroughUnmodified · MessageDataModifiedByRelay · MessageRepairRefusedByRelay · RelayUsedToObscureOrigin · MessageSubmittedForDistribution · MessageCrossedTransportEnvironmentBoundary · MessageTransformedByGateway · AddressRewrittenByGateway · HeaderFieldRewrittenAtGateway · EnvelopeFoldedIntoHeaderSection · PrivateRecipientDisclosedByEnvelopeFolding · HeaderAddressTransformedForInternet · NonconformingHeaderAddressForwardedIntoInternet · ForeignErrorRoutedToReversePath · ForeignErrorRoutedToHeaderOriginator · EnvelopeReturnPathSetAtGateway · ReturnPathDefaultedToOriginatorAddress · MessageGatewayedFromForeignMailSystem · MessageGatewayedFromHeaderFieldsAlone · MessageRemailedIntoOriginatingEnvironment · LayeringSemanticsPreservedAcrossBoundary · EnvelopeInformationLostInTranslation · PseudoMailboxExpanded (see below) · ReturnPathRewrittenToListAdministrator · ListReturnPathLeftUnchanged · HeaderSectionModifiedByExpander · MessageModifiedByList · DeliveryAcceptedByList · MessagePostedByList · ErrorMessageReturnedToListAdministrator · ErrorMessageReturnedToOriginator · MailingListExpandedAtSource · DuplicateRecipientEliminatedBySourceExpansion · DuplicateEliminationBySourceExpansionAttempted · BccHeaderFieldRemoved · EmptyBccHeaderFieldInserted · RecipientListCopiedIntoHeaderSection · HeaderSectionAlteredFromDeducedEnvelopeRelationship · BlindCopyRecipientDisclosed · MessageIdAdded · OriginationDateAdded · AddressCorrectedToFqdn · RepairDocumentedInTraceHeader · MessageRepairRefusedByRelay · MalformedMessagePassedOnUnchanged · HeaderValueEncodedForAsciiTransport · TerminatingLineEndingAdded · MessageRejectedAsInvalidByOriginator · UnsentMessageDiscardedWithoutQueuing · MessageResubmittedToCorrectedAddress · MessageSpoofedToAppearFromElsewhere · EnvelopeReturnPathSetToAnotherUsersAddress · MessageCopyCreated

**Message → Delivery Obligation**

PseudoMailboxExpanded · AliasExpanded · ListExpanded · RecipientSuppressedByHeuristic · NotificationSuppressedForNullReturnPath · NullAddressEventLoggedLocally · NotificationSuppressedToPreventLoop · FaultyMessageDeliveredAnyway · MessageReturnedAsNonDeliverable · MessageDeliveredFinallyByDesignatedExchanger · FinalDeliveryMade · MessageHandedOffToMailUserAgent

**Message / Mailbox mixed → Mailbox**

MailDataTransformedForLocalStorage · IrreversibleStorageTransformApplied

**Server Configuration → SMTP Service Policy** (the whole of group 15 except the moves below, plus every Server-Configuration-tagged event in groups 1, 3, 4, 6, 7, 8, 10, 11, 12, 13, 14, 16)

SoftwareVersionAnnouncementDisabled · ConnectionRefusalConfigured · TimeoutPolicyConfigured · TimeoutValuesReconfigured · SizeRestrictionImposedWithoutSizeExtension · TraceTimeFormatConfigured · LoopRejectionThresholdConfigured · ServerIdentityNamesConfigured · AlternateAddressLimitConfigured · RetryParametersConfigured · RelayPolicyConfigured · RelayAccessRestrictedToKnownSources · RelaySourceAuthorized · RelayDomainAuthorized · SelectiveAcceptancePolicyAdopted · MailAcceptancePolicyConfigured · SubmissionArrangementEstablished · AddressExpansionSupportEnabled · AddressExpansionModelLeftUnsupported · ExpansionDeferredUntilAfterAcceptance · AddressUpdateDisclosureRestricted · ServerChosenForDisclosureBehavior · InvalidReturnAddressDropPolicyAdopted · PostmasterRedirectConfigured · VrfySupportDisabled · ExpnSupportDisabled · ExpnSupportEnabled · VerificationCommandDisabled · DisabledVerificationReplyCodeConfigured · VerificationRestrictedToAuthenticatedRequestors · AddressDiagnosticsRestrictedToAuthenticatedRequestors · UserNameRecognitionRuleConfigured · HelpSupportConfigured · ServerIdentityDisclosurePolicySet · ServerIdentityDisclosureConfigured · ReplyTextCustomized · SizeLimitConfigured · RecipientLimitConfigured · NonConformingRecipientLimitConfigured · ClientRecipientsPerTransactionLimitConfigured · ConcurrentOutboundTransactionLimitConfigured · MailProblemContactPointPublished · ServiceExtensionEnabled · ReturnPathAlterationRestricted · BilateralAgreementEstablished · ExpnServiceExtensionRegistered

**Server Configuration → Implementation Release**

MinimumCommandSetProvisioned · RequiredCommandLeftUnimplemented · DomainlessPostmasterSupportProvisioned · SizeServiceExtensionImplemented · RelayFilteringCapabilityImplemented · QueuingStrategyAdopted · NonStandardReplyCodeInvented · UnimplementedCapabilityAdvertised

**Server Configuration → Served Domain**

PostmasterMailboxProvisioned · MailServiceDomainAdded · PostmasterMailBlocked · PostmasterMailBlockLifted

**Server Configuration → Mailbox / SMTP Service Instance / Mailing List**

ProgramDeliveryMailboxNameAssigned → Mailbox · SmtpListenerStarted → SMTP Service Instance · ServerRenderedInaccessibleByAttack → SMTP Service Instance · AddressesHarvestedFromMailingList → Mailing List

**Extension Registry → the four named registries**

ServiceExtensionRegistered and ExtensionApprovedByIesg → SMTP Service Extensions Registry · AddressLiteralTagRegistered → Address Literal Tags Registry · TransmissionTypeRegistered, TraceClauseRegistered, ProtocolNameRegistered → Mail Transmission Types Registry · TraceHeaderFieldRegistered → Trace Header Field Registry (BCP 90's, not RFC 5321's) · BilateralAgreementEstablished → **not a registry event at all**: §2.2.2, lines 568–571 says keywords beginning with "X" "MUST NOT be used in a registered service extension" · ExpnServiceExtensionRegistered → **misnamed**; it is a server enabling `EXPN` and thereby owing its advertisement, not a registration

**Next-Hop Destination → Transmission Attempt** (per-attempt derivations, not durable destination state)

TargetNameResolvedToCanonicalName · TargetDomainFoundNonExistent · ImplicitMailExchangerAssumed · NoUsableMailExchangerFound · AddressRecordsUsedDespiteMailExchangersPresent · MailExchangerAddressResolved · MailExchangerTargetFoundUnresolvable · EqualPreferenceExchangersRandomized · EqualPreferenceRandomizationSkipped · SelfIdentifiedInMailExchangerList · LowerPreferenceExchangersDiscarded · DeliveryAttemptFailedForAddress · CandidateAddressesTriedOutOfOrder · NextHopChannelEstablished · NextHopChannelRefused · AddressFamilyCandidateSkipped · RoutingErrorReported · DeliveryRoutedWithoutDnsResolution · AlternateMailExchangerTried

**Next-Hop Destination → Queue Entry** (the clock is the message's time in queue, not the destination's — §4.5.4.1, lines 3727–3730)

DestinationRetriedWithoutDelay · DestinationBackoffWidened

**Queue Entry → Transmission Attempt**

DeliveryAttemptFailed · AlternateAddressLimitReached · CandidateAddressesExhausted · DeliveryAbandonedAfterSingleAddress

**Queue Entry → Delivery Obligation**

DeliveryDeterminedImpossible · UndeliverableMailSilentlyDiscarded

**Relay Task → Accepted Message** (Q4)

RelayTaskAccepted · SmtpClientRoleAssumed · MailRelayedOnward

**Relay Task → Mail Transaction**

RelayTaskDeclined · RelayDeclinedWithNonStandardCode

**Notification Message / Notification → Mail Object**

UndeliverableMailNotificationSent · NonDeliveryNotificationComposed · NonDeliveryNotificationSent · NotificationSentWithNonNullReversePath · NotificationSentWithNonNullReturnPath · NotificationLoopGenerated · NotificationSentAboutNotificationFailure · SourceRouteStrippedFromReturnPath · NullReversePathReplacedOnForward · AutoReplySentToNullReversePathMessage

**Notification → Delivery Obligation**

NotificationDeliveryFailed · FailedNotificationRedirectedToPostmaster

**Notification Message → Mail Transaction**

NullReversePathSet (it is the `MAIL` argument — §3.6.3, lines 1507–1510)

**Reply → Session**

MultilineReplyCompleted · InconsistentMultilineReplyEmitted · BareReplyCodeTolerated

**Address Diagnostic Request → Mailbox / Mailing List / Session** (Q5)

To Mailbox: MailboxExistenceDisclosedViaVrfy · AmbiguousMailboxAlternativesDisclosed · UserNameExpandedAsSingletonList · VerifiedAddressReportedAsForwarded (from Session)
To Mailing List: MailingListVerifiedAsUser
To Session: MailboxNameReportedAsAmbiguous · CrossTypeVerificationRefused · AddressVerificationDeclinedAsUnverifiable · VerificationDeclinedAsDisabled · VerificationRefusedAsUnimplemented · VerificationFalselyClaimed · VerificationDeniedWithoutChecking · FreeFormVerificationTextReturned · SourceRouteReturnedInVerificationReply

**Session → Mail Transaction** (a transaction command that leaves no transaction still touched none)

BufferCleared · DataTerminationTimeoutExpired

**Transaction → Session** (no transaction instance exists for the event to land on — §4.1.4, lines 2497–2499: a `501` to a transaction-beginning command leaves the server "in the same state")

TransactionCommandArgumentRejected · MailCommandRejectedForUninitializedSession

**Transaction → Delivery Obligation**

ReversePathResolvedAtDelivery (§3.6.1, lines 1441–1444) · PartialDeliveryFailureDetected

**Transaction → Accepted Message**

MessageAcceptedForProcessing (the birth event of that stream)

**Transaction → Queue Entry**

BlindCopySentAsSeparateTransaction (a sending-strategy choice about how to transmit one message — §7.2, lines 4224–4226)

**Session → SMTP Service Instance**

ConcurrentConnectionLimitReached (the ceiling is the instance's, and no session was ever taken up — §4.5.4.2, lines 3778–3782)

**Session / Message → Mail Object**

MessageSpoofedToAppearFromElsewhere

**Next-Hop Destination → Transport Service Environment**

ForeignTransportEnvelopeCapabilityRecorded

**Address Book → Correspondent Address Record**

AddressBookEntryUpdated · AddressBookPoisonedByForgedReply · AddressCorrectionAcceptedFromUnauthenticatedServer

**Message → Transmission Attempt**

CorrectedAddressFoundUnreachableBySender

**Split root — one event covering two**

GatewayDesignatedForTransportBoundary is currently Server Configuration. §3.7, lines 1533–1536 gives two mechanisms — "MX records or various forms of explicit routing". By MX it is a DNS Zone event; by explicit routing it is an SMTP Service Policy event. One event covering two roots is a sign it should be two events.

---

## Roster conflicts the sweeps left behind

The same event name carries different provisional roots in different groups. Each is a real disagreement, not a transcription slip.

| Event | Conflicting assignments | Proposed |
|---|---|---|
| OverflowReplyReclassifiedAsTransient | Queue Entry (group 2) vs Recipient (group 14) | Queue Entry — the effect is that addresses stay queued |
| DeliveryFailedAfterAcceptance | Recipient (group 5) vs Message (group 12) | Delivery Obligation — the exemplar for that root |
| PartialDeliveryFailureDetected | Transaction (group 5) vs Message (group 12) | Accepted Message |
| NullAddressEventLoggedLocally | Server Configuration (group 5) vs Message (group 12) | Delivery Obligation |
| MailAcceptedDespiteMalformedTraceHeader | Message (group 4) vs Transaction (groups 9, 14) | Mail Transaction |
| MailRejectedOnTraceHeaderFormat | Message (group 4) vs Transaction (group 9) | Mail Transaction |
| RecipientValidationDeferred | Recipient (groups 2, 5) — consistent, but the group 5 wording ties it to post-acceptance | Mail Transaction for the RCPT-time fact |

---

## Duplicate semantics — the kit forbids these, recorded not resolved

The eleven sweeps produced several sets of names for one state change. Under "Unique semantics, no duplicates" these must collapse, but which name survives is a sub-step 4 decision, not this one's.

- **The responsibility handoff has three names**: MessageAcceptedForProcessing, DeliveryResponsibilityAccepted, ResponsibilityForMessageAccepted. One state change, §6.1, lines 3962–3964.
- **The reverse-path arriving has three**: ReversePathDeclared, ReversePathBuffered, ReversePathAccepted — for what §4.1.1.2, lines 1895–1897 describes as one command clearing the buffers and inserting the reverse-path.
- **The recipient arriving has three**: RecipientDeclared, RecipientBuffered, RecipientAccepted.
- **The channel opening has two, one per end**: TransmissionChannelOpened and NextHopChannelEstablished are the *same channel* named from the two ends of it (§2.1, lines 362–364). This is the clearest artifact of running eleven independent sweeps.
- **Buffer clearing has three**: BufferCleared, TransactionBuffersCleared, MailDataBuffersCleared, plus StateTableReset alongside.
- **Loop detection has two**: MailLoopDetected and MessageLoopDetected.
- **Trace stamping has two**: TraceLinePrepended and ReceivedHeaderFieldAddedByRelay.
- **Relay inspection has two**: RelayInspectedMessageData and MessageContentInspectedByRelay.
- **Recipient overflow discard has two**: PreviouslyAcceptedRecipientsSilentlyDiscarded and PreviouslyAcceptedRecipientsDiscarded.
- **The null reverse-path has two**: NullReversePathDeclared and NullReversePathSet.
- **Service start has two**: SmtpServiceStarted and SmtpListenerStarted.
- **Verification disabling has three or four**: VrfySupportDisabled, ExpnSupportDisabled, VerificationCommandDisabled, and the paired VerificationRestrictedToAuthenticatedRequestors / AddressDiagnosticsRestrictedToAuthenticatedRequestors.
- **Queueing has two**: MessageQueuedForDelivery and MessageQueuedForSubsequentDelivery.
- **ServiceExtensionNegotiated** restates ServiceExtensionsAdvertised as a summary of the same exchange.

---

## Derived-value and query flags (the kit's rules, applied)

- **SlowRelayDetected** — a conclusion from comparing timestamps across a trace chain. §4.4, lines 3184–3187 frames comparability as being *for detecting* problems. Read model. Its root, Relay Host, dies with it.
- **SpoofedOriginDetected** — a judgment, and §7.1, lines 4163–4166 says only that detection by an expert is "somewhat more difficult". Derived.
- **ListTreatedAsTransitContinuation / ListReclassifiedAsFullMua** — classifications, not state changes. §3.9.2, lines 1743–1745: "such lists can be treated as a continuation in email transit"; lines 1747–1749: such lists "need to be viewed as full MUAs". "Can be treated as" and "need to be viewed as" are not things that happened.
- **EqualPreferenceExchangersRandomized survives the derived-value test**, and the reason is worth recording: a random draw is precisely the one ordering that *cannot* be recomputed from the inputs, so it is a fact rather than a derivation. §5.1, lines 3877–3881 makes the draw mandatory. Its sibling EqualPreferenceRandomizationSkipped is the same fact with the opposite value.
- **HostileContentDetected, MessageJudgedSeriouslyFraudulent, ReturnAddressDeterminedInvalid** — legitimate events with **no RFC-supplied fields at all**, because §6.2, lines 4054–4056 puts the mechanism outside scope: hostile content is "a decision that is outside the scope of an SMTP server as defined in this document". These are the roster's clearest instances of the kit's fieldless-event tension — an event with no meaningful payload beyond its identity is fine, and an event without fields is an opaque label. Both readings are true here at once. Recorded, not resolved.

---

## Roots the 32-event cap starved

Every sweep returned exactly 32. These roots came back thin or empty, which is where the truncation shows.

- **Size limits.** §4.5.3.1.1 through §4.5.3.1.7, lines 3484–3534, name **seven distinct ceilings**: local-part 64 octets (line 3486), domain 255 (line 3491), path 256 (lines 3495–3496), command line 512 (lines 3500–3501), reply line 512 (lines 3506–3507), text line 1000 (lines 3512–3513), message content at least 64K (lines 3518–3520). The roster has one generic SizeLimitConfigured plus three overflow events. Seven configuration events and seven overflow events are missing.
- **Reply codes.** §4.2.3, lines 2863–2930 lists twenty-four codes in numeric order. `211`, `214`, `450`, `451`, `452`, `455`, `504`, `553` have no event anywhere in the roster.
- **Address literals.** §4.1.3, lines 2380–2442 defines the IPv4 and IPv6 literal forms and the `IPv6:` tag. Two events touch literals in the whole roster.
- **Domain names.** §2.3.5, lines 691–738 — FQDN requirement, label character restriction, the two exceptions (the EHLO argument, and bare `postmaster` at lines 735–737). Three events.
- **Postmaster.** §4.5.1, lines 3394–3402, one of the RFC's few unconditional `MUST`s about a specific mailbox. Five events.
- **HELO versus EHLO.** §4.1.1.1, lines 1783–1878 — the full divergence in what each command binds. The roster treats them almost interchangeably.
- **Transport Service Environment.** Two events for a concept §2.3.10 uses to define all four system roles.

---

## Citation corrections

**One inherited citation is wrong at the source — in the RFC itself.** §4.5.5, line 3790 says non-delivery notifications are "as discussed in Section 3.7". §3.7 begins at line 1531 and is *Mail Gatewaying*; it says nothing about notifications. The undeliverable-notification rule and the null reverse-path requirement are in §3.6.3, lines 1494–1519, and in §6.1, lines 3968–3972. A modeler following §4.5.5's pointer lands in the wrong section. (§4.4, line 3293's pointer to "Section 3.6" for the null return path is correct — the material is at §3.6.3, lines 1507–1510.)

**TURN is not in RFC 5321's body.** ClientServerRolesSwitched is a legitimate candidate, but its only support is Appendix F.1, lines 5164–5171, which describes `TURN` as an RFC 821 command whose "use is deprecated". Any citation placing it in §4.1.1 or in the command set is wrong.

**MailboxExistenceDisclosedViaVrfy's `250` form is constrained.** §3.5.1, lines 1223–1230: a normal response "MAY include the full name of the user and MUST include the mailbox of the user", in one of two named forms — User Name \<Smith@bar.com> or Smith@bar.com. The roster's meaning text omits the MUST.
