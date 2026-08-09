# Step 1 Role Catalog — RFC 5321, rewritten actor-generically

Sources are kept apart throughout, because blurring them is the failure this rewrite corrects:

- **[RFC]** — RFC 5321 (and 7504), cited by section with the heading's line number in
  `docs/rfc/rfc5321.txt`.
- **[KIT]** — the Event Modeling skill kit, cited by file and line in the private research repo.
  Cited and quoted briefly, never reproduced (rule 7).
- **[THIS PROJECT]** — already settled here, cited to `docs/event-model.md`,
  `docs/model-altitude.md` or `docs/DECISIONS.md`.
- **[INFERENCE]** — mine, in this pass, resting on the three above. Not settled.

---

## ⚠️ Correction — the humans-are-scarce inference was wrong

**What was claimed.** The catalog as produced carried an editorial note to the effect that
*RFC 5321 names almost no humans … that predicts where Step 3 storyboarding will find no screen to
draw*, and treated the five-or-six human roles it had found as the whole catalog, with the
scarcity itself read as a deficiency.

**Why it was wrong — four ways.**

1. **It measured the wrong thing.** A lane is a partition of the actor pool, not a census of
   humans. A processor, an automation, a bot or an external system occupies an actor lane on
   exactly the same terms a person does. The count of humans therefore predicts nothing at all
   about how many lanes get drawn.
2. **This project had already ruled it, and the note contradicted the ruling.**
   `docs/event-model.md` states that human-versus-machine *"was never the distinguishing axis.
   Working a todo list is"*, and gives the SMTP-specific reason: the server cannot tell a human
   from a script, since `HELO` bar.com is identical whether it came from Postfix or from someone
   typing at a terminal. An element type that flips on a property the system cannot observe is
   not well formed. [THIS PROJECT]
3. **The prediction is false on this board.** The Operator lane in `docs/event-model.md` is a
   human role and it does have a screen — a ⬜ view screen showing the session transcript. Step 3
   finds a screen to draw for the one human who is actually in the timeline.
4. **It inverted the census.** RFC 5321 is written about hosts and processes. The automated
   actors are the majority and they are first-class; reading them as a residual left the catalog
   at six entries where the RFC names more than twenty roles. The corrected catalog below puts
   them first, in the RFC's own proportion.

**What replaces it.** The absence of humans in the modeled timeline means the lanes there are
automations and remote systems — not that there are no lanes. And the reason the remaining humans
draw no screen is **scope, not species**: they act *on* the model (configuration, a required
address, a design-time decision) rather than *in* its timeline. Two different facts that the
original note fused into one wrong one.

*(A smaller error carried in the same note: it announced five human roles and then listed six.
All six survive below.)*

---

## The catalog

Form is the kit's own — Name, Description, Key actions, and the permissions boundary phrased as
its question, *What can this role NOT do?* [KIT `eventmodeling-brainstorming-events/SKILL.md`
lines 372-420]. The **Cannot** entry is the one that gets dropped, so it is filled for every role
here, and where the RFC supplies a MUST NOT or a SHOULD NOT that is what fills it.

Real values throughout (rule 8): a client at 192.0.2.10 announcing bar.com, a server at
203.0.113.20 serving foo.com, reverse path \<Smith@bar.com>, forward path \<Jones@foo.com>.

### A. Protocol-position roles — the two the RFC defines before any machine

The RFC's first move in its terminology section is to rename the two parties from hosts to
positions, and to say outright that one host can hold both: a given host may act as both server
and client in a relay situation (§2.3.2, line 645). **Position, not machine** — which is the
owner's point stated by the specification itself, eight sections in.

**1. SMTP client** — the party that issues commands and consumes replies on an open channel.
Occupied by an originating system, a relay, a gateway, a submission client, or a person at a
terminal; the role is identical in all five cases.

- *Key actions*: open the two-way channel; read the `220` greeting; issue `EHLO` (falling back to
  `HELO`) carrying its own FQDN or, having no meaningful name, an address literal (§4.1.1.1,
  line 1783); issue `MAIL`, `RCPT`, `DATA`, `RSET`, `VRFY`, `NOOP`, `QUIT`; transmit message data
  after `354`; transfer the message to one or more servers **or report its failure to do so**
  (§2.1, line 348).
- *Cannot*: send message data unless a `354` has been received (§3.3); issue `MAIL` before `HELO`
  or `EHLO` (§4.1.1.1); generate invalid source routes, or depend on serial resolution of names
  (§3.6.1, line 1421); cache a `5yz` response to `MAIL` (§4.5.4.1, line 3685); transmit a bare CR
  or LF except as a `<CRLF>` terminator (§2.3.8); issue replies — the reply direction is the
  server's alone.

**2. SMTP server** — the party that answers. Also identified by name in its greeting and in its
`EHLO` response (§4.1.1.1).

- *Key actions*: emit `220` on connection, or `554` to refuse the session outright while still
  waiting for `QUIT` (§3.1, line 959); accept or reject each command; on `RSET`, discard stored
  sender, recipients and mail data and clear all buffers and state tables (§4.1.1.5, line 2079);
  **accept responsibility for the message when it issues the `250` that follows the end-of-data
  indicator** (§2.1, §4.2.5 line 2941); insert the trace header at the head of the content
  (§4.4, line 3156); accept mail for postmaster (§4.5.1, line 3378).
- *Cannot*: close the connection under normal circumstances except after `QUIT` with `221`, after
  a `421` service shutdown, or on timeout (§3.8, line 1640); close the connection because a
  command was not understood — that is a specification violation, `500` and wait is the required
  behavior (§3.8); close on `RSET` (§4.1.1.5); return `250` to `VRFY` on syntax alone (§3.5.3,
  line 1371); list an extension in `EHLO` for which it will answer `502` (§4.2.4, line 2931);
  lose an accepted message for frivolous reasons (§6.1, line 3960); initiate — it never sends a
  command.

### B. System-position roles — §2.3.10 (line 801), the four kinds by role played

The RFC distinguishes four SMTP systems *based on the role those systems play*, which is a
partition of one pool by function, not four kinds of hardware.

**3. Originating system** — introduces mail into the transport service environment.

- *Key actions*: construct the envelope from the submitted message when the user agent did not
  supply one — recipients from To, Cc and Bcc into `RCPT` commands, the return address from the
  system's identity for the submitting user (Appendix B, line 4720); MAY complete an incomplete
  message: add a message-id, add a date or time zone, correct addresses to FQDN form (§6.4,
  line 4074).
- *Cannot*: send a message that already carries a Return-path header field (§4.4); make those
  §6.4 completions while acting as an intermediate relay — the same host in the relay role is
  forbidden them; gateway a foreign message on header fields alone (Appendix B).

**4. Delivery system** — receives from the transport environment and passes the message to a user
agent, or deposits it in a message store.

- *Key actions*: make final delivery, inserting the return-path line at the head of the mail data
  — required, and mail systems MUST support it (§4.4); insert the message into the destination
  mailbox per local conventions (§4.1.1.3, line 1916); MAY remove existing Return-path fields
  before adding its own (§4.4).
- *Cannot*: alter or delete a Received line already present (§4.4); reject mail on the format of
  a trace header field (§3.7.2, line 1575); leave more than one return path in a delivered
  message (§4.4).

**5. Relay system** — receives from an SMTP client and transmits onward to another SMTP server
without modification to the message data other than adding trace information.

- *Key actions*: accept or decline the relaying task exactly as it would for a local user, and
  on acceptance **become an SMTP client** and open a channel to the next server (§3.6.2,
  line 1446); strip or ignore source routes (§4.1.1.3); prepend its own Received line (§4.4);
  optionally count Received lines to detect a loop (§6.3, line 4065).
- *Cannot*: inspect or act upon the header section or body at all, beyond adding Received and
  attempting loop detection (§3.6.3, line 1475; §6.4); apply the originating system's
  message-completion fixes — MUST NOT, explicitly (§6.4); infer an FQDN from a partial name or
  local alias — MUST NOT for relays, where submission servers have some latitude (§5.1,
  line 3824); copy source-route names into the reverse-path (§4.1.1.3).

**6. Gateway system** — sits at the boundary between two transport environments and translates.
The RFC counts an address-rewriting firewall as one.

- *Key actions*: transform the message as the difference between environments requires —
  transformations a relay is **not** permitted (§2.3.10); rewrite header fields where necessary
  (§3.7.1); prepend a Received line and indicate environment and protocol in its via clause
  (§3.7.2); set the envelope return path from the foreign environment's error-return address,
  defaulting to the originator's address as the last resort (§3.7.5).
- *Cannot*: alter in any way a Received line already in the header section — MUST NOT
  (§3.7.2); emit addresses that are not fully-qualified or not usable for replies (§3.7.4,
  line 1604); build an SMTP envelope from header To and Cc alone when gatewaying inward —
  the documented cause of mail loops (Appendix B).

### C. Agent and store roles — §2.3.3 (line 656)

The RFC introduces these and then warns the reader off them in the same breath: the implied
boundaries often do not match conforming practice, so be cautious about inferring strong
relationships. Cataloged with that caution attached.

**7. Mail Transfer Agent (MTA)** — what an SMTP client or server *is*, seen from the mail-system
vocabulary rather than the protocol's: they provide the transport service and therefore act as
MTAs (§2.3.3). Not a fifth party. Key actions and boundary are the union of roles 1 and 2.

- *Cannot*: guarantee the MUA boundary — the RFC declines to fix it.

**8. Mail User Agent (MUA / UA)** — normally thought of as the sources and targets of mail. At the
source it collects mail from a user and hands it to an MTA; at the destination the delivery MTA
hands off to it, or transfers responsibility by depositing in a message store.

- *Key actions*: hand a message to the submission-client MTA, and — recommended — hand it an
  **envelope separate from the message itself** (Appendix B).
- *Cannot*: rely on the MUA–MTA protocol being standardized; the RFC states it is a private
  matter outside any Internet standard (Appendix B). A mailing list that modifies a message
  extensively stops being a list and becomes one of these (§3.9.2, line 1718).

**9. Message store** — the depository a delivery MTA deposits into, which a user agent is expected
to access subsequently (§2.3.3). §2.3.11 (line 826) makes the mailbox that depository and the
address a reference to it.

- *Key actions*: hold the delivered message; be the thing responsibility transfers to.
- *Cannot*: have its local-part interpreted by anyone but the host named in the domain part of the
  address — MUST, and the reason intermediate hosts must not optimize by rewriting (§2.3.11).

**10. Message submission server / submission-client MTA** — the intermediate server a
limited-capability client sends everything to (§3.6.3, line 1475; §2.1). The RFC names the role
and then pushes it out of scope to RFC 4409.

- *Key actions*: accept everything from clients that cannot queue or retry, then distribute; make
  the FQDN inferences a relay may not, having *somewhat more flexibility* (§5.1).
- *Cannot*: be specified here — the arrangements are private and fall outside this specification
  (§3.6.3). 🟥 **Hotspot.** This role's rules live in a document outside our normative set
  (`docs/event-model.md` fixes that set at RFC 5321 + RFC 7504), so any slice that needs them is
  reaching outside the model's own authority.

### D. Processing roles inside a system — the automations, named individually

**11. Alias resolver** — expands a pseudo-mailbox by substitution (§3.9.1, line 1710).

- *Key actions*: replace the pseudo-mailbox in the envelope with each expanded address in turn,
  then deliver or forward to each.
- *Cannot*: change the reverse-path — that single change is what makes the operation a list
  instead of an alias (§3.9.2); alter the rest of the envelope or the message body (§3.9.1);
  apply heuristics to drop addresses from the expansion, e.g. the originator's — strongly
  discouraged (§3.9, line 1687).

**12. List expander** — expands by redistribution (§3.9.2).

- *Key actions*: replace the pseudo-mailbox with each expanded address; **change the
  backward-pointing address** so final-delivery errors return to the list administrator rather
  than the originator (§3.9.2) — the return address MUST become that of the person or other
  entity who administers the list (§3.9).
- *Cannot*: touch the message header section — MUST be left unchanged, the From field expressly
  unaffected (§3.9); stay a list while making extensive modifications, at which point it must be
  viewed as a full MUA (§3.9.2).

*(Note the RFC's own wording at §3.9: a person **or other entity**. The specification declines to
require a human at precisely the place a role catalog would most expect one — the owner's
correction, in the source's words.)*

**13. Address resolver (MX lookup)** — §5.1, line 3824.

- *Key actions*: perform the DNS lookup — MUST; try MX first, reprocess a CNAME result as the
  initial name, treat an empty MX list as an implicit MX at preference 0 pointing at the host;
  sort by preference, lowest first; **randomize equal-preference destinations** to spread load —
  MUST; try each address in the returned order, at least two of them; queue and retry on a
  temporary DNS error, report a non-existent domain as an error.
- *Cannot*: use address RRs for a name that has MX RRs unless reached through them — MUST NOT
  (§5.1); resolve past a data field that yields a CNAME, which is outside the standard's scope;
  make FQDN inferences on behalf of a relay (§5.1). When the relay finds itself in the MX list,
  it MUST discard its own preference level and every higher-numbered one — a resolver that keeps
  them builds a loop.

**14. Retry scheduler / queue runner** — §4.5.4.1, line 3685. The RFC's general model for an SMTP
client is *one or more processes that periodically attempt to transmit outgoing mail* — a
processor working a todo list, in the method's terms, and the clearest automation in the whole
specification.

- *Key actions*: queue what cannot go immediately and retry it; delay after a failure — MUST;
  interval at least 30 minutes, give-up generally at least 4 to 5 days; keep a list of unreachable
  hosts with their timeouts; two attempts in the first hour then back off; shorten the queuing
  delay opportunistically when mail arrives from a host that has queued traffic; batch multiple
  recipients at one destination behind a single `DATA`.
- *Cannot*: retry without delay (§4.5.4.1); operate with hard-coded parameters — they MUST be
  configurable, which is what puts the Site Operator upstream of this automation (§4.5.4.1); send
  an error message in response to an error message, under any circumstances — MUST NOT (§4.5.4,
  line 3671); cache a `5yz` reply to `MAIL` (§4.5.4.1); retry to the same server after a `5yz`
  following end-of-data without user review and intervention (§4.2.5).

**15. Bounce generator / notifier** — §6.1, line 3960.

- *Key actions*: on a delivery failure after acceptance, formulate and mail the notification —
  MUST; send it with a null reverse-path `MAIL FROM:<>` — MUST; address it to the envelope return
  path, stripping an explicit source route down to its final hop — MUST; originate it from the
  relay or the host that first determines delivery cannot be accomplished (§3.6.3).
- *Cannot*: send a notification at all when the return path is null — MUST NOT (§6.1); send
  notifications about problems transporting notifications — MUST NOT (§3.6.3); bounce content
  rejected as hostile, where the default is to stay silent (§6.2, line 4013); reply to, or add a
  non-null reverse path to, a null-reverse-path message when forwarding (§4.5.5, line 3787).

### E. Human roles — real, cited, and mostly acting from outside the session

Six roles, kept in full. What changes is the account of *where* they act.

**16. Postmaster** — the reserved local name every relaying or delivering system MUST support,
case-insensitively, at every domain it serves, including the bare `RCPT TO:<Postmaster>` form with
no domain (§4.5.1, line 3378; syntax at §4.1.1.3). §3.1 says a contact point announced in the
greeting is no substitute for maintaining it.

- *Key actions in the protocol*: **none.** The obligation is on the server, not on the person.
  Outside it: receive the mail so directed, including failed notifications forwarded via the
  postmaster alias (§4.5.5).
- *Cannot*: be dispensed with, unless the server always answers `554` on connection opening
  (§4.5.1); be blocked other than narrowly, to contain an attack (§4.5.1).

**17. Mailbox Owner** — the user an address identifies, or the location mail is deposited into
(§2.3.11).

- *Key actions*: none in SMTP. The mailbox is written to, never by this role.
- *Cannot*: assign meaning to their own local-part outside their own host — semantics belong only
  to the host named in the domain part (§2.3.11); expect case-sensitivity to be exploitable, since
  doing so impedes interoperability (§2.4); rely on a blind-copy recipient list staying private if
  a FOR clause is supplied carelessly (§7.6, line 4319).

**18. Originating User / submitter** — the sending user whose host introduces the message (§2.1;
Appendix B).

- *Key actions*: compose and submit; interpret a returned transient failure as a non-delivery
  indication (§4.2.5); where the system permits it, override the envelope return address.
- *Cannot*: override that return address unless the system provides the way, and it may restrict
  the ability to privileged users (Appendix B); be sure of hearing about a transient failure at
  all — if the client handles the condition successfully, no such reply reaches the user (§4.2.5);
  count on their From field being the envelope return path, which it need not be (§4.4).

**19. Site Operator** — the site providing the server, which the RFC repeatedly grants discretion
(§7.9, line 4355; §7.3, line 4237; §3.5.2, line 1335; §3.1).

- *Key actions*: refuse mail for any operational or technical reason that makes sense to the site
  (§7.9); limit relaying to identifiable sources, answering `550` when policy rejects (§7.9);
  disable `VRFY` or `EXPN`, or restrict them to authenticated requestors (§7.3); suppress the
  software and version announcement (§3.1); **set the retry algorithm's parameters, which MUST be
  configurable** (§4.5.4.1).
- *Cannot*: make the server appear to have verified an address it did not — a disabled `VRFY`
  MUST answer `252`, never a code confusable with a verification result (§7.3); "support" `VRFY`
  by always answering `550` (§7.3); drop the postmaster mailbox (§4.5.1); take excessive advantage
  of the right to reject, which the RFC frames as a threat to the ubiquity that makes the system
  work (§7.9).

**20. List Administrator** — the person *or other entity* who administers a list, and the
destination the list expander redirects errors to (§3.9; §3.9.2; §4.4 names the same party the
list maintainer; §7.3 notes list administrators installing protections against harvesting).

- *Key actions*: administer the list; receive the errors from final deliveries.
- *Cannot*: alter the message header From — the list expander is forbidden it on their behalf
  (§3.9); prevent the envelope return address from becoming theirs, which is a MUST on expansion
  (§3.9).

**21. Implementer** — named where the RFC hands a judgment to build time rather than run time:
evaluate the 251 and 551 forwarding codes carefully (§3.4, line 1162); handle null-reverse-path
messages correctly when building automated processors (§4.5.5); study the multihoming procedures
for IPv6 and dual-stack (§5.2, line 3937).

- *Key actions*: choose, before deployment, what the automations above will do at every point the
  RFC leaves a MAY.
- *Cannot*: act during a session — every decision is frozen before the first `220`; claim full
  compliance while answering `500` or `502` to `VRFY` (§3.5.3); advertise in `EHLO` what the
  server will not implement (§4.2.4).

### Folded, not dropped

Four behaviors that look like roles and are not — each is an act of a role already cataloged, and
giving each its own entry would inflate the catalog and, worse, invite a lane apiece:

| Behavior | Belongs to | Cited |
|---|---|---|
| Trace writer — prepend Received, insert Return-path at final delivery | SMTP server, in its relay, delivery and gateway roles | §4.4 (3156), §3.7.2 (1575) |
| Loop detector — count Received lines, threshold at least 100 | SMTP server; provisions for trivial loops are a MUST | §6.3 (4065) |
| Address verifier — answer `VRFY` and `EXPN` | SMTP server, under Site Operator configuration | §3.5.2 (1335), §7.3 (4237) |
| Listener — keep a pending listen on port 25, accept concurrent connections | SMTP server | §4.5.4.2 (3774) |

---

## Which roles get a lane

### First: two pools, not one — which dissolves the kit's own contradiction

The research record notes a **DISCREPANCY**: the corpus uses *swimlane* in three incompatible
senses and never reconciles them — a Step 1 board row that 🟧 event nodes live in, a Step 3 lane
per role, and a Step 6 team boundary — with senses one and two said to pull in opposite directions
on how many lanes to create [`event-modeling-research:research/METHOD-REFERENCE-DETAIL.md`
lines 476-486].

Under the owner's framing the contradiction is an artifact of counting two pools as one. The kit's
own row types make the split visible: the only rows named anywhere in the seven skills are
`actor`, `interaction` and `swimlane`; 🟧 events live in the `swimlane` row, ⬜ screens and
automations in the `actor` row [same file, lines 498-500].

| Pool | What occupies it | Partitions available |
|---|---|---|
| **Actor pool** (`actor` row) | ⬜ screens, automations — whoever acts | by role (Step 3 sense), by owning team (Step 6 sense), by inside/outside the boundary |
| **Event pool** (`swimlane` row) | 🟧 events — what happened | by ownership, by chapter, by stream root, by causing actor, by team |

Senses two and three were never in competition with sense one; they partition a different pool.
And senses two and three are not in competition with each other either — they are two partitioning
ideas over the same pool, and a board picks the one that makes a rule visible. [INFERENCE, resting
on the owner's correction and the row types above.]

### The actor-pool lane set for this model

The charter fixes what is in the timeline: *a conformant inbound RFC 5321 + RFC 7504 server,
operated by us, whose obligations begin at `MessageAccepted`* (`docs/model-altitude.md` §2.1). The
lane set I would actually draw is the one already on the board:

| Lane | Occupied by | Node type | Why it is one lane |
|---|---|---|---|
| **Remote client** | whichever role holds the SMTP client position this session | ⬜ command screens | One type: issue a line, read a reply |
| **Operator** | a human watching | ⬜ view screen — session transcript | Reads only; issues no 🟦 command |

Two lanes. Our own server gets none — it is the system under model, and its behavior is 🟦
commands, 🟧 events and 🟩 read models, not an actor-row occupant.

**Which roles share the Remote client lane, and why.** Roles 3, 5, 6 and 10 — originating system,
relay, gateway, submission client — plus a human at a terminal, all occupy it together. Step 1's
restraint rule is the test and it passes cleanly — *"Before adding a lane, check whether an
existing lane already covers the element's type. If yes, place the element in that lane."* And the
prohibition is explicit: **never** add a lane "just to group things visually or because a second
role appears"
[KIT `eventmodeling-brainstorming-events/SKILL.md` lines 281-289].

The type is identical, and the RFC says so three times over. §2.3.2 makes client a position a host
occupies, not a kind of host. §2.1 gives one dialog regardless of who is on the far end. And the
model already records that the server cannot distinguish them anyway: `HELO` bar.com from
192.0.2.10 is the same line whether Postfix or a person produced it (`docs/event-model.md`). The
one place the model *would* branch on which of them it is — whether this client may relay through
us — was ruled a separate context, not a lane: the Directory context, H3 resolved
(`docs/DECISIONS.md`, `docs/model-altitude.md` §4.3).

Note what this does **not** license: the remote client gets ⬜ command screens, not an automation
node. Actor-generic lanes do not make every machine an automation — automation-ness turns on
working a todo list of *ours*, which the remote client does not (`docs/event-model.md`).

**Roles that get no lane here, and where they act instead.** This is the corrected inference doing
its work — the reason is scope, and it is different for each group:

| Roles | Where they act | Why not a lane here |
|---|---|---|
| Retry scheduler, address resolver, bounce generator, delivery system | The outbound model | Already ruled a separate model — the roles invert, and *"the automation lives there"* (`docs/model-altitude.md` §4.3, `docs/event-model.md`) |
| Alias resolver, list expander, message store | Downstream of `MessageAccepted` | The charter's obligations begin there; expansion is past the boundary |
| Postmaster, Mailbox Owner | The Directory context | Postmaster is a required **address**, an obligation on the server (§4.5.1) — a Directory fact, not an actor |
| Site Operator, Implementer | Before the session | Configuration and build-time decisions; they arrive on the board as 🟤 `Given`, never as a step |
| Originating User, List Administrator | Another system's model | The submitting user is in the sender's model; the list administrator in the list's |

A lane records who acts **during** the timeline. Every role above acts **on** it. [INFERENCE.]

**Where a lane would be added, outbound.** One automation lane for our delivery processor — the
retry scheduler, address resolver and bounce generator are the same type (our processors working
our todo lists), so restraint puts them together. The one candidate for a second lane is the
bounce generator, and it has the right kind of warrant: an explicit rule separates it from the
queue it would otherwise sit beside — *"A queuing strategy MUST NOT send error messages in response
to error messages"* (§4.5.4), reinforced by the MUST NOT at §6.1 when the reverse path is null.
That is Step 1's second condition, *"An explicit business rule requires a distinct lane"*, rather
than the forbidden reason of a second role appearing. I would draw it as one lane until the
outbound model is walked and the loop-prevention rule is either visible on the board or not.
[INFERENCE — flagged, not settled.]

### The event-pool partition

Same principle, applied to the other pool, and this project already did it. `docs/event-model.md`
partitions the events by **ownership** — Edge holding `ConnectionAccepted`, `ClientIdentified`,
`SessionReset`, `SessionClosed`; Transaction holding `ReversePathDeclared`, `RecipientAccepted`,
`RecipientRejected`, `DataRequested`, `MessageAccepted`, `MessageRejected`, `TransactionAborted`;
Directory split off as a separate context rather than a lane.

The competing partitions and why they lose here:

| Partitioning idea | What it yields on this board | Verdict |
|---|---|---|
| By owning entity (adopted) | Edge / Transaction | Makes a real rule visible: `RSET` discards the transaction and clears its buffers while the session survives — the server MUST NOT close on it (§4.1.1.5) |
| By causing actor | One lane — every event traces to the remote client | Degenerate; carries no information |
| By chapter | One lane — the model is one session | Degenerate at this altitude |
| By owning team (Step 6 sense) | Not yet determinable | Deferred; no team boundary has been drawn |

The adopted partition is not the truth about these events, it is the view that makes the `RSET`
rule readable. Under a different charter — say an operations model of a whole mail plant — the
partition by owning team would win and Edge/Transaction would carry nothing. That is exactly the
owner's third claim, instantiated. [THIS PROJECT for the partition; INFERENCE for the reading of
why it wins.]

---

## What Step 3's gate becomes when the roles are automations

### The four texts, which do not agree

| Text | Says | Cited |
|---|---|---|
| Orchestrator gate | Every **human** role from the Role Catalog has at least one screen | `eventmodeling-orchestrating-event-modeling/SKILL.md:195` |
| Storyboarding checklist | Every **human** role from the Role Catalog has at least one **swimlane** | `eventmodeling-storyboarding-events/SKILL.md:607` |
| Storyboarding sub-step 5 | "Every human role in the catalog MUST have its own swimlane. **Every system actor that has a UI or todo-list view gets a swimlane too.**" | same file, line 216 |
| Storyboarding validation | If **a role** — unqualified — has zero screens, add screens or remove it from the catalog | same file, line 249 |

Two of the four say *human*, one says *screen* where another says *swimlane*, and one drops the
qualifier entirely.

### Verdict: satisfiable here, and vacuous in general — and sub-step 5 already fixes it

**Satisfiable on this board.** The Operator is a human role in the timeline with a ⬜ view screen.
The gate is met, non-vacuously, by exactly one role — which is worth noticing, because the
original inference predicted zero.

**Vacuous in general.** For a domain whose actors are processes, the gate ranges over an empty set
and passes without testing anything. A gate that cannot fail is not a gate. It would have passed
just as silently on a catalog that had lost the retry scheduler, the resolver and the bounce
generator altogether — three roles the specification loads with MUSTs from end to end.

**In need of restatement, and the kit supplies the material.** Sub-step 5's second sentence is
already actor-generic: the qualifier is *having a UI or a todo-list view*, not being human
[`eventmodeling-storyboarding-events/SKILL.md:216`]. Its placement rule then says human roles get
SCREEN nodes and system actors and processors get AUTOMATION nodes, both placed during
storyboarding [same file, lines 464-500]. Read together, the general rule is there — the gate is a
special case of it that got frozen into the orchestrator and then hardened into a species test.

**Restatement** [INFERENCE, ours, not settled]:

> Every role in the Role Catalog that acts **within this model's timeline** has at least one node
> in the actor row — a ⬜ SCREEN if it works a rendered view, an AUTOMATION if it works a todo
> list. A role that acts **on** the model rather than in it — through configuration, at build
> time, as a required address, or inside another context — stays in the catalog with the place it
> acts recorded, and gates nothing here.

Two properties worth stating: it is satisfiable and failable in a domain with no humans at all,
and it still catches the failure the original gate was built to catch — a role in the timeline
that nobody drew.

### The validation rule needs a third branch

As written, a role with zero screens is either missing screens or does not belong in the catalog
[line 249]. Applied to Postmaster that reads *remove it in Step 1* — which would delete a
MUST-level obligation (§4.5.1) from the only place the model records obligations. The role is
real, cited, and permission-bearing; it simply does not act in this timeline. The third branch:

> …or the role acts outside this model's altitude. Keep it in the catalog and record where it
> acts. It gates a different model, or no model.

[INFERENCE.] The cost of not having this branch is a catalog that shrinks to whatever the current
board happens to draw, which is the failure mode Step 8's completeness check exists to prevent —
it asks that every role have at least one command path
[`eventmodeling-brainstorming-events/SKILL.md` lines 372-420, naming Step 8 as a consumer].

---

## Flow candidates and hotspots

**To the method repo (rule 11) — method-generic, SMTP-free:**

1. *Pool and partition dissolves the three-senses swimlane discrepancy.* The actor row and the
   event row are two pools; lane-ness is a partitioning of a pool, not a property of its contents;
   any partition is one view among several. Resolves the recorded DISCREPANCY without picking a
   winner among the three senses.
2. *The lane is actor-generic.* Restates the storyboarding gate so it ranges over roles that act
   in the timeline rather than over humans, and adds the third branch to the validation rule.
   Both are corrections to kit text, so they travel with the kit citations above.

**Hotspots raised by this catalog** (rule 12 — on the model, at the point they bite):

- 🟥 **Message submission server.** Its rules are outside our normative set; §3.6.3 declares the
  arrangements private and out of scope. Bites wherever a slice needs submission semantics.
- 🟥 **Bounce generator lane.** One automation lane or two, outbound. §4.5.4's MUST NOT is the
  only warrant for a second, and it cannot be tested until the outbound model is walked.
- 🟥 **`VRFY` and `EXPN` as Site Operator configuration.** The site's choice changes what the
  server may answer (`252` when disabled, §7.3) — configuration reaching into the reply set. It
  arrives as 🟤 `Given`, but which `Given`, and at which step, is not yet placed.
