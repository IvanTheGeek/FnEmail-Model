# Corrections to Event Model v0.1

Target: `/home/user/FnEmail/docs/event-model.md` (draft v0.1, 295 lines).

Sources are the two primary corpora on local disk. Shorthand used below:

- **BOOK** = `/tmp/claude-0/-home-user-FnEmail/c95cf585-89d5-5e54-9e14-0abdf455ed4a/scratchpad/book/` — Martin Dilger, *Understanding Eventsourcing*, foreword by Adam Dymitruk.
- **KIT** = `/tmp/claude-0/-home-user-FnEmail/c95cf585-89d5-5e54-9e14-0abdf455ed4a/scratchpad/npm/eventmodelers-agent-modeling-kit-0.1.74/templates/.claude/skills/` — canonical skill definitions (MIT, Nebulit/eventmodelers).

Where the two disagree, the disagreement is stated rather than reconciled.

Ordering is by severity: things that make the model unbuildable first, vocabulary last.

---

## 1. The `READ MODEL (decide)` lane is the one construct the corpus calls non-negotiable

**v0.1 claimed** (§3, lines 72–73): a single `SessionState` projection consulted by `Ehlo` and `MailFrom`, and a single `TransactionState` projection consulted by `RcptTo`, `BeginData` and `SubmitContent`. The lane is labeled `(decide)`. §1 glosses read models as feeding "UI and command decisions".

**The method says**: "**NEVER use DDD Aggregate pattern for state design** Every command handler must have its own minimal state projection derived from events. This is non-negotiable." (KIT `eventmodeling-brainstorming-events/SKILL.md:517`, and lines 423–432). The worked anti-pattern is a fat state object shared by several commands: "Loads unused data, couples unrelated commands, violates minimal state principle." The validator escalates it to a gate — Section 6, *Command State Read Models Validation (CRITICAL)*: "This is the PRIMARY validation gate. Violations are CRITICAL and must be fixed before approval." (KIT `eventmodeling-validating-event-models/SKILL.md:121–124, 173–215`). Its common-issues table lists the exact shape: "Command state too broad | Shared state used by 2+ commands | Split into per-command minimal state projections." A separate checklist item requires that "Read models serve actual query needs (not used by commands)."

**Why it matters**: v0.1 does not merely fall into this by accident — it names the lane `(decide)`, declaring the conflation on purpose. Everything downstream (slicing, testability, parallel team work) is built on top of it.

**Fix**: split into per-command minimal state named `[CommandName]State`, each rebuilt from events and discarded:

- `MailFromState { greeted, tls, authenticated, txnOpen }`
- `RcptToState { txnOpen, recipientCount, relayAllowed }`
- `SubmitContentState { txnOpen, acceptedRecipients[], declaredSize, dataPhase }`
- `AttemptRelayState { queueId, forwardPath, host, alreadyDelivered }`

Lifecycle per the kit: "Load events from stream / Replay via evolve() to rebuild state / Process command, emit outcome events / Discard state (it's ephemeral, not persisted)." The named anti-pattern there is precisely the mail-queue temptation — persisting the command state table "Duplicates event sourcing, loses audit trail." An SMTP queue table must not become the source of truth.

`DeliveryQueue` and `RelayPolicy` stay: they are genuine query read models serving an automation.

---

## 2. `DeliveryRequested` is a read-model row wearing an event's clothes

**v0.1 claimed** (§3.3, §4.1, §5.3): `SubmitContent` emits `MessageContentReceived` + `MessageAccepted` + **N** `DeliveryRequested`, one per accepted recipient, so that "partial success falls out for free".

**The method says**, in two parts:

*The two-event part is fine.* Dilger: "It's perfectly fine to apply more than one event in a command handler, although most often there is a one-to-one relationship." (BOOK `0326-implementing-the-first-slice-add-item.txt`, p.321). A whole figure is devoted to it — "Fig. 17.3 - A command can have multiple outcomes" — with a worked GWT producing `CartCreated` + `ItemAdded` from one `AddItem` (BOOK `0280-use-case-price-changed.txt`, pp.259–261). Notation rule: "When there are multiple events in a 'Given / When / Then' scenario, the order of events is important and follows a left-to-right sequence."

*The N-event part is a defect.* `DeliveryRequested` carries `queue_id, forward_path`. Both are already in `MessageAccepted` (`queue_id`, `recipients[]`). It contains zero new information; it is a per-row expansion of an existing event. KIT `eventmodeling-identifying-outputs/SKILL.md:87–170`: "if value changes multiple times, it's a read model"; do not model "Scheduled calculations (processor outputs that are pure calculation)" as events. KIT `eventmodeling-checking-completeness/SKILL.md:492–500`: "No aggregation events: Totals, averages, counts are read models, NOT events." The canonical shape puts the fan-out in the read model: "Processor-Todo-Lists are calculated based on the state of the System and typically provided by a Read Model." (BOOK `0514-pattern-processor-todo-list-pattern.txt`, p.493). The book's price-change chapter is the identical shape and resolves it identically — one `PriceChanged` → read model listing affected carts → processor issues N commands: "It is perfectly fine for a Processor to schedule more than one command." (BOOK `0408-implementing-automations.txt`, p.405).

**Why it matters**: v0.1 already half-built the correct version in §4 (`DeliveryQueue` as todo-list) and then also kept the N events, paying twice. It is also an unbounded-cardinality event write — RFC 5321 requires supporting at least 100 recipients — and it silently asserts that acceptance and queueing are one atomic transaction.

**Fix**: delete `DeliveryRequested`. `SubmitContent` emits `MessageContentReceived` + `MessageAccepted(recipients[])`. A read model `DeliveryQueue` projects one row per `(queue_id, forward_path)` from `MessageAccepted`. The delivery processor drains it and issues `AttemptRelay` per row. Partial delivery still falls out for free, because per-recipient outcomes are recorded by `RelayAccepted` / `RelayFailed` — those are facts.

---

## 3. There are no slices in the document

**v0.1 claimed**: the word "slice" appears three times — §5 title, §5.2 ("This single slice justifies…"), §7 ("Slice boundaries"). Zero slices are defined.

**The method says**: "A Feature Slice is the thinnest possible vertical cut through the model — exactly one decision or one query… Exactly one COMMAND (state-change), exactly one READMODEL (state-view), or one AUTOMATION's command — never combined." Slices are discovered by walking the timeline column by column, not designed (KIT `eventmodeling-slicing-event-models/SKILL.md:23–51, 77–129`). Completeness gate: "Every column with a COMMAND or READMODEL node must have a slice defined… otherwise it can never be built as a feature." (KIT `eventmodeling-checking-completeness/SKILL.md:151–174`). Naming: "Use the exact command, read model, or automation title as the slice title — don't invent a broader 'feature' name that spans multiple elements."

**Why it matters**: the slice is the unit that makes the model buildable. Without it §7's boundaries table is a diagram of intentions.

**Fix**: enumerate roughly 20 slices. State-change: `IdentifyClient`, `NegotiateTls`, `StartMailTransaction`, `AddRecipient`, `EnterDataPhase`, `SubmitContent`, `ResetTransaction`, `CloseSession`, `ResolveRoute`, `AttemptRelay`, `GenerateBounce`, `GiveUpDelivery`. State-view: `SessionCapabilities`, `RelayPolicy`, `DeliveryQueue`, `MessageStatus`, `PostmasterQueueView`. Automation: `QueueRunner`, `RetryScheduler`, `BounceGenerator`, `ProtocolResponder`.

No slice called "Inbound Mail Handling"; no slice spanning `AttemptRelay` and `DeliveryQueue` — "that's two slices, not one." §4's `ResolveRoute → AttemptRelay` box is three slices, not one.

Cross-slice rule that §7 needs: "Slices depend on each other only through events — never directly." The book allows exactly one structural exception: "One typical dependency between slices is an automation that uses the corresponding Read Model… For this the Read Model exposes a specific query interface." (BOOK `0181-vertical-slicing.txt`, pp.163–169).

Processors must be named as first-class elements — the automation slice is the only shape with events on both ends (`EVENT(s) → AUTOMATION → COMMAND → EVENT(s)`), and it is most of what FnEmail is. v0.1 names none of its processors.

---

## 4. No Role Catalog, no actor attribution

**v0.1 claimed**: nothing. There is no roles section. No command is attributed to anyone.

**The method says**: "Before brainstorming events, define **who** interacts with the system. Every event model needs an explicit role catalog — without it, downstream steps (storyboarding, commands, scenarios) lack clarity on who does what." (KIT `eventmodeling-brainstorming-events/SKILL.md:374`, section at 372–420). Two categories — Human roles and System actors — each with Name, Description, Key actions, and a Permissions boundary ("What can this role NOT do?"). The validator makes the absence a CRITICAL failure twice: "CRITICAL: No Role Catalog found — commands have no actor attribution", and "No command uses generic 'User'" (KIT `eventmodeling-validating-event-models/SKILL.md:109–120, 231–234`). v0.1 attributes nothing at all, which is worse than attributing to "User".

**Why it matters**: it gates storyboarding, scenarios, and the Conway's Law step. It is also where `RelayPolicy` actually belongs conceptually — "what can this role NOT do" *is* the relay rule.

**Fix**: catalog — Sending MTA (unauthenticated peer), Submitting Client (authenticated, :587), Receiving MTA, Queue Runner, MX/DNS Resolver, Bounce Generator (system actors), Postmaster and Mailbox Owner (humans). Then honour the gates: "Every human role has at least one command and one read model" and "Every human role from the Role Catalog has at least one screen."

---

## 5. "The wire is the wireframe" is good rhetoric and a rule violation

**v0.1 claimed** (§1.1): "SMTP has no screen. The **reply codes are the UI**… Every reply is a *view*… This is the reframing that makes the rest of the model work." The `WIRE` row in §3 carries `250-SIZE`, `250 ok`, `354`, `250 queued as X`.

**The method says**: "Human roles get SCREEN nodes. System actors and processors get AUTOMATION nodes. Place both types during storyboarding — do not defer automations to a later step." (KIT `eventmodeling-storyboarding-events/SKILL.md:489`, criteria at 464–500). A column is an automation column when "The action is triggered by the system, not a user gesture" and "No human sees or interacts with the UI at this step." A sending MTA is a system actor.

The book states the purpose of a wireframe: "Screens or wireframes basically show how a UI for the system might look like and how a User can interact with the system" (BOOK `0065-planning-systems-using-event-modeling.txt`, p.50); their value is that "screens help foster understanding and ensure everyone knows exactly what is being discussed", resolving arguments "by drawing the screen mockups" (BOOK `0208-modeling-use-cases-with-wireframes.txt`). Rendering `250-SIZE 10485760` as a mockup buys none of that — it is the protocol trace you already have.

The kit further requires every SCREEN node to carry `meta.fields` with a `mapping` per field *and* a rendered sketch: "A screen node without a wireframe sketch is an empty placeholder. It must not be left unrendered." "The wire is the wireframe" commits v0.1 to sketching every SMTP reply. It will not do that, so those nodes sit permanently as empty placeholders.

Also violated: "Never place more than one SCREEN node in the same column, even across different actor lanes… always signals a design error." The `WIRE` row stacks `250-SIZE` / `250-STARTTLS` and `250 ok` / `550 no relay` in single columns.

**What survives**: the second half of the sentence. "Every reply is a *view*" is right — a reply is a read model rendered to a consumer, and read models legitimately feed background processes.

**Fix**: retitle the row `Actor (Sending MTA / Queue Runner)` and populate it with gears. Model replies as read models (`SessionCapabilities`, `TransactionReplyState`) consumed by the `ProtocolResponder` automation. Add the small number of genuine screens the system actually has — Postmaster queue inspector, deferred-mail report, admin dequeue — because the kit hard-gates on it: "If a role from the catalog has zero screens, either: The role is missing screens (add them), or The role doesn't belong in the catalog." v0.1 has zero human roles and zero screens, i.e. it silently skipped storyboarding entirely.

---

## 6. Error paths are crammed into the happy-path timeline, and there are no chapters

**v0.1 claimed** (§3 ASCII): `250 ok` and `550 no relay` occupy the same column; `250-SIZE` and `250-STARTTLS` occupy the same column. §8 asks whether submission and relay are "one model with a flag, or two".

**The method says**: "In Event Modeling, we focus on one use case at a time along a single timeline… Not all software follows a linear timeline - we have conditions and loops to consider. How do we model them? The short answer: we don't. Instead, pick one flow and model it." And: "Typically, we model the 'good case' first as a starting point, and all 'error cases' as separate flows or simple 'Given / When / Thens' if sufficient." (BOOK `0293-structuring-an-event-model.txt`, pp.272–274). Alternative flows get a marker sticky below the slice pointing at a separate named model.

Chapters are mandatory in Step 1: "Every event must be placed into its dedicated timeline — never into a generic or unnamed chapter" (KIT `eventmodeling-brainstorming-events/SKILL.md:133–279`); each chapter must "be understood as a standalone narrative on its own" and "be naturally owned by one team or one domain expert". Plotting may not invent new ones: "Timelines (chapters) are created and assigned during Step 1… This step does not create new chapters."

**Why it matters**: the branch-in-a-column habit is what makes SMTP models degenerate into one flat state machine — the exact failure §5.2 correctly diagnoses elsewhere.

**Fix**: chapters — Inbound Reception (:25) | Authenticated Submission (:587) | Outbound Delivery | Bounce/DSN | Local Delivery, with Session / Transaction / Data sub-chapters under Reception. Move `550 no relay` and `421` into their own flows. Note the corpus's counter-caution too: "If all events belong to a single flow, one timeline is correct — do not split artificially."

This also answers **open question 5** directly: two chapters on one board, not one model with a flag. "It is perfectly fine to have more than one model on a board. In fact, this is the rule rather than the exception for me. I prefer having many smaller models over one large model."

---

## 7. Stream design is parked in "open questions"; the corpus answers it

**v0.1 claimed** (§8.3): "One stream per session? Per transaction? Per message post-acceptance? The pivot event suggests a handoff between two streams."

**The method says**: stream design is a first-class modeling activity, and the validator requires per swimlane a "Clear name (identity)", "At least one event type", "Initial event (what creates the stream)", "State transitions documented", plus "Every event belongs to exactly one lane" (KIT `eventmodeling-validating-event-models/SKILL.md:44–77`; checklist 49–95). Guidance: "You group together what belongs together. Every stream has a unique identifier… the stream should mimic the lifecycle of the underlying business capability"; streams are "often something between 10 and 100" events; keep them short by ending them on a business event; stream roots are "NOT DDD aggregates—they're simply the logical roots of events." (BOOK `0137-event-streaming-event-sourcing-and-stream-design.txt`, pp.121–125).

**Fix**, derivable entirely from the above:

- `session-{connectionId}` — initial `ConnectionAccepted`, closed by `SessionClosed`.
- `transaction-{txnId}` — `MailTransactionStarted` → `MessageAccepted` | `TransactionAborted`. Multiple per session.
- `message-{queueId}` — **`MessageAccepted` is the initial event**. This is the handoff v0.1 is reaching for.
- `delivery-{queueId}-{recipient}` — `RelayAttempted…` → `MessageDelivered` | `DeliveryGivenUp`. One stream per recipient is what keeps partial delivery clean and keeps retry history out of the message stream.

Checklist 7.1 bites here: "each stream can be versioned/restored independently". The message stream must be restorable without the session stream, which means `MessageAccepted` must *carry*, not reference, everything delivery needs — retroactively justifying its apparently redundant `reverse_path` and `recipients[]`.

Also apply the validator's Phase 6 check, which is unusually apt for a protocol: "no impossible event sequences (state machine sound)". `DataPhaseEntered` before `RecipientAccepted` must be shown impossible via explicit preconditions — Phase 4 warns specifically against encoding such rules implicitly "in event stream structure".

---

## 8. The translation boundary is asserted in prose and never modeled; one sentence is backwards

**v0.1 claimed** (§4.3): remote MTA replies and DNS answers are external facts translated into our events. "Our timeline never contains a foreign system's events verbatim."

**The method says**: the Translation pattern exists for exactly this — "The external event represents data that comes from an external system. This can be an API call, an incoming Kafka record, or even a simple CSV file… We typically need to translate the data from the external event to a format suitable to our needs." (BOOK `0378-example-integration-with-apache-kafka-and-translations.txt`, p.355). But the foreign fact must *appear on the model*: "Whenever we receive data from an external source, we model it as an external event in the Event Model. These external events are explicitly modeled in yellow, indicating that external data is entering the system during this process step." (BOOK `0065-…`, pp.56–58, Figs 3.10–3.12).

v0.1's §4 diagram shows `RelayAccepted` / `RelayDeferred` appearing from nowhere. It has no External Event element anywhere.

The quoted sentence is wrong twice: (i) the yellow external event *is* on the timeline — what it is not is a stored internal domain event; (ii) the rule being reached for runs the other direction — it is about not letting foreigners read *your* internal events: "Accessing the internal events of a system is technically the same as directly accessing the database of a system… it creates massive coupling." (BOOK `0112-internal-versus-external-data.txt`).

**Fix**: add yellow external events to the left of the translating command — `RemoteReplyReceived` (the peer's 250/4xx/5xx), `MxRecordsReturned` / `DnsFailure`. Rewrite the sentence as: "External facts enter as yellow external events and are translated into our domain events; our internal domain events are never exposed outward — the outward contract is an integration event."

Pick a shape explicitly. The corpus permits two: "we can either represent the external events directly as a Read Model and do the translation under the cover, or we can model it basically as a State Change where the external event is explicitly translated and stored as a new internal event… I typically tend to do the latter." For remote MTA replies take the latter (the reply is a fact you must retain for the DSN). For DNS the former is arguable — but the validator forbids a projection doing I/O ("Deterministic Projections… bad: 'Projection uses: external API call during replay'"), so any read model needing MX data requires `RouteResolved` to be captured as a fact first. v0.1 already does this correctly.

**Missed boundary**: inbound. A sending MTA is a foreign system. v0.1 models its SMTP verbs directly as our commands, which is defensible — a client calling our API issues commands — but it must *say so*, because the kit has no `protocol:` or `wire:` field-mapping form (the closest is `webhook:<payloadField>`). Record it as a gap: "When you encounter a mapping that cannot resolve to a connected element, write the mapping as-is but flag it as a gap in the completeness notes. Do not invent connections that don't exist." (KIT `eventmodeling-identifying-inputs/SKILL.md:306–338`).

---

## 9. A slice is not a GWT, and the scenario set is effectively empty

**v0.1 claimed** (§5): "Given / When / Then slices — Each slice is one testable spec and one unit of work." Four scenarios total.

**The method says**: "Slices are either State Changes (where a user or an automated process modifies the system's state) or State Views (where the system's stored data is used to generate views or projections)." (BOOK `0065-…`, pp.61–63). GWTs hang *below* a slice, many per slice: "For some more complicated parts of the system, we could define up to 10 or more GWTs for a single State Change to describe all business rules that apply. Don't save on GWTs; they are a perfect opportunity to describe business rules in detail." (BOOK `0228-given-when-then-scenarios.txt`, pp.205–213). The kit is harder: "No command has only 2 scenarios unless all other types were reviewed and found inapplicable", against seven types — Happy Path, Validation Failure, State Violation, Duplicate Action, Alternative Path, External Failure, Compensation (KIT `eventmodeling-elaborating-scenarios/SKILL.md:199–213, 568–702`).

**Why it matters**: measured against that, v0.1 has 4 scenarios for ~8 commands and ~4 read models. `SubmitContent` — the most consequential command in SMTP — has exactly one (§5.3, the happy path). Missing for it alone: size exceeded (552), too many recipients, content-policy rejection (550), 4xx deferral at acceptance, timeout mid-DATA, dot-stuffing violation, duplicate/pipelined DATA, and connection dropped before the final `CRLF.CRLF` — the compensation case, RFC 5321's own famous ambiguity, unaddressed. The escape hatch is narrow: "If the answer is 'that situation cannot occur in this business' then no scenario is needed for that type — but that judgment must come from the domain, not from a desire to write fewer scenarios."

Zero read-model scenarios. "If you want to describe how a Read Model projects data to a view, you typically do not use GWTs but GTs (Given - Then). Read Models only rely on previously stored events, so there is no 'When' part necessary." Every read model needs GT scenarios with concrete example data: "The key is to provide clear, concrete examples."

**Credit**: §5.4 is written as GIVEN/THEN with no WHEN. That is correct form for an automation — "Test cases for processors are typically modeled using a 'GIVEN / THEN' approach, skipping the 'WHEN' part." (BOOK `0408-implementing-automations.txt`, p.405). Say so, so it doesn't read as an omission.

**"Unit of work" — defensible, with a source conflict.** The book supports it: "These slices may eventually evolve into stories for implementation"; "My goal is to keep each slice to about a day's worth of work, at most." The kit's slicing skill explicitly denies it: "Slicing itself does not require planning team allocation, sprint sizing, or effort estimates — that's a separate concern." Keep the book's framing and note the disagreement.

**Fix**: retitle §5 "Scenarios"; hang scenarios under named slices; drop "one testable spec" (a slice is many). Label §5.1's `⊘ no transaction event → 503` as an error scenario in its defined shape — `expectError: true`, non-empty `errorDescription`, and "`then` must be `[]` — no events are produced on rejection".

---

## 10. The completeness audit runs backward only

**v0.1 claimed** (§6): "Every event field traces to a command or a prior event. No orphans in either direction." The audit that follows runs only backward, from `Received:` header clauses to events.

**The backward half is well executed and canonical**: "Where does this data come from? Ask backwards again: What data must have been stored in the event(s) to populate the read model?"; "Commands generally have to provide all data necessary to persist an event." The book names the exact technique v0.1 used: "hide all streams and start from the right. Uncover one Event at a time. For every event, check that the preceding events deliver all information necessary to create the current event… Going from right to left forces you to look at it from a different angle and focus on the data." (BOOK `0208-…`, pp.191–193; `0065-…`, pp.59–60). The three inverted findings are exactly the payoff the method promises. Keep all of it.

**The forward half is missing.** Completeness is a matrix with two columns — `Origin:` / `Destinations:` / `Status:` — and the checklist requires both "Every field has clear origin" **and** "Every field has identified destinations" (KIT `eventmodeling-checking-completeness/SKILL.md:40–95`). A field with an origin and no destination is dead weight.

**Why it matters**: run that pass and v0.1 loses fields immediately — `ServiceGreetingSent.reply_code` (read by whom?), `ClientIdentified.extensions_offered`, `TlsNegotiated.peer_cert_subject`, `RelayAccepted.remote_reply`, `MessageContentReceived.line_count`, and `SessionReset` which carries nothing at all. Some will survive with "audit log" as their destination — legitimate, but it must be written down.

**Also missing from the same step**: "Every column with a COMMAND or READMODEL node has a slice defined", "Workflow step contracts defined for each step", "Processors are identified", "No circular dependencies", "Error paths documented". None are present.

**Run the audit on the DSN, not just `Received:`.** RFC 3464 fields are exactly what a backward trace is designed to catch, and several are currently unsourced: `Diagnostic-Code` needs the remote reply text (only `RelayFailed.reason`, unstructured); `Remote-MTA` has `RelayAttempted.host` (present, good); `Will-Retry-Until` needs the give-up deadline, which is on **no** event — only `DeliveryGivenUp`, after the fact. That is a genuine hole v0.1's own method would have found had it been pointed at bounces.

---

## 11. No workflow step contracts

**v0.1 claimed**: nothing — there is no equivalent artifact.

**The method says**: "Each workflow step is a contract between the previous step and the next. Document preconditions and postconditions." Format: Preconditions, Postconditions (including the exact event and its fields), and an explicit Contract sentence — "Contract: Any system can assume if these postconditions are true, the order has been properly created through this step." Checklist: contracts for each step, explicit pre/postconditions, documented dependencies, "Teams can work in parallel based on contracts" (KIT `eventmodeling-checking-completeness/SKILL.md:230–293, 486–490`).

**Why it matters**: this is the artifact that makes §2's insight *enforceable*, and a far better home for it than a coined noun.

**Fix**, for the step that matters most:

> **Step: SubmitContent**
> **Preconditions:** `MailTransactionStarted` exists; ≥1 `RecipientAccepted` exists; DATA acknowledged; `declared_size` ≤ limit.
> **Postconditions:** `MessageContentReceived{content_ref, actual_octets}` and `MessageAccepted{queue_id, reverse_path, recipients[]}` exist.
> **Contract:** once `MessageAccepted` exists, responsibility has transferred (RFC 5321 §2.1); the delivery side may proceed without re-reading any reception event, and no path may discard the message silently — every recipient must terminate in `MessageDelivered`, `RelayAccepted`, or a DSN.

---

## 12. No metadata

**v0.1 claimed**: nothing. Payload fields only.

**The method says**: "In Event Sourcing, the unit of persistence is the event itself. We attach metadata to each event to give it context." Canonical list: Event ID, Event version, Correlation ID, Causation ID, User ID, Session ID. "It's important to distinguish between payload and metadata. The event payload holds the business-relevant information, while the metadata contains context-specific details." And: "One powerful use of metadata is the inclusion of correlation and causation IDs, which enable system traceability by creating a clear data trail for each event… The causation ID identifies the current step of a process… As the process continues, the correlation ID is propagated across all subsequent steps, linking them together." (BOOK `0547-handling-metadata.txt`, pp.523–527).

**Why it matters**: this is not bookkeeping for an MTA — it is the model's answer to requirements v0.1 already discovered. The `Received:` trace, loop detection, and DSN diagnostics are all causation chains. Several fields v0.1 put in payloads belong in metadata: `occurred_at` on every event, the session id threading transaction events back to their connection, and the queue id threading reception → delivery → bounce.

**Fix**: `correlation_id = queue_id` (it survives the reception→delivery handoff); `causation_id` = the SMTP command or prior event; `occurred_at` moves to metadata across the board.

---

## 13. Event catalogue defects, and the open questions that the corpus already answers

**v0.1 claimed** (§3.1, §3.2, §4.1, §8): a 26-event catalogue plus six unresolved questions.

All names are past tense, satisfying the only explicit naming rule ("Events: Verb past tense"). Specific failures:

**Fails "events are facts, not calculations"** — `DeliveryGivenUp.attempts` (a count) and `RelayAttempted.attempt_no` (a running counter). "No aggregation events: Totals, averages, counts are read models, NOT events." Drop both; the counts are projections of `RelayAttempted`, and `DeliveryQueue` already tracks `attempts`. `DeliveryGivenUp.first_attempt_at` is borderline — defensible only as a frozen contract field for the bounce; call that out rather than leaving it implicit.

**Fails "is this a business moment?"** — `ServiceGreetingSent` and `DataPhaseEntered`. "No CRUD events (UserUpdated, RecordDeleted) — events describe business moments, not database operations" (KIT `eventmodeling-brainstorming-events/SKILL.md:573–588`). `ServiceGreetingSent` records that we emitted a banner; its only consumer is the `Received:` `BY` clause, better sourced from configuration or `ConnectionAccepted`. **Open question 4** is thereby answered: `DataPhaseEntered` is derivable from the presence of `MessageContentReceived` and adds no field — drop it unless the destinations audit finds a real consumer.

**Overlapping semantics** — `SessionReset` (no fields) overlaps `TransactionAborted(cause=rset)`. The validator checks "Unique semantics (no duplicates)". Keep one. Separately, `MessageDeferred` (4yz at acceptance) and `RelayDeferred` (4yz at relay) are different facts on different streams sharing a word; and `RelayAccepted` vs `MessageDelivered` need a stated distinction (peer accepted responsibility vs we wrote a local mailbox) or they will be conflated in code.

**Open question 1 — are rejections events?** The corpus answers it; stop hedging. Scenario rules: with `expectError: true`, "`then` must be `[]` — no events are produced on rejection". But the same corpus requires "Every command produces events OR documents rejection" and lists "Missing event: 'No event for failure' → Fix: Add failure event". The resolution the corpus supports: a rejection that is *input validation* (syntax, bad sequence → 500/501/503) is an error scenario with no event; a rejection that is a *business decision with downstream consequences* (relay denied, unknown recipient, over quota — feeding reputation, abuse detection, the DSN) is a fact and gets an event. So keep `RecipientRejected` and `MessageRejected`; invent nothing for syntax and sequencing errors. §5.1 is already right to emit none.

**Open question 2 — is `ReplyIssued` an event?** No. A reply is a view — a read model rendered to the peer. Making it an event records a projection as a fact. v0.1's own volume argument is the right one.

**Open question 6 — pipelining.** Columns are narrative steps, not round-trips: "Don't get stuck on exact timing — the point is the logical flow." Pipelining is a transport optimisation; the model does not change.

---

## 14. The automation is right but missing its back-channel — and the race condition it hides is duplicate mail

**v0.1 claimed** (§4): `DeliveryQueue` read model as todo-list, drained by a processor issuing `ResolveRoute` / `AttemptRelay`.

**Verdict: correct**, and the strongest part of the draft. It matches the Processor-Todo-List pattern almost exactly: "Just as users create todo-lists for their tasks… we can imagine our processor also maintaining its own technical todo-list of tasks that need to be done next." (BOOK `0514-pattern-processor-todo-list-pattern.txt`, pp.491–500).

**Two mandated pieces are missing.**

*(1) The back-channel.* "The 'Todo Expired' event feeds back into the Read Model and checks it off the Processor-Todo-List to not expire it twice. For this 'Back-Channel' I typically use a dotted line to indicate that this is not part of the 'Flow' but just updating the data of the Read Model." §4's ASCII has no back-channel. Ask the canonical question per todo — "How does a Task get checked off the Processor-Todo-List?" For FnEmail: `RelayAccepted`, `MessageDelivered`, `RelayFailed` or `DeliveryGivenUp` for that `(queue_id, forward_path)`.

*(2) The race condition*, which for SMTP is a duplicate-mail bug, not a cosmetic one: "There is a race condition in the implementation. The update of the Read Model after an item is expired is eventually consistent… which could lead to items being expired twice. If this is not feasible, the back channel should be designed to be immediately consistent by running it in the same thread." Duplicate delivery is the classic MTA failure and the corpus hands you the analysis; v0.1 ignores it.

The pattern's failure semantics also match SMTP 4xx precisely: "If the refund does not work, the task will not get closed and will be retried on the next processor schedule until something happens that closes this task (maybe moving it to a Dead-Letter-Queue and processing it manually)." `DeliveryGivenUp` is the DLQ move.

---

## 15. `next_attempt_at` on `RelayDeferred` — correct, and the two sources disagree about why

**v0.1 claimed** (§4.2): `RelayDeferred` carries a calculated `next_attempt_at`; the todo-list filters on it; "Retry needs no scheduler."

**The book endorses this exact construction for this exact use case**: "To be able to expire items after 24 hours, we either need the creation-date or the expiry-date in the system. We calculate the expiry-date at the time when we create the todo and store it directly in the Event." The read model then filters (`WHERE expiration_date < NOW() AND expired = false`) and the processor picks up what is due (BOOK `0514-…`, pp.492–498).

**The kit appears to forbid it**: "do not include computed or derived values ('those belong in read models')"; "No Computed Fields in Events: OrderCreated includes 'totalTax' (computed) → Includes items + amounts, tax computed in projection."

**Reconciliation, in the kit's own words**: the distinction is computed-once-then-frozen versus recalculated. "Recalculated state identified: If a value changes multiple times, it's a read model"; "Calculations change multiple times as source data changes. Events are immutable." `next_attempt_at` never changes after `RelayDeferred` is written — the next deferral writes a new event with a new value. It is a decision taken, i.e. a fact. The kit's own completeness example even lists `total` on `OrderCreated` with "Origin: Calculated from items[] and unit prices — Status: Complete". The operative test is not "is it computed?" but "can it be recomputed later and change?"

The validator's Final Question 2 is the sharpest test here: "Could you change your core algorithm/calculation without changing event history?" With `next_attempt_at` frozen, changing the backoff curve does not rewrite history — old messages keep their scheduled times. That is a point *in favour* of v0.1's design. With `attempt_no` / `attempts` baked in, replay gets brittle (see §13).

**Fix**: keep `next_attempt_at` and add one line of justification so a reviewer does not delete it — it is the frozen output of the backoff decision at attempt time, not a recalculated projection.

**Overstatement to correct**: "Retry needs no scheduler." The give-up window as a predicate on a projection is canonical, but the processor still needs a periodic trigger — the book uses `@Scheduled(fixedDelay=1000)` or "a cron-Job running periodically or a job scheduled by a workflow engine". No per-message timer state, yes. No scheduler at all, no.

---

## 16. `content_ref` is right and has a canonical name: forgettable payload

**v0.1 claimed** (§7): message bodies stay out of the event log behind a `content_ref`, "otherwise the timeline becomes a mail spool and replay becomes impossible."

**Verdict: correct.** "What if we strictly separate personal and non-personal data from the outset? What if we store personal information in database tables, for instance, and reference it by an ID in our events? … rather than directly storing personal information in the Event Payload, we store a reference using the event ID and the personal data identifier." (BOOK `0575-handling-sensitive-data-with-gdpr.txt`, pp.560–563). That is `content_ref` exactly.

The size/replay justification is true and consistent with the stream-design chapter's insistence on short streams — but the corpus's framing is GDPR, which for an email system is the stronger and more urgent argument: message bodies are personal data by definition, and a mail store must honour erasure. Cite it that way.

**Two consequences v0.1 misses**:

1. The alternative, **crypto shredding** ("encrypt all [personal data] and throw away the key"), deserves an explicit rejection with a reason — it may be preferable for a mail spool where the blob store is the same store.
2. Deletion does not propagate: "Deleting an encryption key or clearing data in an external data store will not automatically update these projections… replays are necessary for projections, and this process is exactly what's required here." FnEmail needs a documented replay path for every projection that touched message content — search indexes, spam scoring, mailbox views. §7 lists "Content Store" as a boundary and says nothing about erasure or replay.

---

## 17. Command names are protocol tokens

**v0.1 claimed** (§3): commands `Ehlo`, `MailFrom`, `RcptTo`, `BeginData`, `SubmitContent`, `Reset`.

**The method says**: "Commands: Verb present (CreateOrder, ConfirmPayment); Events: Verb past tense (OrderCreated, PaymentConfirmed)." (KIT `eventmodeling-validating-event-models/SKILL.md:70–77`). The goal is a ubiquitous language: "A language which does not contain technical jargon and only business terms." (BOOK `0065-…`, p.44).

**Why it matters**: `Ehlo` and `RcptTo` are wire tokens, not verb phrases. They leak the transport encoding into the domain language, which is the opposite of the intent.

**Fix**: `IdentifyClient`, `NegotiateTls`, `StartMailTransaction`, `AddRecipient`, `EnterDataPhase`, `SubmitContent` (already fine), `ResetTransaction`, `CloseSession`; `AttemptRelay` and `ResolveRoute` are already fine. Keep the SMTP verb as a documented alias on each slice — that alias *is* the wire→domain translation, and naming it makes the inbound translation boundary visible (see §8 above).

**Zombie commands**: NOOP, VRFY, EXPN, HELP produce no events. "No Zombie Commands: Commands that never produce events (read-only OK)" — permitted, but they must be explicitly documented as read-only, since "Every command produces events OR documents rejection." v0.1 omits them entirely.

---

## 18. "Swimlanes" — four lanes is wrong, and the word means three different things across the sources

**v0.1 claimed** (§1): four horizontal lanes — Wireframe, Command, Read model, Event — with time left to right.

**Correct**: the timeline. "Event Modeling works on a single timeline, and we model the steps a system provides from left to right. You can read 'what happens in this system' like a story."

**Error 1 — there are five elements, not four.** Canonical: Event (orange), Command (blue), Read Model (green), Screen/wireframe (white), **Processor/gear (black)**, plus External Event (yellow) for translations. v0.1's lane table omits the Processor entirely; automation appears only as prose in §4 and an unlabeled arrow in the ASCII. It has no External Event element at all — which is fatal to its own §4.3 claim.

**Error 2 — "swimlane" is used incorrectly, and the sources each mean something different by it.** The book: "Swimlanes define stream boundaries. Typically, all events in one swimlane end up in a physical stream." (BOOK `0137-…`, verified at line 205). The agent kit: `swimlane` is the board **row type** that EVENT nodes live in — the only row types defined are `actor`, `interaction`, `swimlane`. So the canonical board has three row types: SCREEN and AUTOMATION share the actor row; COMMAND and READMODEL share the interaction row; EVENTs live in swimlane rows, one per stream. Not four equal lanes. (A third usage — swimlane as an ownership lane for Teams or Systems in the Conway's Law step — exists in the kit's cheat sheet; it is not the book's meaning either.)

**Fix**: call the four rows "lanes" or "element rows". Reserve "swimlane" for stream boundaries inside the Event row — Session / Transaction / Message / Delivery — which is the meaning that actually does work for §8's open question 3. Add a Processor row and a yellow External Event element.

**Error 3 (small but load-bearing)**: the Read model lane is glossed "feed UI and command decisions". The book says read models feed "screens, but also… background processes". Feeding *command decisions* is a different thing, and it is where the model breaks — see correction 1.

---

## 19. "Pivot event" is not a term in the corpus

**v0.1 claimed** (§2): `MessageAccepted` is "the pivot event" — negotiation to its left, obligation to its right.

**The insight is right. The label must go.** `grep -i pivot` across all 57 book chapters and `FULL.txt` returns **zero hits** (verified). It appears in no skill file. Coining vocabulary is what the method exists to prevent — "Miscommunication, misunderstandings and wrong assumptions make projects fail" — and a reader who searches eventmodeling.org for "pivot event" finds nothing.

Three canonical concepts each carry part of the meaning, at no cost:

- **Stream boundary / "Closing the Books"** — "A stream can end, for example, at the end of the day (like in trading by closing the books), after each month (payroll management), or after a certain event has happened (order was submitted)." `MessageAccepted` ends the transaction stream and opens the message stream.
- **Integration event** — "It's much better to provide a 'Cart Submitted' event that already contains the pre-calculated data necessary for an external system to process it. From the perspective of the cart-system, this is an integration event and serves as a contract with external systems." `MessageAccepted` must carry everything delivery needs, so delivery never re-reads reception's internal events. This is the real design force behind the fan-out, and it justifies the otherwise-redundant `reverse_path` and `recipients[]`.
- **Chapter boundary** — "I tend to logically group slices together… a chapter defines kind of a context for a given slice." Reception and Delivery are two chapters.

**Fix**: delete "pivot event". Write: "`MessageAccepted` is the stream boundary (Closing the Books) between the Reception chapter and the Delivery chapter, and the integration event carrying the contract between them." Every claim v0.1 makes about it survives verbatim; only the coined term goes.

**Related**: "The model is split into two lanes at this event" is a stream-boundary statement dressed as a layout statement, and it contradicts v0.1's own four-lane table. Streams subdivide the Event row; they are not a second set of columns.

---

## 20. Method framing: steps were skipped silently, and the step count should not be cited

**v0.1 claimed** (line 5): "An Event Modeling blueprint (Adam Dymitruk's method)", followed by four patterns and a finished model.

**What was actually done**: Brainstorming, Plotting, Inputs, Outputs, and a fragment of Conway's Law. Skipped: Storyboarding (no roles, no screens), Conway's Law in substance, and Completeness / Validate / Slice entirely. Say so rather than presenting a partial pass as a complete one.

**Do not cite a step count.** The package contradicts itself: the workshop reference says "across all 7 steps"; the orchestrator says "Coordinates the 10-step Event Modeling workflow" and "All 10 modeling steps completed"; and a nearby section of the same orchestrator file says "across all 9 steps". The book never enumerates numbered steps at all — it teaches five elements, four patterns, GWTs, and slices.

**Fix**: say "five elements, four patterns" (every source agrees) and describe the process narratively, rather than claiming conformance to a numbered pipeline whose own package cannot count itself.

**Attribution**: the book flags several of its own tools as unofficial extensions — chapters and sub-chapters are "not part of the original Event Modeling definition", and yellow external events are "not an 'official' notation in Event Modeling". A model that adopts them should say it is adopting Dilger's extensions, not "Dymitruk's method" unqualified.

---

## What v0.1 got right

Credit where it is due; none of the following should change.

- **The framing sentence.** "This document is the design surface. Code should follow it; where code and model disagree, one of them is wrong and we fix it here first." That is the corpus's own posture, nearly word for word: "Whenever you change something in the system, you always start with the model, adjust the model, and then you go to code." (BOOK `0065-…`, p.65). Best line in the document.
- **Time left to right on a single timeline.** Correct and correctly stated.
- **The automation shape** (§4). `DeliveryQueue` as todo-list drained by a processor is the canonical Processor-Todo-List pattern, arrived at independently. It needs a back-channel and a race-condition note, not a redesign.
- **`next_attempt_at` frozen on `RelayDeferred`** (§4.2). Explicitly endorsed by the book for the identical use case, and defensible against the kit's apparently contrary rule.
- **The give-up window as a predicate on a projection** rather than timer state scattered through the codebase. Correct; only "no scheduler" overstates it.
- **`MessageAccepted` as the responsibility-transfer point**, and the observation that a large share of SMTP bugs are failures to respect it. Right; it just needs the corpus's vocabulary and a workflow step contract instead of a coined noun.
- **`MessageAccepted` carrying `reverse_path` and `recipients[]` redundantly.** Looks like duplication; is actually the integration-event contract, and required by the stream-independence checklist.
- **Message bodies behind a `content_ref`** (§7). Correct, and canonically named "forgettable payload".
- **The inverted information audit** (§6). The three findings — dropping `peer_address` makes a conformant `Received:` impossible; `declared_size` and `actual_octets` are different facts; loop detection requires stored trace headers — are exactly the payoff the backward-trace technique promises, and the book names that technique explicitly. This is the second-strongest section in the draft.
- **Replies modeled as views rather than events** (§8.2's current position). Correct, for the reason given.
- **§5.4's GIVEN/THEN with no WHEN.** Correct form for an automation scenario, per the book.
- **§5.2's diagnosis** that implementations modeling SMTP as one flat state machine wrongly demand a second `EHLO` after `RSET`. Right, and it is the argument for per-command minimal state rather than the shared projection the same document then draws.
- **`RouteResolved` captured as a fact** before any read model consumes MX data. This satisfies the validator's deterministic-projection rule, which forbids I/O during replay. Deliberate or not, it is correct.