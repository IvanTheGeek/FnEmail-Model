# Step 1 sub-step 5 — Business Rules and Constraints in RFC 5321, and the violation events they demand

ём# Step 1 sub-step 5 — Document Business Rules and Constraints

Dymitruk's **Brain Storming** step, sub-step 5, run over RFC 5321. The corpus check called the
normative sentences "the richest seam in the source", and that is what this pass mines: the
MUST / MUST NOT / SHOULD / SHOULD NOT / MAY / REQUIRED statements *are* the business rules,
already written down by the protocol's author.

The kit ties each rule straight to an event. For every rule the follow-up trigger is **"When this
rule is violated, what event signals that?"** — the brainstorming skill's business-rules question,
at roughly line 42 of its `SKILL.md` in the private research repo (rule 7: cited, not reproduced).
That trigger is the whole method of this document.

## How to read this

Every section and line number below was found by opening
`/home/ivan/FnEmail-Model/docs/rfc/rfc5321.txt` and locating the heading and the sentence, not
recalled. Line numbers are 1-indexed against that file. Quotes are verbatim, joined across the
RFC's own line wraps and otherwise untouched (rule 2) — the RFC's punctuation, its `MUST BE` in
caps at §4.5.3.1.7, its hyphenated wraps.

- Rules are grouped by what they constrain, not by section order. Prioritized toward statements
  that constrain **state** over statements that constrain **syntax**; the ABNF productions of
  §4.1.2, the per-object octet counts of §§4.5.3.1.1–4.5.3.1.6, verb case-insensitivity (§2.4,
  L862–868) and trailing-whitespace tolerance (§4.1.1, L1758–1759) are deliberately left out.
- **Violation event** names an event from the roster where one exists, and proposes a new one
  where none does. New events are **bold** and are specified in full in the second half.
- Where a rule has two arms — one binding the client, one the server — both violations are given.

Roughly 137 rules are recorded here. That is not all of them; §4.2.2 and §4.2.3's reply-code
catalog alone would add dozens more, and I stopped short of them because they classify replies
rather than constrain state.

---

## 1. Session opening and closing

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| A server that answers `554` at connection opening must still wait for `QUIT` before closing | MUST | §3.1, L976–977 | "A server taking this approach MUST still wait for the client to send a QUIT ... before closing the connection" | **ConnectionClosedBeforeQuitAfterSessionRefusal** |
| It should answer intervening commands with `503` | SHOULD | §3.1, L978–979 | "SHOULD respond to any intervening commands with \"503 bad sequence of commands\"" | **InterveningCommandProcessedAfterSessionRefusal** |
| A server must not intentionally close the connection except after `QUIT`+`221`, after `421`, or after a timeout | MUST NOT | §3.8, L1646–1658 | "An SMTP server MUST NOT intentionally close the connection under normal operational circumstances" | **SessionClosedWithoutPermittedCause** |
| Closing on a command not understood is named a violation | stated violation | §3.8, L1660–1662 | "a server that closes connections in response to commands that are not understood is in violation of this specification" | ConnectionClosedInViolation |
| A forcibly shut-down server should try to send `421` before exiting | SHOULD | §3.8, L1666–1668 | "SHOULD attempt to send a line containing a 421 response code to the SMTP client before exiting" | **ShutdownNoticeOmittedOnForcedExit** |
| A client hit by an unintended close should treat the transaction as if `451` had arrived | SHOULD | §3.8, L1671–1676 | "treat the mail transaction as if a 451 response had been received and act accordingly" | **ConnectionFailureTreatedAsPermanentFailure** |
| `QUIT` obliges `221` and then a close | MUST | §4.1.1.10, L2207–2208 | "the receiver MUST send a \"221 OK\" reply, and then close the transmission channel" | **QuitAcknowledgmentOmitted** |
| The receiver must not close before it has replied to `QUIT` | MUST NOT | §4.1.1.10, L2210–2211 | "The receiver MUST NOT intentionally close the transmission channel until it receives and replies to a QUIT command" | **SessionClosedWithoutPermittedCause** |
| The sender must not close before sending `QUIT`, and should wait for the reply | MUST NOT / SHOULD | §4.1.1.10, L2212–2214 | "The sender MUST NOT intentionally close the transmission channel until it sends a QUIT command" | ClientAbandonedSessionWithoutQuit&#10;<br>**ClientClosedChannelWithoutAwaitingClosingReply** |
| On a premature close the server must cancel the pending transaction and must not undo completed ones | MUST | §4.1.1.10, L2215–2219 | "the server MUST cancel any pending transaction, but not undo any previously completed transaction" | PendingTransactionCanceled&#10;<br>**CompletedTransactionUndoneAfterConnectionLoss** |
| `RSET` must not cause the connection to close | MUST NOT | §4.1.1.5, L2089–2091 | "An SMTP server MUST NOT close the connection as the result of receiving a RSET" | **ConnectionClosedOnReset** |
| `RSET` must discard stored sender, recipients and mail data and clear all buffers | MUST | §4.1.1.5, L2082–2084 | "Any stored sender, recipients, and mail data MUST be discarded, and all buffers and state tables cleared" | **TransactionStatePreservedAcrossReset** |
| The last command in a session must be `QUIT` | MUST | §4.1.4, L2530 | "The last command in a session MUST be the QUIT command." | ClientAbandonedSessionWithoutQuit |
| A server that can handle only one transaction at a time is non-conformant | stated violation | §4.5.4.2, L3776–3781 | "servers that cannot handle more than one SMTP transaction at a time are not in conformance with the intent of this specification" | **SingleTransactionServerDeployed** |
| A server may detect an attack and defend itself, closing the connection | permitted | §7.8, L4347–4353 | "rational operational behavior requires that servers be permitted to detect such attacks and take action to defend themselves" | ConnectionClosedDefensively |

## 2. Greeting, identity and extension advertisement

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| Servers must not answer `HELO` with the extended response | MUST NOT | §3.2, L995–996 | "Servers MUST NOT return the extended EHLO-style response to a HELO command." | **ExtendedGreetingResponseReturnedToHelo** |
| Servers must support `HELO`; clients must greet before a transaction | MUST | §4.1.1.1, L1818–1820&#10;<br>App F.3, L5202–5204 | "servers MUST support the HELO command and reply properly to it" | **HeloCommandLeftUnsupported**&#10;<br>MailCommandRejectedForUninitializedSession |
| A session carrying mail transactions must first be initialized by `EHLO` | MUST | §4.1.4, L2448–2449 | "A session that will contain mail transactions MUST first be initialized by the use of the EHLO command." | MailCommandRejectedForUninitializedSession |
| A mid-session greeting must clear all buffers and reset state as if `RSET` | MUST | §4.1.4, L2455–2456 | "the SMTP server MUST clear all buffers and reset the state exactly as if a RSET command had been issued" | **BuffersRetainedAcrossReissuedGreeting** |
| After rejecting a greeting the server must stay in the state it was in | MUST | §4.1.4, L2471–2472 | "The SMTP server MUST stay in the same state after transmitting these replies that it was in before the EHLO was received." | **ServerStateChangedAfterRejectedGreeting** |
| 🟥 A failed name-to-address verification must not be used to refuse a message | MUST NOT | §4.1.4, L2481–2484 | "if the verification fails, the server MUST NOT refuse to accept a message on that basis" | **MessageRefusedOnGreetingNameMismatch** |
| The greeting domain must be a primary host name, or an address literal if the host has none | MUST | §2.3.5, L722–726&#10;<br>§4.1.4, L2474–2479 | "The domain name given in the EHLO command MUST be either a primary host name ... or, if the host has no name, an address literal" | **NonPrimaryHostNameDeclaredInGreeting** |
| Local aliases must not appear in any SMTP transaction | MUST NOT | §2.3.5, L712 | "Local aliases MUST NOT appear in any SMTP transaction." | LocalAliasAppearedInTransaction |
| The `EHLO` response must carry keywords for every non-required command | MUST | §4.1.1.1, L1874–1877 | "The EHLO response MUST contain keywords (and associated parameters if required) for all commands not listed as \"required\" in Section 4.5.1" | **ImplementedCommandOmittedFromExtensionList** |
| Extended systems must not advertise what they will answer `502` or `500` | MUST NOT | §4.2.4, L2937–2939 | "Extended SMTP systems MUST NOT list capabilities in response to EHLO for which they will return 502 (or 500) replies." | UnimplementedCapabilityAdvertised |
| A conforming server must not offer unregistered non-X keywords | MUST NOT | §2.2.2, L574–579 | "A conforming server MUST NOT offer non-\"X\"-prefixed keyword values that are not described in a registered extension." | UnregisteredKeywordOffered |
| Clients should not append the legacy identifying string after an address literal | SHOULD NOT | §4.1.1.1, L1802–1807 | "In the interest of interoperability, it is probably wise for servers to be prepared for this string to occur, but SMTP clients SHOULD NOT send it." | LegacyClientIdentificationStringSent |

## 3. Transaction sequencing and the envelope

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| `MAIL` must not be sent while a transaction is already open | MUST NOT | §4.1.4, L2507–2512&#10;<br>§3.3, L1019–1020 | "MAIL (or SEND, SOML, or SAML) MUST NOT be sent if a mail transaction is already open" | NestedMailCommandRejected |
| `RCPT` with no prior `MAIL` must draw `503` | MUST | §3.3, L1088–1089 | "If a RCPT command appears without a previous MAIL command, the server MUST return a 503 \"Bad sequence of commands\" response." | **RecipientAcceptedWithoutMailCommand** |
| Transaction commands must be used in order | MUST | §3.3, L1160 | "Mail transaction commands MUST be used in the order discussed above." | CommandRejectedAsOutOfSequence |
| An unacceptable transaction-beginning argument must draw `501`, leaving state unchanged | MUST | §4.1.4, L2514–2516 | "a 501 failure reply MUST be returned and the SMTP server MUST stay in the same state" | TransactionCommandArgumentRejected |
| Unprocessable out-of-order commands must draw `503`, leaving state unchanged | MUST | §4.1.4, L2516–2528 | "a 503 failure reply MUST be returned and the SMTP server MUST stay in the same state" | CommandRejectedAsOutOfSequence |
| An unacceptable reverse-path must draw a reply that says permanent or temporary | MUST | §3.3, L1036–1039 | "the server MUST return a reply indicating whether the failure is permanent ... or temporary" | **ReversePathFailureReportedWithoutPermanenceIndication** |
| Message data must not be sent unless `354` has been received | MUST NOT | §3.3, L1133–1135 | "message data MUST NOT be sent unless a 354 reply is received" | MailDataSentWithoutAuthorization |
| Contemporary systems should not use source routing in the reverse-path | SHOULD NOT | §3.3, L1047–1049&#10;<br>App C, L4790–4793 | "contemporary systems SHOULD NOT use source routing" | SourceRoutedReversePathSent |
| Receiving systems must recognize source-route syntax and should strip it | MUST / SHOULD | §4.1.1.3, L1925–1928&#10;<br>App F.2, L5188–5189 | "Receiving systems MUST recognize source route syntax but SHOULD strip off the source route specification" | **SourceRouteSyntaxUnrecognized**&#10;<br>SourceRoutedAddressAccepted |
| Source-route host names must not be copied into the reverse-path | MUST NOT | §4.1.1.3, L1930–1931 | "relay hosts SHOULD strip or ignore source routes, and names MUST NOT be copied into the reverse-path" | SourceRouteNamesCopiedIntoReversePath |
| Clients must not generate invalid source routes or rely on serial name resolution | MUST NOT | §3.6.1, L1437–1439 | "SMTP clients MUST NOT generate invalid source routes or depend on serial resolution of names" | InvalidSourceRouteGenerated |
| The client must not transmit parameters the server never offered | MUST NOT | §4.1.1.3, L1979–1981 | "The client MUST NOT transmit parameters other than those associated with a service extension offered by the server in its EHLO response." | **UnofferedExtensionParameterTransmitted** |
| Clients must not send parameters on `RSET`, `DATA` or `QUIT` | MUST NOT | §4.1.1, L1777–1781 | "clients MUST NOT send such parameters and servers SHOULD reject commands containing them as having invalid syntax" | **ParameterSuppliedToParameterlessCommand** |
| The local-part must be interpreted only by the host named in the domain part | MUST | §2.3.11, L847–849 | "the local-part MUST be interpreted and assigned semantics only by the host specified in the domain part of the address" | **LocalPartInterpretedByIntermediateHost** |
| Local-part case must be preserved | MUST | §2.4, L869–871 | "SMTP implementations MUST take care to preserve the case of mailbox local-parts" | **MailboxLocalPartCaseAltered** |
| postmaster must be accepted, with a domain and in the bare form | MUST | §4.5.1, L3394–3402&#10;<br>§2.3.5, L735–737 | "the special case of \"RCPT TO:\<Postmaster>\" (with no domain specification), MUST be supported" | **PostmasterRecipientRejected** |
| Restricted-capability clients must not assume any server will relay for them | MUST NOT | §3.3, L1084–1087 | "restricted-capability clients MUST NOT assume that any SMTP server on the Internet can be used as their mail processing (relaying) site" | **ArbitraryServerAssumedAsRelay** |
| Nine commands must be supported by every receiver | MUST | §4.5.1, L3380–3392 | "The following commands MUST be supported to conform to this specification" | RequiredCommandLeftUnimplemented |

## 4. Content, transparency and the end of data

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| Nothing but `<CRLF>` may be recognized or generated as a line terminator | MUST NOT | §2.3.8, L768–770 | "Conforming implementations MUST NOT recognize or generate any other character or character sequence as a line terminator." | BareLineFeedTerminationAccepted&#10;<br>BareLineTerminatorTransmitted |
| Clients must not transmit bare CR or LF | MUST NOT | §2.3.8, L776–779 | "SMTP client implementations MUST NOT transmit these characters except when they are intended as line terminators" | BareLineTerminatorTransmitted |
| Servers must not accept bare-LF line endings, and `<LF>.<LF>` must not end mail data | MUST NOT | §4.1.1.4, L2028–2034 | "SMTP server systems MUST NOT do this, even in the name of improved robustness" | BareLineFeedTerminationAccepted |
| An extra `<CRLF>` must not be added before the terminating period | MUST NOT | §4.1.1.4, L2012–2014 | "An extra \<CRLF> MUST NOT be added, as that would cause an empty line to be added to the message." | ExtraLineEndingAppended |
| An originator whose final body line lacks `<CRLF>` must reject or repair it | MUST | §4.1.1.4, L2023–2026 | "the originating SMTP system MUST either reject the message as invalid or add \<CRLF>" | **UnterminatedFinalLineForwardedByOriginator** |
| Success must draw an OK reply and failure a failure reply; no partial failure exists here | MUST | §4.1.1.4, L2040–2045 | "The SMTP model does not allow for partial failures at this point" | PartialFailureSignaledInDataReply |
| Errors diagnosed after `250` must be reported in a mail message | MUST | §4.1.1.4, L2047–2049 | "Errors that are diagnosed subsequently MUST be reported in a mail message" | **SubsequentErrorWithheldFromMailReport** |
| Servers should not reject on header or body defects, and must not on mismatched Resent- counts | SHOULD NOT / MUST NOT | §3.3, L1154–1158 | "they MUST NOT reject messages in which the numbers of Resent-header fields do not match" | MessageRejectedForHeaderDefects&#10;<br>**ResentHeaderMismatchRejected** |
| Storage transformations must be reversible | MUST | §4.5.2, L3450–3456 | "If such transformations are necessary, they MUST be reversible, especially if they are applied to mail being relayed." | IrreversibleStorageTransformApplied |
| A client that has not negotiated an extension must not set the high-order bit | MUST NOT | §2.4, L903–906 | "MUST NOT transmit messages with information in the high-order bit of octets" | **EightBitContentTransmittedWithoutNegotiation** |
| Envelope commands may not leave US-ASCII absent an extension; receivers should reject | prohibition / SHOULD | §2.4, L915–919 | "a sending SMTP system is not permitted to send envelope commands in any character set other than US-ASCII" | **NonAsciiEnvelopeCommandTransmitted** |
| 8BITMIME must not be requested for high-bit material that is not MIME | MUST NOT | §2.4, L927–929 | "8BITMIME MUST NOT be requested by senders for material with the high bit on that is not in MIME format with an appropriate content-transfer encoding" | **EightBitMimeRequestedForNonMimeContent** |
| Control characters other than SP, HT, CR, LF should be avoided | SHOULD | §4.1.1.4, L1998–2001 | "use of control characters other than SP, HT, CR, and LF may cause problems and SHOULD be avoided when possible" | ProblematicControlCharacterTransmitted |

## 5. Trace information and the return path

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| A server taking a message for delivery or further processing must insert trace at the top | MUST | §4.4, L3158–3161 | "it MUST insert trace (\"time stamp\" or \"Received\") information at the beginning of the message content" | **TraceLineOmittedOnAcceptance** |
| The FROM clause must be supplied, and should carry both the greeting name and the connection address literal | MUST / SHOULD | §4.4, L3165–3168 | "The FROM clause, which MUST be supplied in an SMTP environment, SHOULD contain both (1) the name of the source host as presented in the EHLO command and (2) an address literal containing the IP address of the source" | FromClauseOmittedFromTrace&#10;<br>TraceSourceNameStampedWithoutConnectionAddress |
| A FOR clause must carry exactly one path | MUST | §4.4, L3173–3174 | "If the FOR clause appears, it MUST contain exactly one \<path> entry, even when multiple RCPT commands have been given." | MultiplePathsStampedInForClause |
| Existing `Received:` lines must not be changed, deleted or reordered; new ones must be prepended | MUST NOT / MUST | §4.4, L3178–3182 | "An Internet mail program MUST NOT change or delete a Received: line that was previously added to the message header section." | ReceivedLineChainAltered&#10;<br>TraceLineInsertedOutOfPosition |
| Obsolete date forms, two-digit years above all, must not be stamped | MUST NOT | §4.4, L3314–3315&#10;<br>App F.5, L5224–5226 | "the \"obs-\" forms, especially two-digit years, are prohibited in SMTP and MUST NOT be used" | ObsoleteDateFormUsedInTraceLine |
| Explicit numeric offsets should be used rather than time zone names | SHOULD | §4.4, L3186–3188 | "SMTP servers that create Received header fields SHOULD use explicit offsets in the dates (e.g., -0800), rather than time zone names of any type." | TraceDateStampedWithZoneNameInsteadOfOffset |
| Unregistered link, protocol and clause names should not be used | SHOULD NOT | §4.4, L3346–3348, L3356–3357, L3373–3374 | "SMTP servers SHOULD NOT use unregistered names." | UnregisteredTraceNameUsed |
| A gateway must prepend a `Received:` line and must not alter one already present | MUST / MUST NOT | §3.7.2, L1577–1579 | "a gateway MUST prepend a Received: line, but it MUST NOT alter in any way a Received: line that is already in the header section" | ReceivedLineOmittedAtBoundaryCrossing&#10;<br>ExistingReceivedLineAlteredByGateway |
| Mail must not be rejected on the format of a trace header field | MUST NOT | §3.7.2, L1586–1588 | "receiving systems MUST NOT reject mail based on the format of a trace header field" | MailRejectedOnTraceHeaderFormat |
| Final delivery writes a return-path line; mail systems must support it | MUST | §4.4, L3204–3207 | "This use of return-path is required; mail systems MUST support it." | ReturnPathLineOmittedAtFinalDelivery |
| A relay must not inspect message data, least of all to look for Return-path fields | MUST NOT | §4.4, L3233–3237 | "SMTP servers performing a relay function MUST NOT inspect the message data, and especially not to the extent needed to determine if Return-path header fields are present." | RelayInspectedMessageData |
| Exactly one return path should be present at delivery | SHOULD | §4.4, L3240–3243 | "For this to be unambiguous, exactly one return path SHOULD be present when the message is delivered." | MultipleReturnPathsPresentAtDelivery |
| The reverse-path must be the target of any mail carrying delivery error messages | MUST | §4.4, L3258–3260 | "The reverse-path address (as copied into the Return-path) MUST be used as the target of any mail containing delivery error messages." | **ErrorMessageAddressedAwayFromReturnPath** |
| A gateway out of SMTP should insert a return-path; a gateway into SMTP should delete it and rebuild the envelope | SHOULD | §4.4, L3262–3273 | "a gateway from elsewhere -> SMTP SHOULD delete any return-path header field present in the message" | ReturnPathInsertedByOutboundGateway&#10;<br>ReturnPathDeletedByInboundGateway |
| New trace header fields must be registered under BCP 90 | MUST | §8, L4444–4447 | "those trace fields MUST be added to the IANA registry established by BCP 90 (RFC 3864) [11]" | **UnregisteredTraceHeaderFieldCreated** |

## 6. Relaying and gatewaying

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| A relay must not inspect or act on header section or body, except to add `Received:` and to detect loops | MUST NOT | §3.6.3, L1524–1529 | "a relay SMTP has no need to inspect or act upon the header section or body of the message data and MUST NOT do so except to add its own \"Received:\" header field" | MessageContentInspectedByRelay&#10;<br>MessageDataModifiedByRelay |
| A relay that accepted the task and cannot deliver must construct and send an undeliverable-mail notification | MUST | §3.6.3, L1494–1501 | "it MUST construct an \"undeliverable mail\" notification message and send it to the originator of the undeliverable mail" | UndeliverableMailSilentlyDiscarded |
| Servers must not send notifications about problems transporting notifications | MUST NOT | §3.6.3, L1505–1506 | "SMTP servers MUST NOT send notification messages about problems transporting notification messages" | NotificationSentAboutNotificationFailure |
| A notification's reverse-path must be null | MUST | §3.6.3, L1508–1510 | "When such a message is transmitted, the reverse-path MUST be set to null" | NotificationSentWithNonNullReversePath |
| A relay must not run validation tests on message header fields | MUST NOT | §4.5.3.1.8, L3540–3542 | "relaying SMTP server MUST NOT, and delivery SMTP servers SHOULD NOT, perform validation tests on message header fields" | **HeaderFieldValidationPerformedByRelay** |
| Message repairs must not be applied by an intermediate relay | MUST NOT | §6.4, L4127–4141 | "These changes MUST NOT be applied by an SMTP server that provides an intermediate relay function." | **MessageRepairedByRelay** |
| Gateway-generated addresses must be FQDN and reply-usable | MUST | §3.7.3, L1598–1600&#10;<br>§3.7.4, L1606–1612 | "MUST reference only fully-qualified domain names, and MUST be effective and useful for sending replies" | NonconformingHeaderAddressForwardedIntoInternet |
| A gateway's translation should route foreign errors to the envelope reverse-path, not to From: or Sender: | SHOULD | §3.7.4, L1612–1617 | "SHOULD ensure that error messages from the foreign mail environment are delivered to the reverse-path from the SMTP envelope" | ForeignErrorRoutedToHeaderOriginator |
| A gateway into the Internet should set the envelope return path from the foreign error return address, falling back to the originator | SHOULD | §3.7.5, L1633–1638 | "the gateway SHOULD set the envelope return path in accordance with an error message return address, if supplied by the foreign environment" | ReturnPathDefaultedToOriginatorAddress |
| A policy refusal to relay should answer `550` | SHOULD | §3.6.2, L1462–1464 | "If it declines to relay mail to a particular address for policy reasons, a 550 response SHOULD be returned." | RelayDeclinedWithNonStandardCode |
| Gateways should preserve layering semantics across the boundary | SHOULD | App E, L5130–5132 | "gateways between the Internet and other mail systems SHOULD attempt to preserve any layering semantics across the boundaries" | EnvelopeInformationLostInTranslation |
| A header-fields-only submission protocol must not be used to gateway foreign mail into SMTP | MUST NOT | App B, L4773–4777 | "A submission protocol based on Standard RFC 822 information alone MUST NOT be used to gateway a message from a foreign (non-SMTP) mail system into an SMTP environment." | MessageGatewayedFromHeaderFieldsAlone |
| A server reached by a source route must remove its own domain name from the forward-path before forwarding | MUST | App C, L4823–4826 | "MUST remove its domain name from any forward-paths in which that domain name appears before forwarding the message" | **OwnDomainLeftInForwardPathAfterRelay** |
| The reverse-path should not be updated by conforming servers | SHOULD NOT | App C, L4827–4828 | "The reverse-path SHOULD NOT be updated by servers conforming to this specification." | **ReversePathUpdatedByRelay** |
| Servers must not derive final routing information from message header fields | MUST NOT | App C, L4834–4835 | "SMTP servers MUST NOT derive final message routing information from message header fields." | **RoutingDerivedFromHeaderFields** |
| A relay that does add itself to a reverse source route must use its name in the environment it is relaying into | MUST | App C, L4841–4845 | "it MUST use its name as known in the transport environment to which it is relaying the mail" | **ReverseSourceRouteStampedWithWrongEnvironmentName** |
| A server that honors a source route must send to the first domain and must not guess shortcuts | MUST / MUST NOT | App F.2, L5191–5193 | "a server MUST NOT guess at shortcuts within the source route" | **SourceRouteShortcutTaken** |
| `TURN` should not be used unless the server can authenticate the client | SHOULD NOT | App F.1, L5169–5171 | "Its use is deprecated; SMTP systems SHOULD NOT use it unless the server can authenticate the client." | **RolesSwitchedWithoutClientAuthentication** |

## 7. Address resolution and next-hop selection

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| A DNS lookup must be performed to resolve the delivery domain | MUST | §5.1, L3826–3829 | "a DNS lookup MUST be performed to resolve the domain name" | DeliveryRoutedWithoutDnsResolution |
| Relays must not infer FQDNs from partial names; submission servers should not | MUST NOT / SHOULD NOT | §5.1, L3830–3835 | "intermediate (relay) SMTP servers MUST NOT make them" | FqdnInferredFromPartialName |
| A non-existent domain must be reported as an error; a temporary error must be queued and retried | MUST | §5.1, L3839–3842 | "If a temporary error is returned, the message MUST be queued and retried later" | MessageFailedWithoutQueueingAfterTemporaryError |
| Address RRs must not be used for a name that has usable MX RRs | MUST NOT | §5.1, L3848–3852 | "SMTP systems MUST NOT utilize any address RRs associated with that name unless they are located using the MX RRs" | AddressRecordsUsedDespiteMailExchangersPresent |
| Unusable MX records, or an unusable implicit MX, must be reported as an error | MUST | §5.1, L3845–3846, L3851–3852 | "If MX records are present, but none of them are usable, this situation MUST be reported as an error." | NoUsableMailExchangerFound |
| An MX data field must be a domain name resolving to at least one address record | MUST | §5.1, L3854–3858 | "That domain name, when queried, MUST return at least one address record (e.g., A or AAAA RR)" | MailExchangerTargetFoundUnresolvable |
| The client must be able to try each candidate address in order, and should try at least two | MUST / SHOULD | §5.1, L3871–3878 | "the SMTP client MUST be able to try (and retry) each of the relevant addresses in this list in order, until a delivery attempt succeeds" | DeliveryAbandonedAfterSingleAddress |
| MX preference must be used in sorting; ties must be randomized | MUST | §5.1, L3883–3889 | "MX records contain a preference indication that MUST be used in sorting"&#10;<br>"the sender-SMTP MUST randomize them" | **PreferenceSortSkipped**&#10;<br>EqualPreferenceRandomizationSkipped |
| Multihomed addresses must be tried in the order presented | MUST | §5.1, L3891–3896 | "the SMTP sender MUST try them in the order presented" | CandidateAddressesTriedOutOfOrder |
| A relay must discard itself and every equal or higher-numbered preference; if nothing remains the message must be returned undeliverable | MUST | §5.1, L3928–3934 | "all records at that preference level and higher-numbered ones MUST be discarded from consideration" | **SelfMatchIgnoredInExchangerList**&#10;<br>MessageReturnedUndeliverableAfterSelfMatch |

## 8. Queueing, retry and timeouts

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| A queuing strategy must time every activity per command, and must never answer an error message with an error message | MUST / MUST NOT | §4.5.4, L3681–3683 | "A queuing strategy MUST NOT send error messages in response to error messages under any circumstances." | ErrorMessageSentInResponseToErrorMessage |
| Mail that cannot be sent immediately must be queued and retried | MUST | §4.5.4.1, L3688–3693 | "mail that cannot be transmitted immediately MUST be queued and periodically retried by the sender" | UnsentMessageDiscardedWithoutQueuing |
| Retry to a destination must be delayed after a failed attempt | MUST | §4.5.4.1, L3703–3705 | "The sender MUST delay retrying a particular destination after one attempt has failed." | DestinationRetriedWithoutDelay |
| Retry parameters must be configurable | MUST | §4.5.4.1, L3709–3714 | "The parameters to the retry algorithm MUST be configurable." | **RetryParametersLeftUnconfigurable** |
| `5yz` responses to `MAIL` must not be cached | MUST NOT | §4.5.4.1, L3746–3750 | "5yz responses to the MAIL command MUST NOT be cached" | MailCommandRejectionCached |
| A client must provide a timeout mechanism and must use per-command timeouts | MUST | §4.5.3.2, L3612–3614 | "An SMTP client MUST provide a timeout mechanism. It MUST use per-command timeouts rather than somehow trying to time the entire mail transaction." | **TransactionScopedTimeoutUsed** |
| A receiver must minimize the time taken to answer the final `<CRLF>.<CRLF>` | MUST | §6.1, L4008–4011 | "a receiver-SMTP MUST seek to minimize the time required to respond to the final \<CRLF>.\<CRLF> end of data indicator" | **EndOfDataReplyDelayedPastClientTimeout**&#10;<br>DuplicateMessageReceived |
| One copy should be sent when several recipients share a destination server | SHOULD | §4.5.4.1, L3759–3767 | "then only one copy of the message SHOULD be transmitted" | SeparateCopyTransmittedPerRecipient |
| A fallback-to-unextended timeout should be shorter than the bounce timeout | SHOULD | §2.2.3, L595–604 | "the timeout to fall back to an unextended format (if one is available) SHOULD be less than the normal timeout for bouncing as undeliverable" | MessageBouncedBeforeFallbackAttempted |

## 9. Responsibility handoff and non-delivery notification

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| A success reply at end of data binds the server to deliver or properly report failure | MUST | §2.1, L419–424 | "the protocol requires that a server MUST accept responsibility for either delivering the message or properly reporting the failure to do so" | AcceptedMessageLost |
| An accepted message must not be lost for frivolous reasons | MUST NOT | §6.1, L3962–3968 | "It MUST NOT lose the message for frivolous reasons, such as because the host later crashes or because of a predictable resource shortage." | AcceptedMessageLost |
| A delivery failure after acceptance obliges a notification, null reverse-path, to the envelope return path | MUST | §6.1, L3970–3974 | "the receiver-SMTP MUST formulate and mail a notification message" | **DeliveryFailureNotificationOmitted** |
| No notification may be sent when the return path is null | MUST NOT | §6.1, L3983–3984 | "if this address is null (\"\<>\"), the receiver-SMTP MUST NOT send a notification" | **NotificationSentToNullReturnPath** |
| A source-routed return path must be stripped to its final hop | MUST | §6.1, L3987–3989 | "If the address is an explicit source route, it MUST be stripped down to its final hop." | **SourceRoutedReturnPathUsedUnstripped** |
| Every failed recipient must be covered, in one notification or in one each | MUST | §4.4, L3285–3288 | "A single notification listing all of the failed recipients or separate notification messages MUST be sent for each failed recipient." | **FailedRecipientOmittedFromNotification** |
| All undeliverable-mail notifications must go out via `MAIL` with a null return path | MUST | §4.4, L3290–3294 | "MUST use a null return path as discussed in Section 3.6" | NotificationSentWithNonNullReturnPath |
| Partial success at end of data must still be answered OK, with a notification following | MUST | §4.4, L3275–3283 | "the response to the DATA command MUST be an OK reply. However, the SMTP server MUST compose and send an \"undeliverable mail\" notification message to the originator" | PartialFailureSignaledInDataReply |
| `4yz` or `5yz` after end of data bars any further delivery attempt by that server | MUST NOT | §4.2.5, L2959–2961, L2981–2983 | "it MUST NOT make a subsequent attempt to deliver that message" | MessageDeliveredAfterResponsibilityDeclined |
| Messages no standard requires to carry a null reverse-path should carry a valid non-null one | SHOULD | §4.5.5, L3803–3805 | "SHOULD be sent with a valid, non-null reverse-path" | NullReversePathUsedForNonNotification |
| Automated processors should not reply to null-reverse-path mail, nor add a non-null path on forwarding | SHOULD NOT | §4.5.5, L3815–3820 | "such systems SHOULD NOT reply to messages with a null reverse-path" | AutoReplySentToNullReversePathMessage&#10;<br>NullReversePathReplacedOnForward |
| Servers must contain provisions for detecting and stopping trivial loops | MUST | §6.3, L4069–4072 | "Whatever mechanisms are used, servers MUST contain provisions for detecting and stopping trivial loops." | **TrivialLoopNotStopped** |
| A drop-on-invalid-return-address policy should apply only under near-certainty | SHOULD | §6.2, L4045–4055 | "it SHOULD be applied only when there is near-certainty that the return addresses are, in fact, invalid" | **MailDroppedOnUnconfirmedInvalidReturnAddress** |
| Bounces for hostile content should not be sent unless the site is confident they will be usefully delivered | SHOULD NOT | §6.2, L4057–4063 | "rejection (\"bounce\") messages SHOULD NOT be sent unless the receiving site is confident that those messages will be usefully delivered" | BounceSentDespiteHostileContent |

## 10. Lists, aliases and blind copies

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| An expanded-list delivery must change the envelope return address to the list administrator | MUST | §3.9, L1690–1693 | "the return address in the envelope (\"MAIL FROM:\") MUST be changed to be the address of a person or other entity who administers the list" | ListReturnPathLeftUnchanged |
| The message header section must be left unchanged, the From field included | MUST | §3.9, L1694–1696 | "the message header section (RFC 5322 [4]) MUST be left unchanged; in particular, the \"From\" field of the header section is unaffected" | HeaderSectionModifiedByExpander |
| Heuristics that drop addresses from an expanded list are strongly discouraged | strongly discouraged | §3.9, L1703–1706 | "application of heuristics or other matching rules to eliminate some addresses, such as that of the originator, is strongly discouraged" | RecipientSuppressedByHeuristic |
| Mail systems should not attempt source expansion to eliminate duplicates | SHOULD NOT | §3.5.4, L1410–1417 | "mail systems SHOULD NOT attempt them" | DuplicateEliminationBySourceExpansionAttempted |
| BCC addresses should be copied to `RCPT` and the BCC fields then removed; an empty BCC should be inserted if nothing remains | SHOULD | App B, L4737–4745 | "Any BCC header fields SHOULD then be removed from the header section." | BccHeaderFieldRemoved&#10;<br>EmptyBccHeaderFieldInserted |
| A submission return address should come from the system's identity, and any pre-existing Sender field should be removed | SHOULD | App B, L4747–4753 | "(Any Sender header field that was already there SHOULD be removed.)" | **PreexistingSenderHeaderFieldRetained** |

## 11. Address diagnostics

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| A `250` to `VRFY` or `EXPN` must mean the address was actually verified | MUST NOT | §3.5.3, L1373–1376 | "A server MUST NOT return a 250 code in response to a VRFY or EXPN command unless it has actually verified the address." | VerificationFalselyClaimed |
| A site that disables these commands must answer `252` | MUST | §7.3, L4241–4245 | "the SMTP server MUST return a 252 response, rather than a code that could be confused with successful or unsuccessful verification" | **DisabledVerificationAnsweredWithVerificationCode** |
| A `250` reply must include the mailbox; `2yz` and `551` must give it as `<local-part@domain>` with an FQDN | MUST | §3.5.1, L1223–1226&#10;<br>§3.5.2, L1337–1340 | "the reply MUST include the \<Mailbox> name using a \"\<local-part@domain>\" construction, where \"domain\" is a fully-qualified domain name" | **VerificationReplyOmittedMailbox** |
| Only addresses usable in `RCPT` may be returned; a program target must be given as the mailbox that reaches it; paths must not be returned | MUST / MUST NOT | §3.5.2, L1351–1357 | "Paths (explicit source routes) MUST NOT be returned by VRFY or EXPN." | SourceRouteReturnedInVerificationReply&#10;<br>**ProgramDeliveryAddressReturnedWithoutMailboxName** |
| If `EXPN` is supported it must be listed as a service extension in the `EHLO` response | MUST | §3.5.2, L1364–1366 | "if EXPN is supported, it MUST be listed as a service extension in an EHLO response" | **ExpnSupportedButNotAdvertised** |
| An implementation of `VRFY` or `EXPN` must at least recognize local mailboxes as user names | MUST | §3.5.1, L1303–1305 | "An implementation of the VRFY or EXPN commands MUST include at least recognition of local mailboxes as \"user names\"." | UserNameRecognitionRuleConfigured |
| `VRFY`, `EXPN`, `HELP` and `NOOP` have no effect on the three buffers | stated invariant | §4.1.1.6, L2114–2115&#10;<br>§4.1.1.7, L2143–2144&#10;<br>§4.1.1.8, L2157–2158&#10;<br>§4.1.1.9, L2197–2198 | "This command has no effect on the reverse-path buffer, the forward-path buffer, or the mail data buffer." | TransactionStateDestroyedByDiagnosticCommand |
| Servers should process `NOOP`, `HELP`, `EXPN`, `VRFY`, `RSET` without a prior greeting | SHOULD | §4.1.4, L2490–2494 | "SMTP servers SHOULD process these normally (that is, not return a 503 code) even if no EHLO command has yet been received" | DiagnosticCommandRejectedAsOutOfSequence |
| A `NOOP` parameter string should be ignored | SHOULD | §4.1.1.9, L2197–2199 | "If a parameter string is specified, servers SHOULD ignore it." | NoopParameterActedUpon |

## 12. Replies and implementation limits

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| Every command must generate exactly one reply | MUST | §4.2, L2551 | "Every command MUST generate exactly one reply." | **CommandLeftUnanswered**&#10;<br>**CommandAnsweredMoreThanOnce** |
| A client must act on the reply code, not the text, outside a named set | MUST | §4.2, L2610–2614 | "An SMTP client MUST determine its actions only by the reply code, not by the text (except for the \"change of address\" 251 and 551 and, if necessary, 220, 221, and 421 replies)" | **ClientActedOnReplyText** |
| A sender must handle unknown codes by the first digit alone | MUST | §4.2, L2624–2626 | "a sender-SMTP MUST be prepared to handle codes not specified in this document and MUST do so by interpreting the first digit only" | UnrecognizedReplyCodeInterpretedByFirstDigit |
| Servers must not send first digits outside 2–5, nor codes of other than three digits | MUST NOT | §4.2, L2628–2630&#10;<br>§4.3.2, L3044–3047 | "SMTP servers MUST NOT send reply codes whose first digits are other than 2, 3, 4, or 5" | **OutOfRangeReplyCodeSent** |
| A client receiving an out-of-range code should treat it as fatal and end the transaction | SHOULD | §4.2, L2639–2640 | "Clients that receive such out-of-range codes SHOULD normally treat them as fatal errors and terminate the mail transaction." | MailTransactionTerminatedOnInvalidReply |
| Substituted reply text must preserve the meanings and actions implied by the code and the sequence | MUST | §4.3.1, L3031–3033 | "the meanings and actions implied by the code numbers and by the specific command reply sequence MUST be preserved" | **ReplyCodeSemanticsAltered** |
| The sender must wait for the reply before sending further commands | MUST | §4.3.1, L2995–2997 | "the sender MUST wait for this response before sending further commands" | CommandSentWithoutAwaitingReply |
| "Command not recognized" to a required command, and "command too long" under 512 octets, are named violations | stated violations | §4.3.2, L3060–3065 | "producing a \"command not recognized\" error in response to the required subset of these commands is a violation of this specification" | RequiredCommandRejectedAsUnrecognized&#10;<br>ShortCommandLineRejectedAsTooLong |
| Every implementation must be able to receive objects of at least the stated sizes | MUST | §4.5.3.1, L3462–3464 | "Every implementation MUST be able to receive objects of at least these sizes." | **UndersizedObjectCeilingEnforced** |
| The message content ceiling must be at least 64K octets | MUST | §4.5.3.1.7, L3518–3519 | "The maximum total length of a message content (including any message header section as well as the message body) MUST BE at least 64K octets." | **MessageContentLimitSetBelowMandatoryMinimum** |
| A server that must restrict size should implement the `SIZE` extension | SHOULD | §4.5.3.1.7, L3522–3526 | "SMTP server systems that must impose restrictions SHOULD implement the \"SIZE\" service extension" | SizeRestrictionImposedWithoutSizeExtension |
| At least 100 recipients must be buffered; refusing below that is a violation | MUST | §4.5.3.1.8, L3537–3539 | "The minimum total number of recipients that MUST be buffered is 100 recipients." | RecipientRejectedBelowMandatoryMinimum |
| A server with a recipient limit must behave in an orderly fashion, not silently discard accepted addresses | MUST | §4.5.3.1.8, L3543–3546 | "MUST behave in an orderly fashion, such as rejecting additional addresses over its limit rather than silently discarding addresses previously accepted" | PreviouslyAcceptedRecipientsSilentlyDiscarded |
| An exhausted implementation limit on `RCPT` must answer `452` | MUST | §4.5.3.1.10, L3598–3600 | "it MUST use a response code of 452" | **RecipientOverflowAnsweredWithWrongCode** |

## 13. Disclosure, forwarding and the registries

| Rule | Strength | § / line | Quote | Violation event |
| --- | --- | --- | --- | --- |
| Clients and servers should not copy the full `RCPT` set into the header section | SHOULD NOT | §7.2, L4218–4223 | "SMTP clients and servers SHOULD NOT copy the full set of RCPT command arguments into the header section" | RecipientListCopiedIntoHeaderSection |
| Receiving systems should not deduce envelope-to-header relationships and rewrite the header on them | SHOULD NOT | §7.2, L4228–4235 | "Receiving systems SHOULD NOT attempt to deduce such relationships and use them to alter the header section of the message for delivery." | HeaderSectionAlteredFromDeducedEnvelopeRelationship&#10;<br>**ApparentlyToHeaderFieldAdded** |
| The FOR clause should be supplied with caution or not at all with multiple recipients | advisory | §7.6, L4327–4330 | "the optional FOR clause should be supplied with caution or not at all when multiple recipients are involved" | BlindCopyRecipientDisclosedInForClause |
| Implementations should minimally make type and version information available to other hosts | SHOULD | §7.5, L4303–4306 | "implementations SHOULD minimally provide for making type and version information available in some way to other network hosts" | **ServerTypeAndVersionConcealedEntirely** |
| A server using `251` or `551` must not assume the client updates its address information | MUST NOT | §3.4, L1194–1196, L1204–1206 | "they MUST NOT assume that the client will actually update address information or even return that information to the user" | ForwardingEntryRetiredOnAssumedClientUpdate |
| Servers supporting `251` and `551` should provide a way to disable or restrict them | SHOULD | §3.4, L1208–1211 | "SHOULD provide configuration mechanisms so that sites that conclude that they would undesirably disclose information can disable or restrict their use" | **AddressUpdateDisclosureLeftUnconfigurable** |
| A client should be certain of the server's authenticity before acting on `251` or `551` | advisory | §7.4, L4286–4290 | "it should be certain of the server's authenticity. If it does not, it may be subject to a man in the middle attack." | AddressCorrectionAcceptedFromUnauthenticatedServer |
| Verbs and keywords not beginning with X must be registered | MUST | §2.2.2, L571–584&#10;<br>§4.1.5, L2543–2544 | "verbs not beginning with \"X\" must always be registered" | **UnregisteredVerbUsedWithoutPrivateUsePrefix** |
| Additional address literal types require standardization before use | requirement | §8, L4400–4405&#10;<br>§4.1.3, L2403–2405 | "Additional literal types require standardization before being used" | **UnstandardizedAddressLiteralTagUsed** |
| #-literals are deprecated and must not be used | MUST NOT | App F.4, L5217–5220 | "It is deprecated and MUST NOT be used." | **PoundLiteralAddressUsed** |
| If `SEND`, `SAML` or `SOML` are implemented, the RFC 821 model must be used and the names published in the `EHLO` response | MUST | App F.6, L5238–5241 | "the implementation model specified in RFC 821 MUST be used and the command names MUST be published in the response to the EHLO command" | **ObsoleteSendingCommandImplementedWithoutAdvertisement** |

---

# Violation events the roster is missing

Seventy-five events. Each is the state change that occurs **when the rule above it is broken** —
past tense, naming a completed state change, not a query. Stream root and actor follow the
roster's own bracket form. Fields are essential only: the identity key plus the one or two facts
that make the event meaningful, each carrying a cardinality.

The last column names the project screen that would have rejected the candidate, where one would.
Per the brief, none of those screens was applied — the screen is recorded, not honored.

## 14.1 Session lifecycle

| Event | Meaning | Fields | § / line | Screen named |
| --- | --- | --- | --- | --- |
| ConnectionClosedBeforeQuitAfterSessionRefusal | A server that had refused the mail session with `554` closed the connection without waiting for the `QUIT` it was required to wait for. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`refusal_code_sent` (Single)&#10;<br>`commands_seen_since_refusal` (List) | §3.1, L976–977 | — |
| InterveningCommandProcessedAfterSessionRefusal | A command that arrived after the `554` refusal was processed normally instead of being refused out of sequence, so the refused session did work. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`command_verb` (Single)&#10;<br>`reply_code_sent` (Single) | §3.1, L978–979 | one event per command |
| SessionClosedWithoutPermittedCause | A server intentionally closed the connection with no `QUIT` answered, no `421` issued and no timeout expired — the general form of the prohibition, of which closing on an unknown command is only one instance. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`last_command_received` (Single)&#10;<br>`transaction_in_progress` (Single) | §3.8, L1646–1658&#10;<br>§4.1.1.10, L2210–2211 | — |
| ShutdownNoticeOmittedOnForcedExit | An SMTP server stopped by external means exited without attempting the `421` line, so every open session learns of the shutdown only by failing. [stream: SMTP Service Instance; actor: operator] | `service_instance_id` (Single)&#10;<br>`open_sessions_at_exit` (Single) | §3.8, L1666–1669 | inbound-only scope |
| ConnectionFailureTreatedAsPermanentFailure | A client whose connection was closed, reset or otherwise failed treated the lost transaction as a permanent failure and bounced it, instead of treating it as `451` and requeueing. [stream: Transaction; actor: SMTP client] | `transaction_id` (Single)&#10;<br>`failure_kind` (Single)&#10;<br>`recipients_bounced` (List) | §3.8, L1671–1676 | responsibility-boundary cutoff |
| QuitAcknowledgmentOmitted | A server closed the transmission channel on `QUIT` without ever sending the `221`, so the client cannot distinguish an orderly close from a drop. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`transactions_completed` (Single) | §4.1.1.10, L2207–2208 | — |
| ClientClosedChannelWithoutAwaitingClosingReply | A client sent `QUIT` and closed the channel without waiting for the reply, so it never learned whether the server had finished. [stream: Session; actor: SMTP client] | `session_id` (Single)&#10;<br>`transactions_completed` (Single) | §4.1.1.10, L2212–2214 | inbound-only scope |
| CompletedTransactionUndoneAfterConnectionLoss | After a premature close the server discarded not only the transaction in progress but one that had already completed, losing a message it had accepted responsibility for. [stream: Transaction; actor: SMTP server] | `session_id` (Single)&#10;<br>`transactions_undone` (List) | §4.1.1.10, L2215–2219 | — |
| ConnectionClosedOnReset | A server closed the connection in response to `RSET`, taking an action reserved for `QUIT`. [stream: Session; actor: SMTP server] | `session_id` (Single) | §4.1.1.5, L2089–2091 | — |
| TransactionStatePreservedAcrossReset | A server answered `RSET` with `250` while keeping some of the stored sender, recipients or mail data, so the next transaction began on residue from the last. [stream: Transaction; actor: SMTP server] | `session_id` (Single)&#10;<br>`buffer_retained` (List) | §4.1.1.5, L2082–2084 | condemned status/phase register |
| SingleTransactionServerDeployed | A server was put into service unable to handle more than one SMTP transaction at a time, a configuration the specification names as out of conformance from the moment it runs. [stream: Server Configuration; actor: implementer or operator] | `service_instance_id` (Single)&#10;<br>`concurrent_transaction_ceiling` (Single) | §4.5.4.2, L3776–3781 | four altitude tiers |

## 14.2 Greeting and extension advertisement

| Event | Meaning | Fields | § / line | Screen named |
| --- | --- | --- | --- | --- |
| ExtendedGreetingResponseReturnedToHelo | A server answered `HELO` with the multiline extended response, so the client now holds an extension list it never asked for and may not be able to parse. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`extension_keywords_sent` (List) | §3.2, L995–996 | HELO-only charter |
| HeloCommandLeftUnsupported | A server shipped or was configured without working `HELO` support, so older clients cannot open a session with it at all. [stream: Server Configuration; actor: implementer] | `service_instance_id` (Single)&#10;<br>`reply_code_returned_to_helo` (Single) | §4.1.1.1, L1818–1819&#10;<br>App F.3, L5202–5204 | HELO-only charter |
| BuffersRetainedAcrossReissuedGreeting | A greeting arrived again mid-session and the server accepted it without clearing the buffers, so a transaction the client believed abandoned survived. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`buffer_retained` (List) | §4.1.4, L2453–2456 | one event per command |
| ServerStateChangedAfterRejectedGreeting | A server rejected a greeting and nonetheless moved out of the state it was in before the command arrived, so client and server no longer share a view of the session. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`reply_code_sent` (Single)&#10;<br>`state_before` (Single) | §4.1.4, L2461–2472 | condemned status/phase register |
| 🟥 MessageRefusedOnGreetingNameMismatch | A server refused a message because the greeting's domain argument did not correspond to the client's IP address — the one basis on which the specification forbids refusal, and the rule most widely broken in practice. [stream: Transaction; actor: SMTP server] | `session_id` (Single)&#10;<br>`claimed_domain` (Single)&#10;<br>`connection_address` (Single) | §4.1.4, L2481–2484 | — |
| NonPrimaryHostNameDeclaredInGreeting | A client identified itself with a domain name that is neither a primary host name nor, where it has no name, an address literal. [stream: Session; actor: SMTP client] | `session_id` (Single)&#10;<br>`claimed_domain` (Single) | §2.3.5, L722–726&#10;<br>§4.1.4, L2474–2479 | inbound-only scope |
| ImplementedCommandOmittedFromExtensionList | A server implements a non-required command but left its keyword out of the `EHLO` response, so a conforming client will never use a capability the server has. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`omitted_keyword` (Single) | §4.1.1.1, L1874–1877 | HELO-only charter |

## 14.3 Transaction and envelope

| Event | Meaning | Fields | § / line | Screen named |
| --- | --- | --- | --- | --- |
| RecipientAcceptedWithoutMailCommand | A server accepted a `RCPT` that arrived with no prior `MAIL`, opening a recipient list against a transaction that was never begun. [stream: Recipient; actor: SMTP server] | `session_id` (Single)&#10;<br>`forward_path` (Single) | §3.3, L1088–1089 | — |
| ReversePathFailureReportedWithoutPermanenceIndication | A server refused a reverse-path with a reply that does not tell the client whether trying the same address again will work, leaving the queue decision undecidable. [stream: Transaction; actor: SMTP server] | `transaction_id` (Single)&#10;<br>`reverse_path` (Single)&#10;<br>`reply_code_sent` (Single) | §3.3, L1035–1039 | — |
| SourceRouteSyntaxUnrecognized | A server failed to recognize source-route syntax in a path it was required to recognize, and so treated a valid address as malformed. [stream: Recipient; actor: SMTP server] | `transaction_id` (Single)&#10;<br>`forward_path` (Single)&#10;<br>`reply_code_sent` (Single) | §4.1.1.3, L1925–1926&#10;<br>App F.2, L5188–5189 | — |
| UnofferedExtensionParameterTransmitted | A client sent a `MAIL` or `RCPT` parameter for an extension the server never advertised, asking for behavior the server has not agreed to. [stream: Transaction; actor: SMTP client] | `transaction_id` (Single)&#10;<br>`parameter_name` (Single) | §4.1.1.3, L1977–1981 | HELO-only charter |
| ParameterSuppliedToParameterlessCommand | A client sent an argument on `RSET`, `DATA` or `QUIT`, none of which take one, in the absence of any extension permitting it. [stream: Session; actor: SMTP client] | `session_id` (Single)&#10;<br>`command_verb` (Single)&#10;<br>`argument_supplied` (Single) | §4.1.1, L1777–1781 | inbound-only scope |
| LocalPartInterpretedByIntermediateHost | A host that is not the one named in the domain part assigned meaning to a mailbox local-part — parsed it, rewrote it, or optimized on it. [stream: Recipient; actor: relay or gateway SMTP server] | `forward_path` (Single)&#10;<br>`interpreting_host` (Single) | §2.3.11, L846–849 | consumer test (G1) |
| MailboxLocalPartCaseAltered | A system changed the case of a mailbox local-part, so the address that leaves is not the address that arrived — Smith and smith are different users on some hosts. [stream: Recipient; actor: SMTP client or server] | `forward_path_as_received` (Single)&#10;<br>`forward_path_as_sent` (Single) | §2.4, L864–871 | consumer test (G1) |
| PostmasterRecipientRejected | A server that relays or delivers mail refused a `RCPT` naming postmaster — at one of its own domains, or in the bare domainless form — which it is required to accept. [stream: Recipient; actor: SMTP server] | `forward_path` (Single)&#10;<br>`reply_code_sent` (Single)&#10;<br>`domainless_form` (Single) | §4.5.1, L3394–3402&#10;<br>§2.3.5, L735–737 | — |
| ArbitraryServerAssumedAsRelay | A restricted-capability client directed mail at a server that had made it no relaying arrangement, assuming the relay would be performed. [stream: Next-Hop Destination; actor: SMTP client] | `destination_server` (Single)&#10;<br>`forward_path` (Single) | §3.3, L1084–1087 | inbound-only scope |

## 14.4 Content and end of data

| Event | Meaning | Fields | § / line | Screen named |
| --- | --- | --- | --- | --- |
| UnterminatedFinalLineForwardedByOriginator | An originating system passed on a message whose final body line lacked `<CRLF>` — neither rejecting it nor adding the terminator — so the receiver cannot recognize the end-of-data condition where the sender meant it. [stream: Message; actor: originating SMTP system] | `message_id` (Single)&#10;<br>`final_line_terminator` (Single) | §4.1.1.4, L2023–2026 | inbound-only scope |
| SubsequentErrorWithheldFromMailReport | A server that had answered `250` later diagnosed an error and never reported it in a mail message, so the sender believes a message was delivered that was not. [stream: Message; actor: SMTP server] | `message_id` (Single)&#10;<br>`error_diagnosed` (Single)&#10;<br>`return_path` (Single) | §4.1.1.4, L2046–2049 | responsibility-boundary cutoff |
| ResentHeaderMismatchRejected | A server refused a message because its Resent- header fields did not balance — Resent-to without Resent-from, or unequal counts — one of the two rejections §3.3 forbids outright. [stream: Message; actor: SMTP server] | `message_id` (Single)&#10;<br>`defect_cited` (Single)&#10;<br>`reply_code_sent` (Single) | §3.3, L1154–1158 | — |
| EightBitContentTransmittedWithoutNegotiation | A client sent message octets with the high-order bit set although it had negotiated no extension permitting them, so the content is outside what the channel guarantees. [stream: Message; actor: originating SMTP client] | `message_id` (Single)&#10;<br>`extension_negotiated` (Single) | §2.4, L903–906 | inbound-only scope |
| NonAsciiEnvelopeCommandTransmitted | A client sent an envelope command carrying characters outside US-ASCII with no server-offered extension permitting it. [stream: Transaction; actor: SMTP client] | `transaction_id` (Single)&#10;<br>`command_verb` (Single) | §2.4, L915–919 | inbound-only scope |
| EightBitMimeRequestedForNonMimeContent | A sender requested 8BITMIME for high-bit material that is not in MIME format with an appropriate content-transfer encoding, claiming a guarantee the content cannot honor. [stream: Transaction; actor: SMTP client] | `transaction_id` (Single)&#10;<br>`content_transfer_encoding` (Single) | §2.4, L924–929 | HELO-only charter |

## 14.5 Trace and return path

| Event | Meaning | Fields | § / line | Screen named |
| --- | --- | --- | --- | --- |
| TraceLineOmittedOnAcceptance | A server took a message for delivery or further processing and added no `Received:` line, so the message carries no record of that hop. [stream: Message; actor: receiving SMTP server] | `message_id` (Single)&#10;<br>`receiving_host` (Single) | §4.4, L3158–3161&#10;<br>§4.1.1.4, L2051–2059 | — |
| ErrorMessageAddressedAwayFromReturnPath | A delivery error message was addressed to something other than the reverse-path — most often a From: or Sender: header address — so the report reaches a party that did not send the mail. [stream: Message; actor: notifying SMTP server] | `message_id` (Single)&#10;<br>`return_path` (Single)&#10;<br>`address_used` (Single) | §4.4, L3255–3260 | — |
| UnregisteredTraceHeaderFieldCreated | A trace header field beyond Return-path and Received came into existence without being entered in the BCP 90 registry, so nothing downstream can be relied on to know what it means. [stream: Extension Registry; actor: implementer or specification author] | `field_name` (Single)&#10;<br>`defining_document` (Single) | §8, L4444–4447 | four altitude tiers |
| HeaderFieldValidationPerformedByRelay | A relay ran validation tests on message header fields — counting header recipients, checking Resent- balance — which relays are forbidden to do. [stream: Message; actor: relay SMTP server] | `message_id` (Single)&#10;<br>`field_validated` (Single) | §4.5.3.1.8, L3540–3542 | — |
| MessageRepairedByRelay | An intermediate relay applied one of the origination repairs — adding a message-id, a date, or correcting an address to FQDN form — which only an originating or submission server may do. [stream: Message; actor: relay SMTP server] | `message_id` (Single)&#10;<br>`repair_applied` (Single) | §6.4, L4127–4141 | — |
| OwnDomainLeftInForwardPathAfterRelay | A server reached by a source route forwarded the message without removing its own domain name from the forward-path, so the route still points back at it. [stream: Recipient; actor: relay SMTP server] | `forward_path_as_sent` (Single)&#10;<br>`relay_domain` (Single) | App C, L4823–4826 | — |
| ReversePathUpdatedByRelay | A relay rewrote the reverse-path it was given, changing where errors about this mail will be reported. [stream: Transaction; actor: relay SMTP server] | `transaction_id` (Single)&#10;<br>`reverse_path_as_received` (Single)&#10;<br>`reverse_path_as_sent` (Single) | App C, L4827–4828 | — |
| RoutingDerivedFromHeaderFields | A server determined where to send a message from its To:, Cc: or From: header fields rather than from the envelope, which the specification forbids outright. [stream: Message; actor: SMTP server] | `message_id` (Single)&#10;<br>`header_field_used` (Single)&#10;<br>`destination_chosen` (Single) | App C, L4830–4835 | — |
| ReverseSourceRouteStampedWithWrongEnvironmentName | A relay that added itself to a reverse source route used its name in the environment the mail came from rather than the one it is relaying into, breaking the return route. [stream: Transaction; actor: relay SMTP server] | `transaction_id` (Single)&#10;<br>`name_stamped` (Single)&#10;<br>`onward_environment` (Single) | App C, L4841–4845 | four altitude tiers |
| SourceRouteShortcutTaken | A server that chose to honor a source route sent the message to a host further along it than the first domain shown, guessing at a shortcut. [stream: Recipient; actor: relay SMTP server] | `forward_path` (Single)&#10;<br>`first_domain_in_route` (Single)&#10;<br>`host_actually_contacted` (Single) | App F.2, L5188–5193 | — |
| RolesSwitchedWithoutClientAuthentication | The two ends of a session swapped roles on `TURN` without the server having authenticated the client, which is the arrangement that lets mail be diverted. [stream: Session; actor: SMTP client issuing TURN] | `session_id` (Single)&#10;<br>`client_authenticated` (Single) | App F.1, L5166–5171 | HELO-only charter |

## 14.6 Address resolution and queueing

| Event | Meaning | Fields | § / line | Screen named |
| --- | --- | --- | --- | --- |
| PreferenceSortSkipped | A client used the MX candidates in the order DNS returned them without sorting on preference, so a backup exchanger may be tried before the primary. [stream: Next-Hop Destination; actor: SMTP client] | `destination_domain` (Single)&#10;<br>`exchanger_tried_first` (Single) | §5.1, L3883–3885 | inbound-only scope |
| SelfMatchIgnoredInExchangerList | A relay found one of its own names or addresses in the sorted candidate list and did not discard that preference level and everything above it, so it is positioned to hand the message back to itself. [stream: Next-Hop Destination; actor: relay host] | `destination_domain` (Single)&#10;<br>`matching_name` (Single)&#10;<br>`candidates_retained` (List) | §5.1, L3928–3934 | responsibility-boundary cutoff |
| RetryParametersLeftUnconfigurable | A sending system was built with its retry interval, give-up time or notification budget fixed in the code, so no site can set them. [stream: Server Configuration; actor: implementer] | `service_instance_id` (Single)&#10;<br>`parameter_fixed` (Single) | §4.5.4.1, L3709–3714 | four altitude tiers |
| TransactionScopedTimeoutUsed | A client timed the whole mail transaction rather than each command and each data buffer, so a long but healthy message is abandoned as though the peer had failed. [stream: Session; actor: SMTP client] | `session_id` (Single)&#10;<br>`timeout_scope` (Single) | §4.5.3.2, L3612–3614 | inbound-only scope |
| EndOfDataReplyDelayedPastClientTimeout | A server took longer to answer the final period than the client was willing to wait, which is the state change that produces the duplicate delivery. [stream: Transaction; actor: SMTP server] | `transaction_id` (Single)&#10;<br>`reply_delay` (Single) | §6.1, L4008–4011&#10;<br>§4.5.3.2.6, L3656–3664 | — |

## 14.7 Responsibility and notification

| Event | Meaning | Fields | § / line | Screen named |
| --- | --- | --- | --- | --- |
| DeliveryFailureNotificationOmitted | A server that had accepted responsibility failed to deliver and never formulated the notification, so the message vanished without the sender being told. [stream: Message; actor: receiver-SMTP holding custody] | `message_id` (Single)&#10;<br>`return_path` (Single)&#10;<br>`failed_recipients` (List) | §6.1, L3970–3974 | responsibility-boundary cutoff |
| NotificationSentToNullReturnPath | A server generated a failure notification for a message whose own return path was null, creating exactly the loop the null path exists to prevent. [stream: Notification; actor: receiver-SMTP] | `message_id` (Single)&#10;<br>`notification_recipient` (Single) | §6.1, L3983–3984 | — |
| SourceRoutedReturnPathUsedUnstripped | A notification was addressed to a return path that still carried its explicit source route, instead of the final hop the specification requires. [stream: Notification; actor: receiver-SMTP composing the notification] | `return_path_as_received` (Single)&#10;<br>`address_used` (Single) | §6.1, L3987–3989 | — |
| FailedRecipientOmittedFromNotification | A notification went out covering some but not all of the recipients that failed, so at least one failure was never reported to anyone. [stream: Notification; actor: notifying SMTP server] | `message_id` (Single)&#10;<br>`recipients_reported` (List)&#10;<br>`recipients_failed` (List) | §4.4, L3285–3288 | — |
| TrivialLoopNotStopped | A server passed on a message that was cycling through an obvious short loop, failing the one loop obligation that carries no conditions. [stream: Message; actor: receiving or relay SMTP server] | `message_id` (Single)&#10;<br>`received_line_count` (Single) | §6.3, L4069–4072 | — |
| MailDroppedOnUnconfirmedInvalidReturnAddress | A site discarded a message under its invalid-return-address policy without having reached near-certainty that the address was in fact invalid. [stream: Message; actor: receiving site] | `message_id` (Single)&#10;<br>`return_path` (Single)&#10;<br>`confidence_basis` (Single) | §6.2, L4045–4055 | — |

## 14.8 Diagnostics

| Event | Meaning | Fields | § / line | Screen named |
| --- | --- | --- | --- | --- |
| DisabledVerificationAnsweredWithVerificationCode | A site that had disabled `VRFY` or `EXPN` answered with a code other than `252` — `250`, `550`, `500` or `502` — so the reply reads as a verification result the server never performed. [stream: Address Diagnostic Request; actor: receiving SMTP server] | `request_id` (Single)&#10;<br>`command_verb` (Single)&#10;<br>`reply_code_sent` (Single) | §7.3, L4241–4245 | — |
| VerificationReplyOmittedMailbox | A `250`, `251` or `551` answer to `VRFY` or `EXPN` came back without the mailbox in `<local-part@domain>` form, so the requester learned an answer it cannot act on. [stream: Address Diagnostic Request; actor: receiving SMTP server] | `request_id` (Single)&#10;<br>`reply_code_sent` (Single) | §3.5.1, L1223–1226&#10;<br>§3.5.2, L1337–1340 | — |
| ProgramDeliveryAddressReturnedWithoutMailboxName | A verification reply named an address whose delivery target is a program or other system without giving the mailbox name that reaches it, so the answer is not usable in a `RCPT` command. [stream: Address Diagnostic Request; actor: receiving SMTP server] | `request_id` (Single)&#10;<br>`address_returned` (Single) | §3.5.2, L1351–1356 | consumer test (G1) |
| ExpnSupportedButNotAdvertised | A server that supports `EXPN` left it out of the `EHLO` response, so no conforming client will discover a facility the server has. [stream: Server Configuration; actor: site administrator] | `service_instance_id` (Single)&#10;<br>`advertised_keywords` (List) | §3.5.2, L1364–1366 | HELO-only charter |

## 14.9 Replies and limits

| Event | Meaning | Fields | § / line | Screen named |
| --- | --- | --- | --- | --- |
| CommandLeftUnanswered | A command was received and no reply was ever generated for it, so the client and server no longer agree on the session state. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`command_verb` (Single) | §4.2, L2551 | one event per command |
| CommandAnsweredMoreThanOnce | More than one reply was generated for a single command, so the client's next action depends on which reply it read. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`command_verb` (Single)&#10;<br>`reply_codes_sent` (List) | §4.2, L2551 | one event per command |
| ClientActedOnReplyText | A client decided what to do from the human-readable text of a reply rather than from its code, outside the `251`, `551`, `220`, `221` and `421` cases where the text is meant to be parsed. [stream: Session; actor: SMTP client] | `session_id` (Single)&#10;<br>`reply_code_received` (Single)&#10;<br>`action_taken` (Single) | §4.2, L2610–2614 | inbound-only scope |
| OutOfRangeReplyCodeSent | A server transmitted a reply code that is not three digits, or whose first digit is not 2, 3, 4 or 5, with no extension permitting it. [stream: Session; actor: SMTP server] | `session_id` (Single)&#10;<br>`reply_code_sent` (Single) | §4.2, L2628–2630&#10;<br>§4.3.2, L3044–3047 | — |
| ReplyCodeSemanticsAltered | An operator substituted reply text that changes the meaning or the implied action of the code, so the code no longer means what the sequence requires. [stream: Server Configuration; actor: server operator] | `reply_code` (Single)&#10;<br>`text_configured` (Single) | §4.3.1, L3021–3033 | four altitude tiers |
| UndersizedObjectCeilingEnforced | An implementation refused an object smaller than the minimum every implementation must be able to receive — a local-part under 64 octets, a path under 256, a command line under 512, a text line under 1000. [stream: Session; actor: receiving SMTP server] | `object_kind` (Single)&#10;<br>`octet_length_refused` (Single)&#10;<br>`reply_code_sent` (Single) | §4.5.3.1, L3462–3464 | — |
| MessageContentLimitSetBelowMandatoryMinimum | A message content ceiling was set below 64K octets, which puts the server out of conformance from the moment the setting takes effect. [stream: Server Configuration; actor: server operator] | `service_instance_id` (Single)&#10;<br>`content_ceiling_octets` (Single) | §4.5.3.1.7, L3516–3519 | four altitude tiers |
| RecipientOverflowAnsweredWithWrongCode | A server whose implementation limit on `RCPT` was exhausted answered with something other than `452` — the obsolete `552`, or a `5yz` that is reserved for a site-policy limit. [stream: Transaction; actor: receiving SMTP server] | `transaction_id` (Single)&#10;<br>`reply_code_sent` (Single)&#10;<br>`recipients_buffered` (Single) | §4.5.3.1.10, L3598–3602 | — |

## 14.10 Disclosure and registries

| Event | Meaning | Fields | § / line | Screen named |
| --- | --- | --- | --- | --- |
| ApparentlyToHeaderFieldAdded | A receiving system added an Apparently-to header field built from the envelope recipients, which the specification names as both a violation of the layering principle and a common source of unintended disclosure. [stream: Message; actor: receiving SMTP server] | `message_id` (Single)&#10;<br>`addresses_exposed` (List) | §7.2, L4231–4235 | — |
| ServerTypeAndVersionConcealedEntirely | A site suppressed all type and version information from both greeting and `HELP`, leaving no way for another host to identify the software for debugging. [stream: Server Configuration; actor: site operator] | `service_instance_id` (Single)&#10;<br>`disclosure_channels_suppressed` (List) | §7.5, L4292–4306 | four altitude tiers |
| AddressUpdateDisclosureLeftUnconfigurable | A server that emits `251` and `551` shipped with no way to disable or restrict them, so a site that judges the disclosure unacceptable cannot stop it. [stream: Server Configuration; actor: implementer] | `service_instance_id` (Single)&#10;<br>`reply_codes_emitted` (List) | §3.4, L1208–1211 | four altitude tiers |
| UnregisteredVerbUsedWithoutPrivateUsePrefix | A verb or parameter name that neither begins with X nor corresponds to a registered extension was sent or accepted, so its meaning is undefined between any two systems. [stream: Session; actor: SMTP client or server] | `session_id` (Single)&#10;<br>`verb_used` (Single) | §2.2.2, L581–584&#10;<br>§4.1.5, L2543–2544 | four altitude tiers |
| UnstandardizedAddressLiteralTagUsed | An address literal was written with a tag that is not specified in a Standards-Track RFC and registered in the Address Literal Tags registry. [stream: Session; actor: SMTP client] | `session_id` (Single)&#10;<br>`literal_tag` (Single)&#10;<br>`literal_body` (Single) | §8, L4400–4405&#10;<br>§4.1.3, L2401–2405 | — |
| PoundLiteralAddressUsed | An address was written in the RFC 821 #-literal form — a decimal host number after a pound sign — which is deprecated and forbidden. [stream: Session; actor: SMTP client] | `session_id` (Single)&#10;<br>`address_used` (Single) | App F.4, L5215–5220 | — |
| ObsoleteSendingCommandImplementedWithoutAdvertisement | A server implemented `SEND`, `SAML` or `SOML` without publishing the command names in its `EHLO` response, so a client can only discover them by trying. [stream: Server Configuration; actor: implementer] | `service_instance_id` (Single)&#10;<br>`commands_implemented` (List) | App F.6, L5238–5241 | HELO-only charter |
| PreexistingSenderHeaderFieldRetained | A submission system generated a Sender header field from the system identity while leaving the one that was already there, so the message carries two claims about who sent it. [stream: Message; actor: submission client MTA] | `message_id` (Single)&#10;<br>`sender_fields_present` (List) | App B, L4747–4753 | — |

---

# Three walked instances

Placeholders find nothing (rule 8). Three of the sharpest new events, instantiated.

**MessageRefusedOnGreetingNameMismatch.** A client opens from 192.0.2.10 and greets as foo.com.
The server resolves foo.com to 203.0.113.20, sees no match, and answers the following `MAIL FROM:`
`<Smith@bar.com>` with `550`. `claimed_domain`: foo.com · `connection_address`: 192.0.2.10.
§4.1.4 permits the check and forbids exactly this use of the result — the information "is for
logging and tracing purposes" (L2484–2485). What the specification wanted here was
ClientNameVerificationRecorded and nothing more.

**SourceRoutedReturnPathUsedUnstripped.** Mail arrives with `MAIL FROM:<@foo.com,@bar.com:Jones@foo.com>`
and fails delivery. The notification is addressed to the whole route rather than to
\<Jones@foo.com>. `return_path_as_received`: @foo.com,@bar.com:Jones@foo.com · `address_used`:
the same string. §6.1 works its own example at L3991–3998 and gets the opposite answer.

**RecipientOverflowAnsweredWithWrongCode.** A transaction reaches 100 buffered recipients, the
101st arrives as `RCPT TO:<Jones@foo.com>`, and the server answers `552`. `reply_code_sent`: 552 ·
`recipients_buffered`: 100. §4.5.3.1.10 says the correct code is `452` and that `552` was RFC 821's
error, still to be tolerated on the client side — so the client-side OverflowReplyReclassifiedAsTransient
already in the roster is the *compensating* event, and this is the one that makes it necessary.

---

# Tensions recorded, not resolved

Three genuine contradictions in the source. Per the brief and rule 12, they are marked where they
bite rather than resolved.

- 🟥 **`VRFY` is both mandatory and disableable.** §4.5.1 (L3381–3392) lists `VRFY` among the nine
  commands that MUST be supported. §3.5.2 (L1359–1362) says implementations MAY give local
  installations a way to disable it. §3.5.3 (L1378–1381) says implementations that answer `500` or
  `502` "are not in full compliance with this specification". §7.3 (L4243–4245) then requires a
  disabled command to answer `252`. A conforming server can therefore be simultaneously required to
  support `VRFY`, permitted to disable it, and obliged to lie about which of those is true. Both
  VrfySupportDisabled and RequiredCommandLeftUnimplemented are legitimate events for the same act.
- 🟥 **Deliver-anyway against reject-to-fix.** §6.2 (L4015–4018) holds that messages that can be
  delivered should be delivered "regardless of any syntax or other faults"; §3.3 (L1154–1156)
  forbids rejecting on header defects; §6.4 (L4083–4085) then records the rejection case —
  "rejection of bad messages is the only way to get the offending software repaired" — without
  ruling. MalformedMessageRejected and MalformedMessagePassedOnUnchanged are both in the roster and
  both conforming, depending on which paragraph binds.
- 🟥 **The fieldless-event tension, live here.** The kit says an event with no meaningful payload
  beyond its identity is fine, and separately that an event without fields is an opaque label.
  Several events above sit exactly on it: ConnectionClosedOnReset carries nothing but
  `session_id`; TrivialLoopNotStopped is meaningful only with the Received count that made it
  detectable. Recorded, not resolved.

---

# Defects found in the inherited roster

Checked while matching rules to events. The method forbids duplicates and requires unique
semantics, so these want reconciling before the roster becomes event files.

| Roster names | Problem |
| --- | --- |
| UndeliverableMailNotificationSent (ch 5, 8) · NonDeliveryNotificationSent (ch 12) | Same event, two names |
| NotificationSentWithNonNullReturnPath (ch 5) · NotificationSentWithNonNullReversePath (ch 8) | Same event; the names differ only in Return / Reverse |
| PreviouslyAcceptedRecipientsSilentlyDiscarded (ch 2) · PreviouslyAcceptedRecipientsDiscarded (ch 14) | Same event, two names |
| MailLoopDetected (ch 4) · MessageLoopDetected (ch 8) | Same event, two names |
| InternalHostNameDisclosedInTraceField (ch 4) · InternalHostNamesDisclosedInTraceField (ch 16) | Same event; singular against plural |
| BlindCopyRecipientDisclosedInTraceField (ch 4) · BlindCopyRecipientDisclosedInForClause (ch 16) | Same event, two names |
| DeliveryResponsibilityAccepted (ch 3) · ResponsibilityForMessageAccepted (ch 5) | Same event, two names |
| SmtpServiceStarted (ch 1) · SmtpListenerStarted (ch 15) | Same event, two names |
| VerificationFalselyClaimed (ch 13) · VerificationFakedAfterSyntaxOnlyCheck (ch 16) | Same event, two names |
| VerificationDeniedWithoutChecking (ch 13) · VerificationAlwaysRefusedWith550 (ch 16) | Same event, two names |
| MailAcceptedDespiteMalformedTraceHeader (ch 4, 9, 14) | One event, three stream roots — Message in two chapters, Transaction in one. The stream root has to be settled |
| TraceClauseRegistered · ProtocolNameRegistered · TraceHeaderFieldRegistered (ch 4, 15, 16) | Each appears in three chapters with differing actors — IANA in one, IANA registrant in another |
| ConnectionClosedInViolation (ch 1, 14) | Scoped in the roster to closing on an unrecognized command. §3.8's MUST NOT is general, and the general violation had no event — hence SessionClosedWithoutPermittedCause above |

No bad RFC citations were found in the roster entries I matched against; the section attributions
I checked (§6.1 for AcceptedMessageLost, §3.9.2 for MessageModifiedByList, §7.2 for the FOR-clause
deprecation at §4.4 L3175–3176) all hold.

---

# What this pass did not reach

- **§4.2.2 and §4.2.3, the reply-code catalog** (L2778–2930). Every code carries a meaning that is
  a rule; they classify replies rather than constrain state, so they were deprioritized. A pass
  over them would likely yield another twenty or thirty rules and a handful of events.
- **The size figures per object** (§§4.5.3.1.1–4.5.3.1.6, L3484–3514). Recorded here only through
  UndersizedObjectCeilingEnforced, which covers all six at once. Splitting it per object is a
  legitimate alternative and would give six events instead of one.
- **The timeout figures** (§§4.5.3.2.1–4.5.3.2.7, L3623–3669). The roster's timeout events cover
  the expiries; the specific minima are configuration facts, reached through
  TimeoutValuesReconfigured.
- **Appendix D's four scenarios** (L4852–5127). They are worked examples rather than rules, and
  they are exactly the material a path walk would use.
- The eleven sweeps were each truncated at 32 by the schema cap, and this sub-step does not repair
  that directly — it recovers what the cap cut off only along the rules axis. The three axes still
  likely under-covered are the reply-code catalog, the scenarios, and §2.2's extension model.
