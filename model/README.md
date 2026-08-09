# The model — Step 1, Brain Storming

**Status: working. Nothing here is ruled.** This directory is the output of *one* step of Adam
Dymitruk's Event Modeling process, run on 2026-08-09 over the whole of RFC 5321. Step 1 is the
widest point of the funnel and deliberately generous — the gentle filtering, the plot, the
storyboard, the commands, the read models and the scenarios are all later steps that have not been
run. Read every event here as *a candidate somebody put on the wall*, not as a decision.

## What Step 1 is

> "We have someone explain the goals of the project and other information. The participants then
> envision what system would look and behave like. They put down all the events that they can
> conceive of having happened. Here we gently introduce the concept that only state-changing events
> are to be specified. Often, people will name "guest viewed calendar for room availability". We put
> those aside for now - they are not events."

The kit states the same step as "Extract every state-changing event the system could have", and
requires three outputs: the event list, the Role Catalog, and the timelines with every event placed
in one. All three are here.

**The screens this project has built for itself were switched off for this pass** — the
capture/decision lens, the condemned status register, the consumer test, the four altitude tiers,
the HELO-only inbound charter, the one-event-per-command target, and the twelve-slice model. Where a
candidate would have been rejected only by one of those, it was kept and the screen named in its
*Notes*. That mapping is deliberate: it shows what the manufactured rules were suppressing.

## What is here

| File | What it holds |
|:--|:--|
| [`events/`](events/) | 492 events, one `<UUIDv7>.md` file each |
| [`ROLE-CATALOG-ACTOR-GENERIC.md`](ROLE-CATALOG-ACTOR-GENERIC.md) | **The Role Catalog that governs.** Automated actors first-class, not residual |
| [`ROLE-CATALOG.md`](ROLE-CATALOG.md) | The first catalog and the workflow discovery. Kept for its trail; its human-scarcity inference is corrected in the file above (AGENTS.md rule 4) |
| [`STREAM-ROOTS.md`](STREAM-ROOTS.md) | Sub-step 2 — the logical roots, and which of them RFC 5321 gives no identity key |
| [`BUSINESS-RULES.md`](BUSINESS-RULES.md) | Sub-step 5 — the MUST / SHOULD / MAY sentences as rules, each with the event that signals its violation |
| [`ANALYSIS.md`](ANALYSIS.md) | Sub-step 6 — business processes, questions for the domain expert, and the recorded procedural decisions |
| [`SET-ASIDE.md`](SET-ASIDE.md) | The "guest viewed calendar" pile — reads, lookups and displays, with reasons. These are the future read models, not discards |
| [`HYGIENE-NOTES.md`](HYGIENE-NOTES.md) | What the field-hygiene pass struck from each workflow, and why |

## How to read an event file

Each file carries four **groupings**, presented as co-equal axes rather than a hierarchy, with the
step that fixes each — so an em dash reads as a schedule, not a defect. *Swimlane* is Dymitruk's
word in both of its uses, and they are different lanes on different rows: actor swimlanes subdivide
the actor section, one per actor; ownership swimlanes subdivide the events section, per team or
system. An event's own row is the ownership swimlane; the actor is recorded as provenance, a
cross-reference into the other section.

**Element type carries no row** — every element here is an event, stated once rather than 492 times.
**Stream-root identity keys** live in [`STREAM-ROOTS.md`](STREAM-ROOTS.md), once.

Every field carries an **Example source**: an RFC line number, `walk scene` for the values the
project's walked path uses, or `illustrative — no RFC source` stated plainly. That column exists
because the first sweep fabricated values at scale, and a value must never again be able to pose as
sourced.

## Known limits, recorded rather than smoothed

- **The first sweep was truncated by a schema cap** — all eleven spans returned exactly 32 events
  because that was the ceiling. A per-workflow re-sweep recovered what was lost, roughly two-thirds
  of most workflows, and almost all of it the *violated* arm of MUST / SHOULD / MAY sentences. Four
  workflows then hit the raised ceiling of 40, so **truncation has not been fully chased out**.
- **The Interview Phase was skipped.** Its conditional entry permits skipping when written
  requirements, documented rules and named domain experts are all present, which the RFC arguably
  supplies as all three. What was lost: its "Known Complexity Areas" question would have aimed the
  sweeps at the 250-after-DATA handoff, partial-recipient failure, and null-reverse-path loop
  suppression before they ran.
- **The first sweep fabricated identity keys and example values at scale** — `session_id` on 99
  events, `transaction_id` on 42, and the value f2C8D14 used 261 times, when RFC 5321 contains no
  session, transaction or connection identifier and f2C8D14 appears nowhere in it. A hygiene pass
  struck them. **The absence of an identity key is a finding**, not a gap to fill, and it is exactly
  what sub-step 2 exists to surface; a fabricated field was concealing it.
- **Sixteen workflow assemblers worked independently**, so 72 event names collided. 71 were merged
  and one was split and renamed; every losing record's notes are carried into the survivor, so no
  research is lost. Semantic duplicates under *different* names have not been hunted.
- **Confidence is the brainstormer's**, not a project status grade: `certain` means the RFC states
  the state change, `likely` that it follows, `doubtful` that it may not be an event at all. 67 of
  the pre-merge records were marked doubtful and deliberately kept — Step 1 is not the filtering
  step.

## The workflows

### 1. Session Lifecycle — 40 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AddressLiteralSubstitutedForDomain` | Session | SMTP client | certain | 1 | [019fe7a8](events/019fe7a8-440a-707a-b440-5a451c508da5.md) |
| `BufferCleared` | Session | SMTP client | certain | 3 | [019fe7a8](events/019fe7a8-440a-7082-8f79-07a9bd07adc0.md) |
| `ClientAbandonedSessionWithoutQuit` | Session | SMTP client | likely | 1 | [019fe7a8](events/019fe7a8-440a-7087-a4aa-ef816160cd91.md) |
| `ClientIdentityDeclared` | Session | SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-7078-99c7-a20f37b85fb5.md) |
| `ClientNameVerificationRecorded` | Session | SMTP server | likely | 3 | [019fe7a8](events/019fe7a8-440a-707c-9d23-c9151e8a179a.md) |
| `ClientServerRolesSwitched` | Session | SMTP client issuing TURN | certain | 1 | [019fe7a8](events/019fe7a8-440a-7096-a8bf-40c7928cbdc0.md) |
| `ClosingReplySent` | Session | SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7085-a0d9-538faaa2ce70.md) |
| `CommandTimeoutExpired` | Session | SMTP client | likely | 1 | [019fe7a8](events/019fe7a8-440a-708d-b6a7-fe8eb5538dac.md) |
| `ConcurrentConnectionLimitReached` | Session | SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7071-8d57-2a00d12b7495.md) |
| `ConnectionClosedDefensively` | Session | SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-7093-a4d3-39224a82b6ad.md) |
| `ConnectionClosedInViolation` | Session | remote SMTP server (peer) | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-7094-bf9a-8ee5df69c4b2.md) |
| `ConnectionFailureTreatedAsTransientError` | Transaction | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-7092-89e3-e437bd14aa02.md) |
| `ConnectionLossTreatedAsQuit` | Session | SMTP server | certain | 0 | [019fe7a8](events/019fe7a8-440a-7091-a7ba-8c4fe0e44bba.md) |
| `ConnectionLostWithoutQuit` | Session | network | certain | 2 | [019fe7a8](events/019fe7a8-440a-7090-88a9-1291625b6e76.md) |
| `ConnectionRefusalConfigured` | Server Configuration | administrator | likely | 1 | [019fe7a8](events/019fe7a8-440a-7077-ad7b-dffab4c6c7f3.md) |
| `DataTerminationTimeoutExpired` | Session | SMTP client | certain | 1 | [019fe7a8](events/019fe7a8-440a-708e-bf97-cfca161f50f2.md) |
| `GreetingRejected` | Session | SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-7080-8c1a-79cb90ac74f4.md) |
| `GreetingTimeoutExpired` | Session | SMTP client | certain | 1 | [019fe7a8](events/019fe7a8-440a-7075-a212-d5f52cbc87ea.md) |
| `HeloFallbackPerformed` | Session | SMTP client | likely | 1 | [019fe7a8](events/019fe7a8-440a-7079-9867-b216d823e31d.md) |
| `InterveningCommandRefusedAfterSessionRefusal` | Session | SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-7074-8b0d-96cce400d08b.md) |
| `LegacyClientIdentificationStringSent` | Session | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-707b-b253-81668b643652.md) |
| `MailSessionRefusedAtConnectionOpening` | Session | SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-7073-95e4-cae225a8f8c0.md) |
| `ServerInactivityTimeoutExpired` | Session | SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-708c-9214-8498df9d8bc5.md) |
| `ServiceExtensionNegotiated` | Session | SMTP client and server jointly | likely | 1 | [019fe7a8](events/019fe7a8-440a-707e-bfb1-7acd740dc24a.md) |
| `ServiceExtensionsAdvertised` | Session | SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-707d-84bc-0a4953d55d00.md) |
| `ServiceForciblyShutDown` | SMTP Service Instance | operator | certain | 1 | [019fe7a8](events/019fe7a8-440a-7089-a226-fa66d3f0d73a.md) |
| `ServiceGreetingSent` | Session | SMTP server | certain | 3 | [019fe7a8](events/019fe7a8-440a-7072-a15c-77e08d1c3834.md) |
| `ServiceShutdownDetected` | SMTP Service Instance | SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-7088-bdcd-f8f7d7d2538f.md) |
| `ServiceUnavailableAnnounced` | Session | SMTP server | certain | 3 | [019fe7a8](events/019fe7a8-440a-708a-9dae-4c61c6dc79b4.md) |
| `SessionEndRequested` | Session | SMTP client | certain | 0 | [019fe7a8](events/019fe7a8-440a-7084-bb36-5cfe2035ac92.md) |
| `SessionInitialStateConfirmed` | Session | SMTP server | certain | 0 | [019fe7a8](events/019fe7a8-440a-707f-a9fb-1a37f35baac1.md) |
| `SessionReinitialized` | Session | SMTP client | certain | 1 | [019fe7a8](events/019fe7a8-440a-7081-97c3-40cdb82aa1ba.md) |
| `ShutdownNoticeReceived` | Next-Hop Destination | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-708b-816c-aa9a728b9d39.md) |
| `SmtpServiceStarted` | SMTP Service Instance | operator | likely | 2 | [019fe7a8](events/019fe7a8-440a-706f-8aa6-8230b95f4d61.md) |
| `SoftwareVersionAnnouncementDisabled` | Server Configuration | server operator | likely | 1 | [019fe7a8](events/019fe7a8-440a-7076-b59b-65b3ad1870a6.md) |
| `StateTableReset` | Session | SMTP client | doubtful | 0 | [019fe7a8](events/019fe7a8-440a-7083-9f7f-61c935f3a999.md) |
| `TimeoutPolicyConfigured` | Server Configuration | administrator | likely | 3 | [019fe7a8](events/019fe7a8-440a-708f-b1b7-dd5673c15857.md) |
| `TransmissionChannelClosed` | Session | SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-7086-8067-979409095fe7.md) |
| `TransmissionChannelOpened` | Session | SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-7070-9ddd-19460ceef99d.md) |
| `UnknownCommandRejected` | Session | SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-7095-b690-f9c96374be90.md) |

### 2. Mail Transaction — 36 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `CommandRejectedAsOutOfSequence` | Session | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70da-adcb-7ffddc41cedc.md) |
| `MailCommandRejectedForUninitializedSession` | Transaction | SMTP server (SMTP-receiver) | likely | 0 | [019fe7a8](events/019fe7a8-440a-70d8-9fe7-4720e63db099.md) |
| `MailTransactionAborted` | Transaction | SMTP client | certain | 1 | [019fe7a8](events/019fe7a8-440a-70db-8835-7587430ccd8d.md) |
| `MailTransactionStarted` | Transaction | SMTP server (SMTP-receiver), on the client's MAIL command | certain | 0 | [019fe7a8](events/019fe7a8-440a-70b9-a40b-bdef3fe05e1a.md) |
| `NestedMailCommandRejected` | Transaction | SMTP server (SMTP-receiver) | certain | 0 | [019fe7a8](events/019fe7a8-440a-70d7-acd7-03dc986a0e92.md) |
| `NullReversePathDeclared` | Transaction | SMTP client (typically a notifying server acting as client) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70bb-9697-3ec03bc6fa64.md) |
| `OverflowReplyReclassifiedAsTransient` | Queue Entry | SMTP client | certain | 4 | [019fe7a8](events/019fe7a8-440a-70d6-a29a-54e816b6e1ec.md) |
| `PendingTransactionCanceled` | Transaction | SMTP server (SMTP-receiver) | certain | 0 | [019fe7a8](events/019fe7a8-440a-70dc-93cd-ace748be6796.md) |
| `PostmasterRecipientAccepted` | Recipient | SMTP server (SMTP-receiver) | certain | 2 | [019fe7a8](events/019fe7a8-440a-70c8-bd81-9eca6dc3e0db.md) |
| `PreviouslyAcceptedRecipientsSilentlyDiscarded` | Transaction | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70d5-99a2-062af0e6183f.md) |
| `RecipientAccepted` | Recipient | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70c7-9e31-2b89141a685f.md) |
| `RecipientBufferLimitReached` | Transaction | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70d2-9fc2-f5f3935aa579.md) |
| `RecipientBuffered` | Transaction | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70c6-a57b-2200cee18211.md) |
| `RecipientCommandRejectedOutOfSequence` | Session | SMTP server (SMTP-receiver) | certain | 0 | [019fe7a8](events/019fe7a8-440a-70d9-83b3-f1907eb4ce48.md) |
| `RecipientDeclared` | Recipient | SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-70c5-a9f5-14b42c782728.md) |
| `RecipientLimitEnforcedBySitePolicy` | Transaction | SMTP server (SMTP-receiver) | likely | 2 | [019fe7a8](events/019fe7a8-440a-70d3-a6dd-5ac238595b43.md) |
| `RecipientRejectedAsUndeliverable` | Recipient | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70c9-8f06-1dc7a1dae1cf.md) |
| `RecipientRejectedBelowMandatoryMinimum` | Transaction | SMTP server (SMTP-receiver) | certain | 2 | [019fe7a8](events/019fe7a8-440a-70d4-a643-fb5b6e154115.md) |
| `RecipientRejectedForPolicy` | Recipient | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70cc-b775-9bc8ed7a7132.md) |
| `RecipientRejectedWithForwardingAddress` | Recipient | SMTP server (SMTP-receiver) | certain | 3 | [019fe7a8](events/019fe7a8-440a-70ca-a5e5-c4217d9c5592.md) |
| `RecipientRejectedWithoutAddressInformation` | Recipient | SMTP server (SMTP-receiver) | certain | 2 | [019fe7a8](events/019fe7a8-440a-70cb-b42b-03903aaefb86.md) |
| `RelayingDeclined` | Recipient | SMTP server (SMTP-receiver) | certain | 3 | [019fe7a8](events/019fe7a8-440a-70cd-adae-bfcab7c1b6a6.md) |
| `ReversePathAccepted` | Transaction | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70be-b782-67b6295e4653.md) |
| `ReversePathBuffered` | Transaction | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70bd-9e9a-aa976e386fa4.md) |
| `ReversePathDeclared` | Transaction | SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-70ba-b1b2-15dba40802b8.md) |
| `ReversePathProblemReportedAfterForwardPathsExamined` | Transaction | SMTP server (SMTP-receiver) | likely | 2 | [019fe7a8](events/019fe7a8-440a-70c0-b1f5-6e98460937d1.md) |
| `ReversePathProvisionallyAccepted` | Transaction | SMTP server (SMTP-receiver) | likely | 1 | [019fe7a8](events/019fe7a8-440a-70bf-b673-f88e998502d2.md) |
| `ReversePathRejectedPermanently` | Transaction | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70c1-ac5f-28ded91e8bba.md) |
| `ReversePathRejectedTemporarily` | Transaction | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70c2-b0e0-61ad5abecf49.md) |
| `SourceRouteNamesCopiedIntoReversePath` | Transaction | SMTP server acting as relay | likely | 1 | [019fe7a8](events/019fe7a8-440a-70d1-a10a-2f53a40650fc.md) |
| `SourceRouteStripped` | Recipient | SMTP server (receiving or relay) | certain | 2 | [019fe7a8](events/019fe7a8-440a-70ce-87cf-d48b05da51a4.md) |
| `SourceRoutedAddressRefused` | Recipient | SMTP server (SMTP-receiver) | likely | 1 | [019fe7a8](events/019fe7a8-440a-70cf-aa8b-d0abfcf8f471.md) |
| `SourceRoutedForwardPathGenerated` | Transaction | SMTP client (sending system) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70d0-bea5-57bb35e7396a.md) |
| `SourceRoutedReversePathSent` | Transaction | SMTP client (sending system) | likely | 1 | [019fe7a8](events/019fe7a8-440a-70c4-9e46-fbf237ff3154.md) |
| `TransactionBuffersCleared` | Transaction | SMTP server (SMTP-receiver) | certain | 1 | [019fe7a8](events/019fe7a8-440a-70bc-aad8-aa4e6df71c70.md) |
| `TransactionCommandArgumentRejected` | Transaction | SMTP server (SMTP-receiver) | certain | 2 | [019fe7a8](events/019fe7a8-440a-70c3-9ffa-7df4bc7b8643.md) |

### 3. Message Content Transfer — 34 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `BareLineFeedTerminationAccepted` | Message | SMTP server | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-70a0-a87e-a1d1b71cd584.md) |
| `BareLineTerminatorTransmitted` | Message | SMTP client | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-709f-8bc4-d9f3426b0eae.md) |
| `DataCommandRefusedAfterRecipientOverflow` | Transaction | SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-709a-9fc2-776c7c908d5e.md) |
| `DataCommandRejectedOutOfSequence` | Transaction | SMTP server | certain | 0 | [019fe7a8](events/019fe7a8-440a-7099-9ee0-d13e8ed30d93.md) |
| `DataRequested` | Transaction | SMTP client | certain | 0 | [019fe7a8](events/019fe7a8-440a-7097-9a6f-be41c91e1523.md) |
| `DeliveryResponsibilityAccepted` | Message | SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-70b5-87f8-e1839a1d076d.md) |
| `DotStuffingApplied` | Message | SMTP client | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-70a4-b9ac-ca876bff93d0.md) |
| `DotUnstuffingApplied` | Message | SMTP server | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-70a5-8a3e-527dc72725bb.md) |
| `EightBitOctetsNormalized` | Message | SMTP client | doubtful | 0 | [019fe7a8](events/019fe7a8-440a-70a2-8d4d-72ce2c56f386.md) |
| `EmptyMessageContentTransmitted` | Message | SMTP client | likely | 0 | [019fe7a8](events/019fe7a8-440a-70a9-9e7e-39800a68fbce.md) |
| `EndOfMailDataIndicated` | Transaction | SMTP client | certain | 0 | [019fe7a8](events/019fe7a8-440a-70a6-b246-3d2871865a3c.md) |
| `EndOfMailIndicatorRecognized` | Transaction | SMTP server | likely | 0 | [019fe7a8](events/019fe7a8-440a-70a7-a503-d1de79ff1b01.md) |
| `ExtraLineEndingAppended` | Message | SMTP client | doubtful | 0 | [019fe7a8](events/019fe7a8-440a-70aa-ad98-00920a439ab7.md) |
| `HeaderValueEncodedForAsciiTransport` | Message | Originating SMTP system | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-70a3-bac9-89d3f0cbaff6.md) |
| `HighOrderBitCleared` | Message | SMTP server | certain | 0 | [019fe7a8](events/019fe7a8-440a-70a1-8f33-8c3fc43080b9.md) |
| `IrreversibleStorageTransformApplied` | Message | Message store | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-70b4-8578-6a762097a650.md) |
| `MailDataAppended` | Message | SMTP client | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-709c-8d8d-f6844d80070a.md) |
| `MailDataBuffersCleared` | Transaction | SMTP server | doubtful | 0 | [019fe7a8](events/019fe7a8-440a-70ad-8083-f2efb2fe9d11.md) |
| `MailDataSentWithoutAuthorization` | Transaction | SMTP client | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-709b-a50e-6d72d49ff276.md) |
| `MailDataTransferAuthorized` | Transaction | SMTP server | certain | 0 | [019fe7a8](events/019fe7a8-440a-7098-a377-7f8eb9782ced.md) |
| `MailTransactionConfirmed` | Transaction | SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-70a8-8161-3a69b47d3e14.md) |
| `MessageAcceptedForProcessing` | Transaction | SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-70ae-b884-faf11b4fabfd.md) |
| `MessageDeliveredAfterResponsibilityDeclined` | Message | SMTP server | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-70b6-9b67-0edbb2155bc1.md) |
| `MessageRejectedAsInvalidByOriginator` | Message | Originating SMTP system | certain | 1 | [019fe7a8](events/019fe7a8-440a-70ac-8208-1d2cd2cd2af1.md) |
| `MessageRejectedAtEndOfData` | Transaction | SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-70af-b709-930cb6f8bfd5.md) |
| `MessageRejectedByPolicy` | Transaction | SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-70b1-ae8a-c4ac3332bd65.md) |
| `MessageRejectedForHeaderDefects` | Message | SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-70b2-892f-a47d1a0c209e.md) |
| `MessageRejectedForIncompleteTransaction` | Transaction | SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-70b0-8955-28c56b9e589e.md) |
| `OversizedMessageDataRejected` | Transaction | SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-70b7-9f92-2605552b4ee4.md) |
| `ProblematicControlCharacterTransmitted` | Message | SMTP client | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-709d-bdba-5cf3a7f9c1a1.md) |
| `RecipientFailureReportedAfterContentAccepted` | Transaction | SMTP server | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-70b3-ab99-2a33b2da6d40.md) |
| `SizeRestrictionImposedWithoutSizeExtension` | Server Configuration | Server operator | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-70b8-9dfb-33b10a9aea2f.md) |
| `TerminatingLineEndingAdded` | Message | Originating SMTP system | certain | 2 | [019fe7a8](events/019fe7a8-440a-70ab-85cb-e928e6ea53b5.md) |
| `TextLineLengthExceeded` | Message | SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-709e-86a2-3c31268ea066.md) |

### 4. Trace and Provenance — 38 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `BlindCopyRecipientDisclosedInTraceField` | Message | Receiving SMTP server (as the unwitting cause) | likely | 2 | [019fe7a8](events/019fe7a8-440a-702f-8872-e32683110343.md) |
| `ErrorReportAddressDesignatedForNonSmtpTransport` | Transport Environment | Administrator of a non-SMTP system | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-7048-ba4e-6318d98295eb.md) |
| `ExistingTraceLinePreserved` | Message | Relay or gateway | doubtful | 0 | [019fe7a8](events/019fe7a8-440a-7032-9111-4eb20f56740d.md) |
| `FromClauseComposed` | Message | Receiving SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7029-ac45-5e3abfa1316f.md) |
| `FromClauseOmittedFromTrace` | Message | Receiving SMTP server (non-conforming) | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-702b-b91d-d2dc82704be4.md) |
| `InternalHostNameDisclosedInTraceField` | Message | Originating or internal SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7030-ad69-fff06e503e56.md) |
| `LoopRejectionThresholdConfigured` | Server Configuration | Administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-704a-b90f-5e74b3a6e95f.md) |
| `LoopedMessagePassedOnUndetected` | Message | Receiving or relay SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-704d-881d-5bd5a76dc639.md) |
| `LoopingMessageRejected` | Message | Receiving or relay SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-704b-b543-f57ec9cd7a0b.md) |
| `MailAcceptedDespiteMalformedTraceHeader` | Message | Receiving SMTP server | likely | 4 | [019fe7a8](events/019fe7a8-440a-703d-afcc-55491d83af6d.md) |
| `MailLoopDetected` | Message | Receiving or relay SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-7049-bcfd-026b7b00db49.md) |
| `MailRejectedOnTraceHeaderFormat` | Message | Receiving SMTP server (non-conforming) | likely | 3 | [019fe7a8](events/019fe7a8-440a-703e-b796-62090f7639c7.md) |
| `MultiplePathsStampedInForClause` | Message | Receiving SMTP server (non-conforming) | likely | 2 | [019fe7a8](events/019fe7a8-440a-702e-9cc0-6a8d142dc4b9.md) |
| `MultipleReturnPathsPresentAtDelivery` | Message | Delivery SMTP server (as the party that discovers it) | likely | 2 | [019fe7a8](events/019fe7a8-440a-7044-8303-3470cbd08799.md) |
| `ObsoleteDateFormUsedInTraceLine` | Message | Receiving SMTP server (non-conforming) | likely | 2 | [019fe7a8](events/019fe7a8-440a-7036-9593-fdd8c5f7b044.md) |
| `ProtocolNameRegistered` | Extension Registry | IANA registrant | likely | 3 | [019fe7a8](events/019fe7a8-440a-7039-a08f-209a9d1c996a.md) |
| `ReceivedLineChainAltered` | Message | Relay or gateway (non-conforming) | likely | 2 | [019fe7a8](events/019fe7a8-440a-7031-a302-319e9d7ffa4f.md) |
| `ReceivedLinePrependedByGateway` | Message | Gateway SMTP | certain | 5 | [019fe7a8](events/019fe7a8-440a-703b-a1bd-3e343f5ee34e.md) |
| `ReturnPathDeletedByInboundGateway` | Message | Gateway (non-SMTP to SMTP) | likely | 3 | [019fe7a8](events/019fe7a8-440a-7046-8541-1542dea18cc8.md) |
| `ReturnPathHeaderStrippedBeforeDelivery` | Message | Delivery SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7042-9d9a-4436d861a944.md) |
| `ReturnPathInsertedByOutboundGateway` | Message | Gateway (SMTP to non-SMTP) | likely | 4 | [019fe7a8](events/019fe7a8-440a-7045-876d-8b1b8650be5c.md) |
| `ReturnPathLineInserted` | Message | Delivery SMTP server | certain | 3 | [019fe7a8](events/019fe7a8-440a-703f-89af-864a8c53b37f.md) |
| `ReturnPathRemovedInTransit` | Message | Forwarding, gateway or relay system | likely | 2 | [019fe7a8](events/019fe7a8-440a-7040-9168-58ecb853a292.md) |
| `ReturnPathSubmittedByOriginatingSystem` | Message | Originating SMTP system (non-conforming) | likely | 2 | [019fe7a8](events/019fe7a8-440a-7043-a476-e757f83d148c.md) |
| `ReversePathConstructedFromForeignEnvelope` | Transaction | Gateway (non-SMTP to SMTP) | likely | 3 | [019fe7a8](events/019fe7a8-440a-7047-8490-d2adb6c13ca7.md) |
| `ReversePathRebuiltForOnwardHop` | Transaction | Forwarding or gateway system | likely | 2 | [019fe7a8](events/019fe7a8-440a-7041-b0d6-cc615ad130c7.md) |
| `SlowRelayDetected` | Relay Host | Operator or postmaster reading trace data | doubtful | 3 | [019fe7a8](events/019fe7a8-440a-7037-8c65-79781fe84fdd.md) |
| `TraceClauseRegistered` | Extension Registry | IANA registrant | likely | 3 | [019fe7a8](events/019fe7a8-440a-7038-922f-00e15fb53d20.md) |
| `TraceDateStampedWithZoneNameInsteadOfOffset` | Message | Receiving SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7035-bcf5-5631e32d86ca.md) |
| `TraceForClauseOmitted` | Message | Receiving SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-702d-9747-38c297a6c3f5.md) |
| `TraceLineInsertedOutOfPosition` | Message | Receiving SMTP server (non-conforming) | likely | 2 | [019fe7a8](events/019fe7a8-440a-7033-b371-4ffa1c7fbdc8.md) |
| `TraceLinePrepended` | Message | Receiving SMTP server (relay or delivery) | certain | 7 | [019fe7a8](events/019fe7a8-440a-7028-a06d-113febb2d53a.md) |
| `TraceRecipientNarrowed` | Message | Receiving SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-702c-a68c-050d0d677880.md) |
| `TraceSourceNameStampedWithoutConnectionAddress` | Message | Receiving SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-702a-86df-fa2aeedcfd3f.md) |
| `TraceTimeFormatConfigured` | Server Configuration | Administrator | doubtful | 4 | [019fe7a8](events/019fe7a8-440a-7034-9ab2-8f2c95e76c51.md) |
| `TrivialLoopStopped` | Message | Receiving or relay SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-704c-9022-4fd652b29cfc.md) |
| `UnregisteredTraceNameUsed` | Message | Receiving SMTP server | likely | 3 | [019fe7a8](events/019fe7a8-440a-703a-a0fd-98b841ed7fab.md) |
| `ViaClauseRecorded` | Message | Gateway SMTP | likely | 4 | [019fe7a8](events/019fe7a8-440a-703c-9430-ccb24aa843f6.md) |

### 5. Responsibility Handoff and Final Delivery — 32 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AcceptedMessageCommittedToStableStorage` | Message | Receiving SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-7055-ba56-6d409016ef9b.md) |
| `AcceptedMessageLost` | Message | Receiving host | doubtful | 3 | [019fe7a8](events/019fe7a8-440a-7056-b089-ac5d1aff8449.md) |
| `DeliveryFailedAfterAcceptance` | Recipient | Receiving SMTP server or its local delivery agent | certain | 3 | [019fe7a8](events/019fe7a8-440a-7065-8d94-cca30034c5e2.md) |
| `DeliveryResponsibilityDeclinedPermanently` | Message | SMTP server (receiver) | certain | 2 | [019fe7a8](events/019fe7a8-440a-7051-a8b5-530812aa0010.md) |
| `DeliveryResponsibilityDeclinedTemporarily` | Message | SMTP server (receiver) | certain | 2 | [019fe7a8](events/019fe7a8-440a-7050-940b-35995f09a94c.md) |
| `DuplicateMessageCopiesDelivered` | Mailbox | Delivery agent or message store | likely | 1 | [019fe7a8](events/019fe7a8-440a-7058-9957-3c7ac738259c.md) |
| `DuplicateMessageReceived` | Message | Receiving SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-7057-afaa-a3ed116d145b.md) |
| `FinalDeliveryMade` | Message | Delivery SMTP server | certain | 3 | [019fe7a8](events/019fe7a8-440a-7059-8bd2-a299be86ed4d.md) |
| `MailCommandRebuiltAfterReturnPathRemoval` | Transaction | Forwarding or gateway system | likely | 2 | [019fe7a8](events/019fe7a8-440a-7062-abe7-a6249b10d44f.md) |
| `MailDataTransformedForLocalStorage` | Mailbox | Message store | likely | 1 | [019fe7a8](events/019fe7a8-440a-705d-af95-8738d73820b3.md) |
| `MessageContentStored` | Message | SMTP server (receiver) | doubtful | 0 | [019fe7a8](events/019fe7a8-440a-704f-8912-d0a44c1e85f2.md) |
| `MessageDeliveredWithAmbiguousReturnPath` | Message | Delivery SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7060-83ef-bff7911d7f31.md) |
| `MessageDepositedInMessageStore` | Mailbox | Delivery agent or message store | certain | 2 | [019fe7a8](events/019fe7a8-440a-705a-bd88-b9595be3e3e0.md) |
| `MessageForwardedToAnotherMailSystem` | Message | Gateway or forwarding system | likely | 3 | [019fe7a8](events/019fe7a8-440a-705c-8699-502b59c1e585.md) |
| `MessageHandedFromUserAgentToMta` | Message | Mail user agent | likely | 3 | [019fe7a8](events/019fe7a8-440a-706e-b1d3-bbd6ee38f42f.md) |
| `MessageHandedOffToMailUserAgent` | Message | Delivery SMTP system | certain | 2 | [019fe7a8](events/019fe7a8-440a-705b-8171-ee1883dcc7e6.md) |
| `MessageIntroducedIntoTransportEnvironment` | Message | Originating SMTP system | likely | 2 | [019fe7a8](events/019fe7a8-440a-706d-9d53-850024a6822e.md) |
| `MessageOriginatedWithReturnPathHeaderPresent` | Message | Originating SMTP system | likely | 2 | [019fe7a8](events/019fe7a8-440a-705f-805a-dded60e1dd76.md) |
| `NotificationSentWithNonNullReturnPath` | Message | Notifying SMTP server | likely | 3 | [019fe7a8](events/019fe7a8-440a-706b-a272-a85fc0951da7.md) |
| `NotificationSuppressedForNullReturnPath` | Message | Receiving SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-7068-bc99-d15bd12ec8e4.md) |
| `NullAddressEventLoggedLocally` | Server Configuration | Receiving site operator, through the receiver-SMTP | likely | 2 | [019fe7a8](events/019fe7a8-440a-7069-8257-5d6b856323ed.md) |
| `PartialDeliveryFailureDetected` | Transaction | Delivery SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-7064-a643-61fd740473d3.md) |
| `PartialFailureSignaledInDataReply` | Transaction | SMTP server (receiver) | likely | 2 | [019fe7a8](events/019fe7a8-440a-7052-8f70-bf59ea97780d.md) |
| `QueuedRecipientsDischargedOnAcceptance` | Queue Entry | Sending SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-7054-8973-c34574fd008b.md) |
| `RecipientValidationDeferred` | Recipient | Receiving SMTP server | likely | 3 | [019fe7a8](events/019fe7a8-440a-706c-8f24-7cc3acc474b5.md) |
| `RelayInspectedMessageData` | Message | Relay SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7063-9e31-3624cd2316a2.md) |
| `ResponsibilityForMessageAccepted` | Message | SMTP server (receiver) | certain | 4 | [019fe7a8](events/019fe7a8-440a-704e-a02c-28178fe678c9.md) |
| `ReturnPathLineOmittedAtFinalDelivery` | Message | Delivery SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-705e-96ad-8de2d438ae21.md) |
| `ReturnPathRemoved` | Message | Forwarding, gateway or relay system | likely | 2 | [019fe7a8](events/019fe7a8-440a-7061-994a-e36d33a5b5d0.md) |
| `SourceRouteStrippedFromReturnPath` | Message | Receiving SMTP server | certain | 3 | [019fe7a8](events/019fe7a8-440a-706a-bbef-4fbcaabbf153.md) |
| `TransferFailureReported` | Message | SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-7053-9a31-978cd8fb6b5e.md) |
| `UndeliverableMailNotificationSentInResponsibilityHandoffandFinalDelivery` | Message | Notifying SMTP server, now acting as an originating system | certain | 4 | [019fe7a8](events/019fe7a8-440a-7066-90e1-b70b3867c542.md) |

### 6. Address Resolution and Next-Hop Selection — 40 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AddressFamilyCandidateSkipped` | Next-Hop Destination | SMTP client | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-701c-b692-e778614b1be2.md) |
| `AddressRecordPublished` | DNS Zone | Domain administrator / DNS zone operator | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-7025-9fa7-983b8f6a42ac.md) |
| `AddressRecordsUsedDespiteMailExchangersPresent` | Next-Hop Destination | SMTP client | certain | 3 | [019fe7a8](events/019fe7a8-440a-700c-89f4-9287fc56dff7.md) |
| `AliasRecordPublished` | DNS Zone | Domain administrator / DNS zone operator | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-7024-b5bf-10f42a303b5c.md) |
| `AlternateAddressLimitConfigured` | Server Configuration | Installation administrator | certain | 3 | [019fe7a8](events/019fe7a8-440a-7015-aea3-99307d79db7c.md) |
| `AlternateAddressLimitReached` | Queue Entry | SMTP client | likely | 1 | [019fe7a8](events/019fe7a8-440a-7016-bb3e-dfef7fcbd06c.md) |
| `CandidateAddressesExhausted` | Queue Entry | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-7017-9d1f-e25f29a4a184.md) |
| `CandidateAddressesTriedOutOfOrder` | Next-Hop Destination | SMTP client | likely | 3 | [019fe7a8](events/019fe7a8-440a-7019-9af7-b027b4fd0c3b.md) |
| `DeliveryAbandonedAfterSingleAddress` | Queue Entry | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-701a-85fe-f9e4a99d00df.md) |
| `DeliveryAttemptFailedForAddress` | Next-Hop Destination | SMTP client | certain | 3 | [019fe7a8](events/019fe7a8-440a-7018-aa03-7cd71aaf3299.md) |
| `DeliveryDomainLexicallyIdentified` | Next-Hop Destination | SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-7000-afec-da3fc3c4c950.md) |
| `DeliveryRoutedWithoutDnsResolution` | Next-Hop Destination | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-7027-9e92-5b5c442530e1.md) |
| `DomainNameLabelSyntaxViolated` | Transaction | SMTP client | likely | 1 | [019fe7a8](events/019fe7a8-440a-7003-ab3f-2a736780ffa1.md) |
| `EqualPreferenceExchangersRandomized` | Next-Hop Destination | sender-SMTP | likely | 3 | [019fe7a8](events/019fe7a8-440a-700f-bbf4-c88f05e9be5b.md) |
| `EqualPreferenceRandomizationSkipped` | Next-Hop Destination | sender-SMTP | likely | 3 | [019fe7a8](events/019fe7a8-440a-7010-99dd-885dd7cbc90e.md) |
| `ForwardPathRewrittenBeforeRelay` | Recipient | Designated mail exchanger | likely | 2 | [019fe7a8](events/019fe7a8-440a-7021-bc0a-aa04018d6425.md) |
| `FqdnInferredFromPartialName` | Next-Hop Destination | Message submission server, or a relay violating the prohibition | likely | 3 | [019fe7a8](events/019fe7a8-440a-7005-9306-50a78738cb19.md) |
| `ImplicitMailExchangerAssumed` | Next-Hop Destination | SMTP client | certain | 1 | [019fe7a8](events/019fe7a8-440a-700a-9f20-fbd2c8867bb4.md) |
| `LocalAliasAppearedInTransaction` | Transaction | SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-7001-94e5-175611ef82d9.md) |
| `LowerPreferenceExchangersDiscarded` | Next-Hop Destination | Relay host | certain | 2 | [019fe7a8](events/019fe7a8-440a-7013-b3de-c20e59407af2.md) |
| `MailExchangerAddressResolved` | Next-Hop Destination | SMTP client, on an answer from the DNS resolver | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-700d-bf7a-d98c56155bf9.md) |
| `MailExchangerRecordPublished` | DNS Zone | Domain administrator / DNS zone operator | likely | 3 | [019fe7a8](events/019fe7a8-440a-7022-919a-d2dc38766f36.md) |
| `MailExchangerRecordWithdrawn` | DNS Zone | Domain administrator / DNS zone operator | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-7023-93eb-43d727b9e546.md) |
| `MailExchangerTargetFoundUnresolvable` | Next-Hop Destination | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-700e-a702-fced3bea5814.md) |
| `MessageDeliveredFinallyByDesignatedExchanger` | Message | Designated mail exchanger acting as delivery system | certain | 2 | [019fe7a8](events/019fe7a8-440a-701e-907f-db16cbdb8b2e.md) |
| `MessageFailedWithoutQueueingAfterTemporaryError` | Queue Entry | SMTP client / outbound queue manager | likely | 1 | [019fe7a8](events/019fe7a8-440a-7009-81d9-de9000b41123.md) |
| `MessageHandedOffOutsideSmtpEnvironment` | Message | Designated mail exchanger acting as gateway | certain | 2 | [019fe7a8](events/019fe7a8-440a-701f-9a03-add187116e0e.md) |
| `MessageQueuedAfterTemporaryDnsError` | Queue Entry | SMTP client / outbound queue manager | certain | 1 | [019fe7a8](events/019fe7a8-440a-7008-a343-235eecfb3744.md) |
| `MessageRelayedByDesignatedExchanger` | Message | Designated mail exchanger acting as relay | certain | 3 | [019fe7a8](events/019fe7a8-440a-701d-a904-586c7334c750.md) |
| `MessageReturnedUndeliverableAfterSelfMatch` | Message | Relay host | certain | 2 | [019fe7a8](events/019fe7a8-440a-7014-af15-555dd997f82a.md) |
| `NextHopChannelEstablished` | Next-Hop Destination | Sending SMTP system acting as SMTP client | certain | 3 | [019fe7a8](events/019fe7a8-440a-701b-805a-db45bc2da591.md) |
| `NoUsableMailExchangerFound` | Next-Hop Destination | SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-700b-b4e1-7b9fdcc6dfc0.md) |
| `ReversePathRewrittenBeforeRelay` | Transaction | Designated mail exchanger | likely | 2 | [019fe7a8](events/019fe7a8-440a-7020-bdca-af225d835788.md) |
| `RoutingErrorReported` | Next-Hop Destination | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-7026-8ab9-835756f22dd0.md) |
| `SelfIdentifiedInMailExchangerList` | Next-Hop Destination | Relay host | certain | 2 | [019fe7a8](events/019fe7a8-440a-7011-9b65-551bd6ce98c0.md) |
| `ServerIdentityNamesConfigured` | Server Configuration | Installation administrator | likely | 3 | [019fe7a8](events/019fe7a8-440a-7012-b77a-07ff526de97b.md) |
| `TargetDomainFoundNonExistent` | Next-Hop Destination | SMTP client, on an answer from the DNS resolver | certain | 1 | [019fe7a8](events/019fe7a8-440a-7007-bf08-05cb7d921fb8.md) |
| `TargetNameResolvedToCanonicalName` | Next-Hop Destination | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-7006-98b2-beadb4c07990.md) |
| `TopLevelDomainAddressUsed` | Next-Hop Destination | SMTP client | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-7004-93ec-c0c8d9a19f02.md) |
| `UnqualifiedDomainNameRefused` | Session | SMTP server | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-7002-8c9b-3985f49fa4a8.md) |

### 7. Sending, Queueing and Retry — 35 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AlternateMailExchangerTried` | Next-Hop Destination | Intermediate SMTP system | likely | 4 | [019fe7a8](events/019fe7a8-440a-70fa-a6c2-f35a30786f2e.md) |
| `CachedNegativeResponseInvalidated` | Next-Hop Destination | Sending SMTP client | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-70f2-84a9-b1ff8b97dfdf.md) |
| `CommandRetryAttempted` | Session | SMTP client | likely | 1 | [019fe7a8](events/019fe7a8-440a-70ff-a145-6e829f4f73d5.md) |
| `DeliveryAttemptFailed` | Queue Entry | Sending SMTP client | certain | 6 | [019fe7a8](events/019fe7a8-440a-70e1-b6be-693c4c43008d.md) |
| `DeliveryGivenUp` | Queue Entry | Sending SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-70e8-9275-816b5455b99b.md) |
| `DestinationBackoffWidened` | Next-Hop Destination | Sending SMTP client / queue runner | likely | 3 | [019fe7a8](events/019fe7a8-440a-70e4-a123-e6caff722de9.md) |
| `DestinationRetriedWithoutDelay` | Next-Hop Destination | Sending SMTP client | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-70e3-b6fb-fd1011cbdf67.md) |
| `ErrorMessageSentInResponseToErrorMessage` | Outbound Queue | Queue runner daemon | likely | 1 | [019fe7a8](events/019fe7a8-440a-70ec-9a54-31e6a4283f51.md) |
| `ExtensionFallbackTimeoutElapsed` | Queue Entry | Queue timer | likely | 3 | [019fe7a8](events/019fe7a8-440a-70fb-b083-cabfa5aecb80.md) |
| `ExtensionListChangedWithinSession` | Session | SMTP server | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-70f3-94e4-d37616cd6c1b.md) |
| `HostMarkedUnreachable` | Next-Hop Destination | Sending SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-70ed-96a0-b3379b62d956.md) |
| `HostReachabilityRestored` | Next-Hop Destination | Sending SMTP client | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-70ee-bc2d-00bdcb4aa3d4.md) |
| `MailCommandRejectionCached` | Next-Hop Destination | Sending SMTP client | likely | 3 | [019fe7a8](events/019fe7a8-440a-70f1-a2fd-63e2c23187b4.md) |
| `MessageBouncedBeforeFallbackAttempted` | Queue Entry | Queue timer | likely | 4 | [019fe7a8](events/019fe7a8-440a-70fd-b1ad-ed5274fe9366.md) |
| `MessageDowngradedToUnextendedFormat` | Queue Entry | Intermediate SMTP system | likely | 3 | [019fe7a8](events/019fe7a8-440a-70fc-863b-04968f465cd3.md) |
| `MessageQueuedForDelivery` | Queue Entry | Sending SMTP client / message submission server | certain | 4 | [019fe7a8](events/019fe7a8-440a-70dd-b64c-51473a6f7c50.md) |
| `MessageRequeued` | Queue Entry | SMTP client / sending MTA queue manager | certain | 2 | [019fe7a8](events/019fe7a8-440a-70e5-989e-823ebc13b8d9.md) |
| `MessageRequeuedForExtensionSupport` | Queue Entry | Intermediate SMTP system | certain | 3 | [019fe7a8](events/019fe7a8-440a-70f9-b901-9aac73986cd3.md) |
| `NegativeResponseCached` | Next-Hop Destination | Sending SMTP client | likely | 3 | [019fe7a8](events/019fe7a8-440a-70f0-ab13-661eb6290b76.md) |
| `NotificationDeliveryAbandoned` | Queue Entry | Sending SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-70e9-8cb5-186ef31429b3.md) |
| `OutboundConcurrencyLimitReached` | Outbound Queue | Sending SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-70f7-8578-166c05603754.md) |
| `QueueActivityTimedOut` | Queue Entry | Queue runner daemon | likely | 2 | [019fe7a8](events/019fe7a8-440a-70eb-930e-2dafab71d786.md) |
| `QueueEntryFlaggedForImmediateAttention` | Queue Entry | Message composition program | likely | 1 | [019fe7a8](events/019fe7a8-440a-70de-838d-fc50c228c90d.md) |
| `QueueFlushTriggeredByPeerContact` | Next-Hop Destination | SMTP server / queue manager | likely | 2 | [019fe7a8](events/019fe7a8-440a-70f8-83ae-87d60c099ce7.md) |
| `QueueRunStarted` | Outbound Queue | Queue runner daemon | likely | 0 | [019fe7a8](events/019fe7a8-440a-70e0-ab8f-0dc486c8107a.md) |
| `QueuedMessageReturnedToUser` | Queue Entry | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-70e6-8dc6-c563d7ca4ea3.md) |
| `QueuedMessageTransmitted` | Queue Entry | Sending SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-70e7-8bd6-e1890c8c2bef.md) |
| `RecipientListSplitAcrossTransactions` | Queue Entry | Sending SMTP client | likely | 3 | [019fe7a8](events/019fe7a8-440a-70f5-b694-37f27ba2f91d.md) |
| `RecipientsGroupedIntoSingleTransaction` | Queue Entry | Sending SMTP client | likely | 3 | [019fe7a8](events/019fe7a8-440a-70f4-84d3-aa17ae478f0d.md) |
| `RetryParametersConfigured` | Server Configuration | Site operator | certain | 4 | [019fe7a8](events/019fe7a8-440a-70ea-b924-ce644e26e3bd.md) |
| `RetryScheduled` | Queue Entry | Sending SMTP client / queue runner | certain | 3 | [019fe7a8](events/019fe7a8-440a-70e2-a92e-3499d64a4b64.md) |
| `RetryStrategyAdjustedForMultipleAddresses` | Next-Hop Destination | Sending SMTP client | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-70fe-b264-77d65c661aa0.md) |
| `SendingSystemBlockedByRetryCycle` | Outbound Queue | Sending SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-70ef-8309-57cd287baa1f.md) |
| `SeparateCopyTransmittedPerRecipient` | Queue Entry | Sending SMTP client | likely | 3 | [019fe7a8](events/019fe7a8-440a-70f6-afb4-823a62264d41.md) |
| `UnsentMessageDiscardedWithoutQueuing` | Message | Sending SMTP client | likely | 0 | [019fe7a8](events/019fe7a8-440a-70df-9af2-8a24cc601482.md) |

### 8. Relaying — 32 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `DeliveryDeterminedImpossible` | Queue Entry | Relay SMTP server, or the first host to determine it | certain | 3 | [019fe7a8](events/019fe7a8-440a-71a4-b6e6-7eebcad5261f.md) |
| `DeliveryFailedAfterSourceRouteStripped` | Recipient | Relay SMTP server (after stripping) | likely | 1 | [019fe7a8](events/019fe7a8-440a-71b3-928a-1c7a8b55a366.md) |
| `ExplicitSourceRouteGenerated` | Recipient | SMTP client (originating or relaying system) | likely | 1 | [019fe7a8](events/019fe7a8-440a-71af-a16f-280ec706562f.md) |
| `InvalidSourceRouteGenerated` | Recipient | SMTP client (originating or relaying system) | likely | 1 | [019fe7a8](events/019fe7a8-440a-71b0-b20d-a585c4e42027.md) |
| `MailRejectedForPolicyReason` | Session | Receiving SMTP server | certain | 3 | [019fe7a8](events/019fe7a8-440a-71ba-99c0-8357d7685937.md) |
| `MailRelayedOnward` | Relay Task | Relay SMTP server acting as SMTP client | certain | 3 | [019fe7a8](events/019fe7a8-440a-71a3-afb5-f931a0002abd.md) |
| `MessageContentInspectedByRelay` | Message | Relay SMTP server (non-conforming) | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-71ac-8ac4-32a805497d6a.md) |
| `MessageContentPassedThroughUnmodified` | Message | Relay SMTP server | certain | 0 | [019fe7a8](events/019fe7a8-440a-71ab-b61b-b5929e6a6a2e.md) |
| `MessageDataModifiedByRelay` | Message | Relay SMTP server (non-conforming) | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-71ad-af2e-71dea64da2c8.md) |
| `MessageLoopDetected` | Message | Relay SMTP server | likely | 0 | [019fe7a8](events/019fe7a8-440a-71aa-a8d0-1ef7dbce9151.md) |
| `MessageQueuedForSubsequentDelivery` | Queue Entry | Relay or submission SMTP server | likely | 3 | [019fe7a8](events/019fe7a8-440a-71a8-93dd-dddbb15c0434.md) |
| `MessageRepairRefusedByRelay` | Message | Intermediate relay SMTP server | certain | 3 | [019fe7a8](events/019fe7a8-440a-71ae-b7fb-c111e062c319.md) |
| `MessageSubmittedForDistribution` | Message | Mail-sending client (originating system) | likely | 2 | [019fe7a8](events/019fe7a8-440a-71bc-9c4a-80eff02ae816.md) |
| `NextHopChannelRefused` | Next-Hop Destination | Relay SMTP server acting as SMTP client | likely | 1 | [019fe7a8](events/019fe7a8-440a-71a2-9e1e-bafbd7e6a262.md) |
| `NotificationLoopGenerated` | Notification Message | Relay or delivery SMTP server (non-conforming) | likely | 1 | [019fe7a8](events/019fe7a8-440a-71a7-be22-a343158caef5.md) |
| `NotificationSentWithNonNullReversePath` | Notification Message | Notifying SMTP server (non-conforming) | likely | 1 | [019fe7a8](events/019fe7a8-440a-71a6-a763-57f45a4e0df7.md) |
| `ReceivedHeaderFieldAddedByRelay` | Message | Relay SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-71a9-8755-f2267ace285d.md) |
| `RelayAccessRestrictedToKnownSources` | Server Configuration | Site operator or administrator | certain | 4 | [019fe7a8](events/019fe7a8-440a-71b7-ad89-e8cb527b8134.md) |
| `RelayDeclinedWithNonStandardCode` | Relay Task | Relay SMTP server | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-71a0-b362-4a602cd32f2e.md) |
| `RelayTaskAccepted` | Relay Task | Relay SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-719e-894d-0823e4474c3a.md) |
| `RelayTaskDeclined` | Relay Task | Relay SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-719f-a91f-16b49e381a91.md) |
| `RelayUsedToObscureOrigin` | Message | Hostile SMTP client | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-71b9-b70e-02f515f7c8c9.md) |
| `ReturnPathVerificationFailed` | Transaction | Relay or delivery SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-71b6-a010-d912e047f601.md) |
| `ReturnPathVerified` | Transaction | Relay or delivery SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-71b5-bddf-bce35ba43956.md) |
| `ReversePathResolvedAtDelivery` | Transaction | Delivery SMTP system | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-71b4-977f-b592efee25fd.md) |
| `SelectiveAcceptancePolicyAdopted` | Server Configuration | Site operator or administrator | likely | 1 | [019fe7a8](events/019fe7a8-440a-71b8-85c1-4aa53a3841a7.md) |
| `SmtpClientRoleAssumed` | Relay Task | Relay SMTP server | likely | 0 | [019fe7a8](events/019fe7a8-440a-71a1-a4e3-f2ba63ff087c.md) |
| `SourceRouteHonored` | Recipient | Relay SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-71b2-81cc-d092c54b77e7.md) |
| `SourceRoutedAddressAccepted` | Recipient | Receiving SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-71b1-94a9-7878b6edafc2.md) |
| `SubmissionArrangementEstablished` | Server Configuration | Mail system operator or administrator, on behalf of a mail-sending client | likely | 2 | [019fe7a8](events/019fe7a8-440a-71bb-a959-b720bececef6.md) |
| `UndeliverableMailNotificationSentInRelaying` | Notification Message | Relay SMTP server, or the first host to determine non-delivery | certain | 3 | [019fe7a8](events/019fe7a8-440a-7067-9b40-d24b903626fa.md) |
| `UndeliverableMailSilentlyDiscarded` | Queue Entry | Relay SMTP server (non-conforming) | likely | 3 | [019fe7a8](events/019fe7a8-440a-71a5-9823-6f22e2fb49cf.md) |

### 9. Gatewaying — 25 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AddressRewrittenByGateway` | Message | Address-rewriting firewall acting as a gateway | likely | 3 | [019fe7a8](events/019fe7a8-440a-7112-aa7b-3064e9bb2100.md) |
| `EnvelopeFoldedIntoHeaderSection` | Message | Gateway SMTP | likely | 4 | [019fe7a8](events/019fe7a8-440a-7114-ba06-01dda965e70f.md) |
| `EnvelopeInformationLostInTranslation` | Message | Gateway SMTP | likely | 3 | [019fe7a8](events/019fe7a8-440a-7127-bb6e-74fa4f869120.md) |
| `EnvelopeReturnPathSetAtGateway` | Message | Gateway SMTP | certain | 3 | [019fe7a8](events/019fe7a8-440a-7120-b404-8c5f0df21788.md) |
| `ExistingReceivedLineAlteredByGateway` | Message | Gateway SMTP | certain | 1 | [019fe7a8](events/019fe7a8-440a-7116-bbe4-a4fa94b90097.md) |
| `ForeignErrorRoutedToHeaderOriginator` | Message | Gateway SMTP | likely | 4 | [019fe7a8](events/019fe7a8-440a-711f-960c-4cbcc394af77.md) |
| `ForeignErrorRoutedToReversePath` | Message | Gateway SMTP | certain | 3 | [019fe7a8](events/019fe7a8-440a-711e-b0ec-e34591d08671.md) |
| `ForeignTransportEnvelopeCapabilityRecorded` | Next-Hop Destination | Gateway operator | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-7122-a417-d18f55de6dce.md) |
| `GatewayDesignatedForTransportBoundary` | Server Configuration | Domain administrator or route operator | doubtful | 4 | [019fe7a8](events/019fe7a8-440a-7128-97d0-01bb1c8e4057.md) |
| `HeaderAddressTransformedForInternet` | Message | Gateway SMTP | certain | 4 | [019fe7a8](events/019fe7a8-440a-711c-9e37-0e77d8767dd5.md) |
| `HeaderFieldRewrittenAtGateway` | Message | Gateway SMTP | certain | 4 | [019fe7a8](events/019fe7a8-440a-7113-ba54-08abf7c99af7.md) |
| `LayeringSemanticsPreservedAcrossBoundary` | Message | Gateway SMTP | doubtful | 0 | [019fe7a8](events/019fe7a8-440a-7126-ad98-5a61a464b0e9.md) |
| `MessageCrossedTransportEnvironmentBoundary` | Message | Gateway SMTP | certain | 3 | [019fe7a8](events/019fe7a8-440a-7110-a4e3-28838bdedda8.md) |
| `MessageGatewayedFromForeignMailSystem` | Message | Gateway SMTP, inbound | likely | 4 | [019fe7a8](events/019fe7a8-440a-7123-b8ff-24e067587a60.md) |
| `MessageGatewayedFromHeaderFieldsAlone` | Message | Gateway SMTP, inbound | likely | 3 | [019fe7a8](events/019fe7a8-440a-7124-8461-5d9cad60cf53.md) |
| `MessageRemailedIntoOriginatingEnvironment` | Message | Header-section-only remailer in the foreign environment | likely | 3 | [019fe7a8](events/019fe7a8-440a-7125-ba18-13b1fffd595a.md) |
| `MessageTransformedByGateway` | Message | Gateway SMTP | likely | 2 | [019fe7a8](events/019fe7a8-440a-7111-a3e2-af7165f17ca5.md) |
| `NonconformingHeaderAddressForwardedIntoInternet` | Message | Gateway SMTP | likely | 3 | [019fe7a8](events/019fe7a8-440a-711d-a35f-08be1ae5496d.md) |
| `NonstandardAddressFormatAccepted` | Transaction | Gateway SMTP | likely | 3 | [019fe7a8](events/019fe7a8-440a-7119-bf2a-86eeaf0998b9.md) |
| `PrivateRecipientDisclosedByEnvelopeFolding` | Message | Gateway SMTP | likely | 3 | [019fe7a8](events/019fe7a8-440a-7115-b043-4b0faf608ce9.md) |
| `ReceivedLineOmittedAtBoundaryCrossing` | Message | Gateway SMTP | likely | 2 | [019fe7a8](events/019fe7a8-440a-7117-b54d-1ca6c0aa9069.md) |
| `ReturnPathDefaultedToOriginatorAddress` | Message | Gateway SMTP | likely | 2 | [019fe7a8](events/019fe7a8-440a-7121-9b5d-b518ebcfb90a.md) |
| `SourceRouteIgnoredByGateway` | Recipient | Gateway SMTP | likely | 3 | [019fe7a8](events/019fe7a8-440a-711b-ae73-fe364dd19ac1.md) |
| `ValidAddressFormatRefusedByGateway` | Transaction | Gateway SMTP | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-711a-b8ad-b8c0f706e1bd.md) |
| `ViaClauseOmittedByGateway` | Message | Gateway SMTP | doubtful | 0 | [019fe7a8](events/019fe7a8-440a-7118-9483-df5aa813b22c.md) |

### 10. Alias and List Expansion — 32 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AddressExpansionModelLeftUnsupported` | Server Configuration | Site operator | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-713d-85d6-f6ae537d695f.md) |
| `AddressExpansionSupportEnabled` | Server Configuration | Site operator | likely | 2 | [019fe7a8](events/019fe7a8-440a-713c-8741-01c7daa6b600.md) |
| `AliasExpanded` | Message | Alias resolver (the RFC's "recipient mailer") | certain | 3 | [019fe7a8](events/019fe7a8-440a-712a-b58f-cd61d6bf56a0.md) |
| `BccHeaderFieldRemoved` | Message | Submission client MTA | certain | 2 | [019fe7a8](events/019fe7a8-440a-7143-b6a9-c99b538c2ad6.md) |
| `BlindCopyRecipientAddressedInEnvelopeOnly` | Transaction | SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-7141-aa29-ee6d2229f411.md) |
| `BlindCopyRecipientCopiedFromBccHeaderField` | Transaction | Submission client MTA | likely | 2 | [019fe7a8](events/019fe7a8-440a-7142-957f-315d77aa313e.md) |
| `BlindCopyRecipientDisclosed` | Message | Message recipient | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-7148-b581-9aab779a32a4.md) |
| `BlindCopySentAsSeparateTransaction` | Transaction | Sending SMTP client | certain | 2 | [019fe7a8](events/019fe7a8-440a-7145-a3f6-5e47cdc0467e.md) |
| `DeliveryAcceptedByList` | Message | Mailing list expander acting as a full MUA | likely | 2 | [019fe7a8](events/019fe7a8-440a-7134-b8ce-36dac63c47a8.md) |
| `DuplicateRecipientEliminatedBySourceExpansion` | Message | Originating system | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-7140-a4fb-2a8a41a07d08.md) |
| `EmptyBccHeaderFieldInserted` | Message | Submission client MTA | certain | 0 | [019fe7a8](events/019fe7a8-440a-7144-a3fb-a847697615a2.md) |
| `ErrorMessageReturnedToListAdministrator` | Message | Delivery agent | likely | 3 | [019fe7a8](events/019fe7a8-440a-7136-a261-f2deb844ca8a.md) |
| `ErrorMessageReturnedToOriginator` | Message | Delivery agent | likely | 3 | [019fe7a8](events/019fe7a8-440a-7137-8ab0-69f6d2407fc8.md) |
| `ExpansionDeferredUntilAfterAcceptance` | Server Configuration | Site operator | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-713e-8759-819b5989fd6c.md) |
| `HeaderSectionAlteredFromDeducedEnvelopeRelationship` | Message | Receiving SMTP server | likely | 5 | [019fe7a8](events/019fe7a8-440a-7147-b720-046465dca714.md) |
| `HeaderSectionModifiedByExpander` | Message | Mailing list expander | likely | 4 | [019fe7a8](events/019fe7a8-440a-712e-8d98-99d8ebf60170.md) |
| `ListAdministratorChanged` | List | List administrator | likely | 3 | [019fe7a8](events/019fe7a8-440a-713b-bc08-0f6c21197c8c.md) |
| `ListExpanded` | Message | Mailing list expander (the RFC's "recipient mailer") | certain | 3 | [019fe7a8](events/019fe7a8-440a-712b-a653-d0eabbed96f7.md) |
| `ListMemberAdded` | List | List administrator | likely | 3 | [019fe7a8](events/019fe7a8-440a-7139-b12d-c18748366ea1.md) |
| `ListMemberRemoved` | List | List administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-713a-9370-c0fce44bbb39.md) |
| `ListReclassifiedAsFullMua` | List | Mailing list expander | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-7133-a0d3-ccfb2611130c.md) |
| `ListReturnPathLeftUnchanged` | Message | Mailing list expander | likely | 3 | [019fe7a8](events/019fe7a8-440a-712d-9d90-3f7210bca4d0.md) |
| `ListTreatedAsTransitContinuation` | List | Mailing list expander | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-7132-abfc-51b5a986d12b.md) |
| `MailingListDefined` | List | List administrator | likely | 3 | [019fe7a8](events/019fe7a8-440a-7138-a103-0700c0fe0849.md) |
| `MailingListExpandedAtSource` | Message | Originating system | likely | 3 | [019fe7a8](events/019fe7a8-440a-713f-84af-1ce6fbf979a8.md) |
| `MessageCopyCreated` | Queue Entry | Mailing list expander / alias resolver | likely | 3 | [019fe7a8](events/019fe7a8-440a-712f-8d37-c0d9d3eb2cd5.md) |
| `MessageModifiedByList` | Message | Mailing list expander | likely | 3 | [019fe7a8](events/019fe7a8-440a-7131-afc5-ce3897b8d956.md) |
| `MessagePostedByList` | Message | Mailing list expander acting as a full MUA | likely | 4 | [019fe7a8](events/019fe7a8-440a-7135-811e-661dd4af5309.md) |
| `PseudoMailboxExpanded` | Message | Mailing list expander / alias resolver (the RFC's "recipient mailer") | certain | 4 | [019fe7a8](events/019fe7a8-440a-7129-8b75-9950087d0ea7.md) |
| `RecipientListCopiedIntoHeaderSection` | Message | SMTP client or receiving server | likely | 3 | [019fe7a8](events/019fe7a8-440a-7146-bed8-60de3fdc2561.md) |
| `RecipientSuppressedByHeuristic` | Message | Mailing list expander | likely | 4 | [019fe7a8](events/019fe7a8-440a-7130-9e52-b743b0ccdb46.md) |
| `ReturnPathRewrittenToListAdministrator` | Message | Mailing list expander | certain | 4 | [019fe7a8](events/019fe7a8-440a-712c-8e3f-bed2b4cd8729.md) |

### 11. Forwarding and Address Correction — 16 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AddressBookEntryUpdated` | Correspondent Address Record | SMTP client or user agent | certain | 3 | [019fe7a8](events/019fe7a8-440a-7107-aa4d-48430ea83016.md) |
| `AddressCorrectionAcceptedFromUnauthenticatedServer` | Correspondent Address Record | SMTP client or user agent | likely | 2 | [019fe7a8](events/019fe7a8-440a-7108-b5fb-8a47292aa290.md) |
| `AddressUpdateDisclosureRestricted` | Server Configuration | Site operator | certain | 4 | [019fe7a8](events/019fe7a8-440a-710b-b7ff-93cb1e99f134.md) |
| `CorrectedAddressFoundUnreachableBySender` | Message | Sending SMTP client | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-710a-9eef-2983e07827ca.md) |
| `CorrectedDestinationDisclosed` | Recipient | Receiving SMTP server acting as forwarder | certain | 2 | [019fe7a8](events/019fe7a8-440a-7104-b563-70904e592673.md) |
| `ForwardingAliasProvisioned` | Mailbox | Administrator | certain | 2 | [019fe7a8](events/019fe7a8-440a-7100-a132-71074c7077d8.md) |
| `ForwardingEntryRetiredOnAssumedClientUpdate` | Mailbox | Receiving SMTP server or its administrator | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-710d-b5dc-09990d2e791f.md) |
| `MailForwardingRuleCreated` | Mailbox | Mailbox owner | likely | 2 | [019fe7a8](events/019fe7a8-440a-7102-99eb-34b7cdbb7aa9.md) |
| `MailboxRelocationRecorded` | Mailbox | Administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7101-a429-0d7cf485f706.md) |
| `MessageForwardedSilently` | Recipient | Receiving SMTP server acting as forwarder | certain | 2 | [019fe7a8](events/019fe7a8-440a-7103-82f8-dd4c1f0e3b4f.md) |
| `MessageRelayedToCorrectedAddress` | Message | Forwarding SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7105-9b39-e8a7db88a063.md) |
| `MessageResubmittedToCorrectedAddress` | Message | Sending SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-7109-b497-7933bcedd374.md) |
| `MessageReturnedAsNonDeliverable` | Message | Delivery agent | likely | 3 | [019fe7a8](events/019fe7a8-440a-7106-a203-8960c68a658c.md) |
| `SensitiveAddressInadvertentlyDisclosed` | Mailbox | Receiving SMTP server | likely | 3 | [019fe7a8](events/019fe7a8-440a-710c-b31b-92bfb80bc319.md) |
| `ServerChosenForDisclosureBehavior` | Server Configuration | Site operator | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-710f-8c17-966239feb9ed.md) |
| `VerifiedAddressReportedAsForwarded` | Session | Receiving SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-710e-8159-b728e8eaa687.md) |

### 12. Non-Delivery Notification — 20 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AutoReplySentToNullReversePathMessage` | Notification | Automated email processor (vacation responder, ticketing system, autoresponder) | likely | 0 | [019fe7a8](events/019fe7a8-440a-715b-bb9e-fb55b211dadb.md) |
| `BounceSentDespiteHostileContent` | Notification | Receiving site | likely | 2 | [019fe7a8](events/019fe7a8-440a-714f-b7c4-2bf272dbbc8d.md) |
| `BounceSuppressedForHostileContent` | Message | Receiving site | certain | 1 | [019fe7a8](events/019fe7a8-440a-714e-91ef-4df946116530.md) |
| `FailedNotificationRedirectedToPostmaster` | Notification | MTA, per host configuration | likely | 2 | [019fe7a8](events/019fe7a8-440a-7158-aef6-a1560c2c3751.md) |
| `FaultyMessageDeliveredAnyway` | Message | Delivery SMTP server | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-7156-8154-dff40402d2b6.md) |
| `HostileContentDetected` | Message | Receiving site (mechanism unspecified by this RFC) | likely | 0 | [019fe7a8](events/019fe7a8-440a-7150-a306-b3ee1eae21be.md) |
| `InvalidReturnAddressDropPolicyAdopted` | Server Configuration | Site administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7155-813b-1c5992893843.md) |
| `MessageDroppedForInvalidReturnAddress` | Message | Receiving site, applying its own policy | likely | 2 | [019fe7a8](events/019fe7a8-440a-7154-acd1-1e554417260c.md) |
| `MessageJudgedSeriouslyFraudulent` | Message | Receiving site | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-7152-8f56-e2b6348628c9.md) |
| `MessageSilentlyDropped` | Message | Receiving site operator, through receiver-SMTP policy | certain | 1 | [019fe7a8](events/019fe7a8-440a-7151-be2e-8818362a304c.md) |
| `NonDeliveryNotificationComposed` | Notification | Notifying SMTP server (relay, or the receiver that accepted the message) | certain | 3 | [019fe7a8](events/019fe7a8-440a-7149-9826-024c1c7f4e59.md) |
| `NonDeliveryNotificationSent` | Notification | Notifying SMTP server, now acting as an originating system | certain | 2 | [019fe7a8](events/019fe7a8-440a-714b-99a1-8c6d1473ed76.md) |
| `NotificationDeliveryFailed` | Notification | Sending MTA | certain | 2 | [019fe7a8](events/019fe7a8-440a-7157-a7a9-5f4a60cb59df.md) |
| `NotificationSentAboutNotificationFailure` | Notification | Non-conforming relay or delivery SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440a-714d-b43a-57bb5c7f7fbf.md) |
| `NotificationSuppressedToPreventLoop` | Message | Relay or delivery SMTP server; equivalently the queuing strategy | certain | 1 | [019fe7a8](events/019fe7a8-440a-714c-a2b1-7ccb6055b47c.md) |
| `NullReversePathReplacedOnForward` | Notification | Forwarding MTA, alias resolver or mailing list expander | likely | 2 | [019fe7a8](events/019fe7a8-440a-715a-baff-97a2ac81d01d.md) |
| `NullReversePathSet` | Notification | Notifying SMTP server | certain | 0 | [019fe7a8](events/019fe7a8-440a-714a-bb4f-06262070c0f3.md) |
| `NullReversePathUsedForNonNotification` | Message | Sending SMTP client | likely | 0 | [019fe7a8](events/019fe7a8-440a-715c-8d87-c53db79ff7e5.md) |
| `PostmasterRedirectConfigured` | Server Configuration | Host administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7159-8aea-efc008f31477.md) |
| `ReturnAddressDeterminedInvalid` | Message | Receiving site | likely | 2 | [019fe7a8](events/019fe7a8-440a-7153-b1ca-5050c9ce7078.md) |

### 13. Address Diagnostics — 32 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AddressDiagnosticsRestrictedToAuthenticatedRequestors` | Server Configuration | Site administrator | likely | 3 | [019fe7a8](events/019fe7a8-440a-7170-976d-74fd64be426b.md) |
| `AddressVerificationDeclinedAsUnverifiable` | Address Diagnostic Request | Receiving SMTP Server | likely | 1 | [019fe7a8](events/019fe7a8-440a-7162-bcfb-8b13288a1f8d.md) |
| `AddressesHarvestedViaExpn` | List | Address harvester (a requesting client acting against the list's interest) | doubtful | 3 | [019fe7a8](events/019fe7a8-440a-716c-9745-07226136dccc.md) |
| `AmbiguousMailboxAlternativesDisclosed` | Address Diagnostic Request | Receiving SMTP Server | likely | 3 | [019fe7a8](events/019fe7a8-440a-715f-a771-72a7caddfd0a.md) |
| `CrossTypeVerificationRefused` | Address Diagnostic Request | Receiving SMTP Server | likely | 3 | [019fe7a8](events/019fe7a8-440a-7161-985a-d3ffb0cf5795.md) |
| `DiagnosticCommandAcceptedWithoutSessionInitialization` | Session | Receiving SMTP Server | likely | 2 | [019fe7a8](events/019fe7a8-440a-717a-aca8-d16bf283840a.md) |
| `DiagnosticCommandRejectedAsOutOfSequence` | Session | Receiving SMTP Server | likely | 1 | [019fe7a8](events/019fe7a8-440a-717b-83bd-3ace9e2a0f84.md) |
| `DuplicateEliminationBySourceExpansionAttempted` | Message | Originating System | doubtful | 3 | [019fe7a8](events/019fe7a8-440a-716d-9239-3bb4449ae956.md) |
| `ExpnAccessDenied` | List | Receiving SMTP Server | likely | 2 | [019fe7a8](events/019fe7a8-440a-716b-bcd3-970cfc3079b5.md) |
| `ExpnServiceExtensionRegistered` | Extension Registry | Receiving SMTP Server | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-7173-aed9-9257772a0db9.md) |
| `ExpnSupportDisabled` | Server Configuration | Site administrator | certain | 2 | [019fe7a8](events/019fe7a8-440a-716f-b264-a0ee5cd9c053.md) |
| `FreeFormVerificationTextReturned` | Address Diagnostic Request | Receiving SMTP Server | doubtful | 3 | [019fe7a8](events/019fe7a8-440a-7167-9282-0de3b9128255.md) |
| `HelpInformationSent` | Session | Receiving SMTP Server | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-7174-b82c-3227188d998f.md) |
| `HelpRequestRefused` | Session | Receiving SMTP Server | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-7175-9748-e2659fb327ee.md) |
| `HelpSupportConfigured` | Server Configuration | Site administrator | doubtful | 4 | [019fe7a8](events/019fe7a8-440a-7176-9e4c-ac72912ca5be.md) |
| `MailboxExistenceDisclosedViaVrfy` | Address Diagnostic Request | Receiving SMTP Server | likely | 4 | [019fe7a8](events/019fe7a8-440a-715d-ada1-f672d03151ac.md) |
| `MailboxNameReportedAsAmbiguous` | Address Diagnostic Request | Receiving SMTP Server | likely | 1 | [019fe7a8](events/019fe7a8-440a-715e-bc0f-e366c50a5408.md) |
| `MailingListAccessRestricted` | List | Mailing list administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7171-a27b-979e66210b1d.md) |
| `MailingListMembershipDisclosedViaExpn` | List | Receiving SMTP Server | likely | 4 | [019fe7a8](events/019fe7a8-440a-7169-866c-cc2ceec63b50.md) |
| `MailingListVerifiedAsUser` | Address Diagnostic Request | Receiving SMTP Server | likely | 1 | [019fe7a8](events/019fe7a8-440a-7160-b56d-911cd94906d7.md) |
| `NoopAcknowledged` | Session | Receiving SMTP Server | doubtful | 0 | [019fe7a8](events/019fe7a8-440a-7178-90cc-83a2512d587e.md) |
| `NoopParameterActedUpon` | Session | Receiving SMTP Server | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-7179-a292-9ee84342e974.md) |
| `ServerIdentityDisclosurePolicySet` | Server Configuration | Site operator | likely | 3 | [019fe7a8](events/019fe7a8-440a-7177-b7fe-c8a926255f0e.md) |
| `SourceRouteReturnedInVerificationReply` | Address Diagnostic Request | Receiving SMTP Server | doubtful | 3 | [019fe7a8](events/019fe7a8-440a-7168-9ac5-08765ea32e58.md) |
| `TransactionStateDestroyedByDiagnosticCommand` | Transaction | Receiving SMTP Server | doubtful | 3 | [019fe7a8](events/019fe7a8-440a-717c-ad81-a72beeea70e7.md) |
| `UserNameExpandedAsSingletonList` | Address Diagnostic Request | Receiving SMTP Server | likely | 2 | [019fe7a8](events/019fe7a8-440a-716a-b469-0d6dbfd9e75c.md) |
| `UserNameRecognitionRuleConfigured` | Server Configuration | Site administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7172-a1e0-51858240e65e.md) |
| `VerificationDeclinedAsDisabled` | Address Diagnostic Request | Receiving SMTP Server | likely | 3 | [019fe7a8](events/019fe7a8-440a-7163-9810-363ff7ba007f.md) |
| `VerificationDeniedWithoutChecking` | Address Diagnostic Request | Receiving SMTP Server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7166-a174-60dafb495a7f.md) |
| `VerificationFalselyClaimed` | Address Diagnostic Request | Receiving SMTP Server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7165-8e5e-132748f9616f.md) |
| `VerificationRefusedAsUnimplemented` | Address Diagnostic Request | Receiving SMTP Server | likely | 2 | [019fe7a8](events/019fe7a8-440a-7164-916d-018d2552121e.md) |
| `VrfySupportDisabled` | Server Configuration | Site administrator | certain | 2 | [019fe7a8](events/019fe7a8-440a-716e-81c9-9aac60839973.md) |

### 14. Protocol Fault Handling — 25 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AddressCorrectedToFqdn` | Message | Originating or submission SMTP server | certain | 3 | [019fe7a8](events/019fe7a8-440a-71d2-9dc0-784b2baca1aa.md) |
| `BareReplyCodeTolerated` | Reply | SMTP client | likely | 2 | [019fe7a8](events/019fe7a8-440a-71c0-8f0f-8a1e356d060f.md) |
| `CommandRejectedAsMalformed` | Session | Receiving SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-71c7-8957-25540ff56dfe.md) |
| `CommandRejectedAsUnimplemented` | Session | Receiving SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-71c8-9882-2cccb33649bd.md) |
| `CommandSentWithoutAwaitingReply` | Session | SMTP client | likely | 3 | [019fe7a8](events/019fe7a8-440a-71cb-a1f4-68380e89e78f.md) |
| `InconsistentMultilineReplyEmitted` | Reply | Receiving SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-71bf-8c3e-9c5340d17396.md) |
| `MailParametersRejected` | Transaction | Receiving SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-71ca-82a5-8b75a9218007.md) |
| `MailTransactionTerminatedOnInvalidReply` | Transaction | SMTP client | certain | 1 | [019fe7a8](events/019fe7a8-440a-71c2-9de9-b817fdd244f3.md) |
| `MalformedMessagePassedOnUnchanged` | Message | Receiving or relaying SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-71d5-98fb-209e5951d909.md) |
| `MalformedMessageRejected` | Message | Receiving or relaying SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-71d4-908f-b7996291c0e7.md) |
| `MessageIdAdded` | Message | Originating or submission SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-71d0-9288-5cc39353c098.md) |
| `MultilineReplyCompleted` | Reply | SMTP client | certain | 1 | [019fe7a8](events/019fe7a8-440a-71be-8196-0465abb36645.md) |
| `NonStandardReplyCodeInvented` | Server Configuration | Server operator or implementer | likely | 3 | [019fe7a8](events/019fe7a8-440a-71c3-afc0-e337fe031b21.md) |
| `OriginationDateAdded` | Message | Originating or submission SMTP server | certain | 3 | [019fe7a8](events/019fe7a8-440a-71d1-9169-73a713a16219.md) |
| `OversizedCommandLineRejected` | Session | Receiving SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-71cc-84e6-4fa27073e654.md) |
| `OversizedPathRejected` | Transaction | Receiving SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-71ce-964e-d9eb069070ec.md) |
| `PermanentFailureReturned` | Session | Receiving SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-71c5-a98b-6cc3d88e8c4e.md) |
| `PreviouslyAcceptedRecipientsDiscarded` | Transaction | Receiving SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-71cf-9da1-62ec00cd71eb.md) |
| `RepairDocumentedInTraceHeader` | Message | Originating or submission SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-71d3-84dc-586a20e9d31a.md) |
| `ReplyIssued` | Session | Receiving SMTP server | certain | 0 | [019fe7a8](events/019fe7a8-440a-71bd-a0e0-70e6bc3a1c73.md) |
| `RequiredCommandRejectedAsUnrecognized` | Session | Receiving SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-71c6-993e-05270e5ade38.md) |
| `ShortCommandLineRejectedAsTooLong` | Session | Receiving SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-71cd-a412-159fa009488a.md) |
| `TransientFailureReturned` | Session | Receiving SMTP server | certain | 1 | [019fe7a8](events/019fe7a8-440a-71c4-9b2d-ba3ae63b3741.md) |
| `UnimplementedCapabilityAdvertised` | Server Configuration | Server operator or implementer | likely | 2 | [019fe7a8](events/019fe7a8-440a-71c9-89c8-26de67322798.md) |
| `UnrecognizedReplyCodeInterpretedByFirstDigit` | Session | SMTP client | certain | 1 | [019fe7a8](events/019fe7a8-440a-71c1-a9b8-4dede1d1678b.md) |

### 15. Server Configuration and Provisioning — 33 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AddressLiteralTagRegistered` | Extension Registry | IANA | likely | 2 | [019fe7a8](events/019fe7a8-440a-719a-ab2c-a18d6ba45c24.md) |
| `BilateralAgreementEstablished` | Server Configuration | Operators of both systems | likely | 4 | [019fe7a8](events/019fe7a8-440a-719d-b857-7bb79947d0af.md) |
| `ClientRecipientsPerTransactionLimitConfigured` | Server Configuration | Client operator | certain | 2 | [019fe7a8](events/019fe7a8-440a-7193-8002-8db6cb21ec57.md) |
| `ConcurrentOutboundTransactionLimitConfigured` | Server Configuration | Client operator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7194-8c01-bcc086788427.md) |
| `DisabledVerificationReplyCodeConfigured` | Server Configuration | Site administrator | likely | 3 | [019fe7a8](events/019fe7a8-440a-7187-847b-53e6aa43b514.md) |
| `DomainlessPostmasterSupportProvisioned` | Server Configuration | Administrator | certain | 1 | [019fe7a8](events/019fe7a8-440a-717f-bd77-fb84b4eefc1f.md) |
| `ExpnSupportEnabled` | Server Configuration | Operator | likely | 1 | [019fe7a8](events/019fe7a8-440a-7186-9889-c26f4e8f0992.md) |
| `MailAcceptancePolicyConfigured` | Server Configuration | Site operator | likely | 2 | [019fe7a8](events/019fe7a8-440a-718e-bf8f-d490e3143f01.md) |
| `MailProblemContactPointPublished` | Server Configuration | Server operator | likely | 2 | [019fe7a8](events/019fe7a8-440a-717e-990e-9c0827afa572.md) |
| `MailServiceDomainAdded` | Server Configuration | Administrator | likely | 3 | [019fe7a8](events/019fe7a8-440a-7180-8529-34813a2e73f7.md) |
| `MinimumCommandSetProvisioned` | Server Configuration | Implementer | certain | 1 | [019fe7a8](events/019fe7a8-440a-7183-9269-20f14bc35c38.md) |
| `NonConformingRecipientLimitConfigured` | Server Configuration | Server operator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7192-af0e-7345d448eefc.md) |
| `PostmasterMailBlockLifted` | Server Configuration | Operator | doubtful | 2 | [019fe7a8](events/019fe7a8-440a-7182-b06a-69e724905cba.md) |
| `PostmasterMailBlocked` | Server Configuration | Operator | likely | 3 | [019fe7a8](events/019fe7a8-440a-7181-959c-50ee1331ab2a.md) |
| `PostmasterMailboxProvisioned` | Server Configuration | Administrator | likely | 1 | [019fe7a8](events/019fe7a8-440a-717d-80ad-7e476d2e6473.md) |
| `ProgramDeliveryMailboxNameAssigned` | Server Configuration | Administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7189-9b90-c51aba3ed2a3.md) |
| `QueuingStrategyAdopted` | Server Configuration | Implementer | likely | 1 | [019fe7a8](events/019fe7a8-440a-7196-932a-a32fd1753f17.md) |
| `RecipientLimitConfigured` | Server Configuration | Server operator | likely | 4 | [019fe7a8](events/019fe7a8-440a-7191-8ac2-44f506d6d145.md) |
| `RelayDomainAuthorized` | Server Configuration | Administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-718b-b633-38a286b245f6.md) |
| `RelayFilteringCapabilityImplemented` | Server Configuration | Implementer | certain | 0 | [019fe7a8](events/019fe7a8-440a-718d-a0c6-ebbfabd010c9.md) |
| `RelayPolicyConfigured` | Server Configuration | Mail system operator | likely | 3 | [019fe7a8](events/019fe7a8-440a-718a-82f2-07640150439c.md) |
| `RelaySourceAuthorized` | Server Configuration | Site administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-718c-ad4b-95cf200e6744.md) |
| `ReplyTextCustomized` | Server Configuration | Server operator | likely | 4 | [019fe7a8](events/019fe7a8-440a-7185-93cd-a0790a2fd66c.md) |
| `RequiredCommandLeftUnimplemented` | Server Configuration | Implementer | likely | 2 | [019fe7a8](events/019fe7a8-440a-7184-89df-bf63820221d3.md) |
| `ServiceExtensionEnabled` | Server Configuration | Mail system administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7198-8a24-b02889e26b51.md) |
| `ServiceExtensionRegistered` | Extension Registry | IANA | certain | 5 | [019fe7a8](events/019fe7a8-440a-7199-8d2c-998e2c0e3926.md) |
| `SizeLimitConfigured` | Server Configuration | Server operator | likely | 3 | [019fe7a8](events/019fe7a8-440a-718f-a830-b1d4ad7b96a5.md) |
| `SizeServiceExtensionImplemented` | Server Configuration | Implementer | certain | 1 | [019fe7a8](events/019fe7a8-440a-7190-80f5-482a99aa796f.md) |
| `SmtpListenerStarted` | Server Configuration | Operator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7197-ad58-b51647f13669.md) |
| `TimeoutValuesReconfigured` | Server Configuration | Administrator | likely | 3 | [019fe7a8](events/019fe7a8-440a-7195-86e2-69906dc13fa2.md) |
| `TraceHeaderFieldRegistered` | Extension Registry | IANA | certain | 2 | [019fe7a8](events/019fe7a8-440a-719c-8711-7890d1f09bc8.md) |
| `TransmissionTypeRegistered` | Extension Registry | IANA | likely | 2 | [019fe7a8](events/019fe7a8-440a-719b-804b-6a83671cd627.md) |
| `VerificationRestrictedToAuthenticatedRequestors` | Server Configuration | Site administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-7188-9253-0a30f1cbc862.md) |

### 16. Abuse, Security and Registry — 22 events

| Event | Stream root | Actor | Confidence | Fields | File |
|:--|:--|:--|:--|--:|:--|
| `AddressBookPoisonedByForgedReply` | Address Book | Man in the middle, through the client | likely | 3 | [019fe7a8](events/019fe7a8-440a-71e2-9853-acb3fd584863.md) |
| `AddressesHarvestedFromMailingList` | Server Configuration | Spammer / harvesting client | likely | 4 | [019fe7a8](events/019fe7a8-440a-71e0-9abc-4575147fcf3e.md) |
| `BlindCopyRecipientDisclosedInForClause` | Message | Receiving SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440b-7003-b97c-2aaa5f33ccc8.md) |
| `EnvelopeReturnPathSetToAnotherUsersAddress` | Message | Originating user / user agent | likely | 3 | [019fe7a8](events/019fe7a8-440a-71d8-b559-3165f233781a.md) |
| `ExtensionApprovedByIesg` | Extension Registry | IESG | likely | 2 | [019fe7a8](events/019fe7a8-440b-7007-98f2-3679a61e0c3a.md) |
| `HostileTrafficDetected` | Session | Receiving SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440b-7005-bab3-a41d18828d46.md) |
| `InternalHostNamesDisclosedInTraceField` | Message | Receiving or relaying SMTP server (conforming) | likely | 1 | [019fe7a8](events/019fe7a8-440b-7002-9a63-106ab35a0070.md) |
| `MailingListProtectionInstalled` | List | Mailing list administrator | likely | 2 | [019fe7a8](events/019fe7a8-440a-71e1-9e06-1ad47914574d.md) |
| `MessageSpoofedToAppearFromElsewhere` | Session | Attacker (any party able to open an SMTP connection) | likely | 2 | [019fe7a8](events/019fe7a8-440a-71d6-ae4c-1bb73163a6ed.md) |
| `PermanentMailboxAddressEstablishedForUser` | Mailbox | Originating mail system operator | likely | 2 | [019fe7a8](events/019fe7a8-440a-71da-b59a-2d03830501c5.md) |
| `ReplacementAddressAssociatedWithMailbox` | Mailbox | Mailbox administrator | likely | 2 | [019fe7a8](events/019fe7a8-440b-7004-8d79-459977811478.md) |
| `ReturnPathAlterationRestricted` | Server Configuration | Site operator | likely | 2 | [019fe7a8](events/019fe7a8-440a-71d9-a431-b98e35c52f0a.md) |
| `ServerIdentityDisclosureConfigured` | Server Configuration | Site operator | likely | 3 | [019fe7a8](events/019fe7a8-440b-7000-bd82-b4a22df6dc99.md) |
| `ServerRenderedInaccessibleByAttack` | Server Configuration | Attacker | doubtful | 2 | [019fe7a8](events/019fe7a8-440b-7006-949b-75511fab1699.md) |
| `ServerTypeAndVersionDisclosed` | Session | Receiving SMTP server | doubtful | 2 | [019fe7a8](events/019fe7a8-440b-7001-85c9-b5fd10ce1897.md) |
| `SpoofedOriginDetected` | Message | Expert examiner (human or downstream analyzer) | doubtful | 1 | [019fe7a8](events/019fe7a8-440a-71d7-b986-855c39abb064.md) |
| `TransportLevelAuthenticationCompleted` | Session | SMTP client and SMTP server together | likely | 1 | [019fe7a8](events/019fe7a8-440a-71db-9950-3f97140d5542.md) |
| `UnregisteredKeywordOffered` | Session | Receiving SMTP server | likely | 1 | [019fe7a8](events/019fe7a8-440b-7008-9d17-be3f3ba0f76e.md) |
| `VerificationAlwaysRefusedWith550` | Session | Receiving SMTP server | likely | 0 | [019fe7a8](events/019fe7a8-440a-71df-8845-883e52d8fe78.md) |
| `VerificationCommandDisabled` | Server Configuration | Site administrator | certain | 2 | [019fe7a8](events/019fe7a8-440a-71dc-b57f-b4e3055909b9.md) |
| `VerificationDeclinedWithoutDisclosure` | Session | Receiving SMTP server | likely | 2 | [019fe7a8](events/019fe7a8-440a-71dd-85b0-b14c35453b95.md) |
| `VerificationFakedAfterSyntaxOnlyCheck` | Session | Receiving SMTP server | certain | 2 | [019fe7a8](events/019fe7a8-440a-71de-8231-4b8df26f5181.md) |
