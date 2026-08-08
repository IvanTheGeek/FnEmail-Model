# One fold, three consumers — state view, todo list, decision model

Events accumulate; something reads the accumulated state. Which pattern is that? The corpus turns
out to answer with a three-way split, and the split is decided by exactly one thing: **who
consumes the fold**. Not the shape of the data, not whether the actor is human, not how many
event types feed it. The consumer.

Researched 2026-08-08 against the method corpus (both authors, spoken and written — see *Sources*
at the end), prompted by the walk-through question of whether FnEmail's `TransactionState` is a
todo list. FnEmail appears here as the worked example; the finding is about the method.

---

## There is one artifact

Everything below is the same thing: a fold over events — a projection, a read model, the green
box. The corpus is explicit that the todo list is not a different element. Dymitruk, introducing
it as his saga replacement: the to-do list is *"another part of these State views"* (YOW talk,
machine transcript). Dilger's book, in the pattern chapter itself: *"Processor-Todo-Lists are
calculated based on the state of the System and typically provided by a Read Model"* — and, on
the composed pattern: *"An automation is basically just the combination of a State View, a State
Change and a gear symbol for the process."*

So the question is never *what is this artifact* — it is *who reads it*, and the answer sorts
every fold into one of three fates.

## Fate 1 — an actor reads it: the state view

The dataset provided to an actor so it can complete its task — the way a GUI takes a dataset and
produces the finished page. It is passive: the canonical article's view specification *"cannot
reject an event"*, and its scenarios are Given/Then. What it carries is decided by what the actor's
task needs, nothing more.

**FnEmail example:** `ServiceReady{server_domain, greeting_text}` — the dataset from which the
`220` greeting is rendered to the MTA Client. The walk's ruling that *the renderer owns the
constants and the view owns the facts* is this fate's discipline: see the
[WORKING walk](paths/WORKING-helo-direct-single-recipient-v2.md), *The form, in one paragraph*.

## Fate 2 — a processor reads it: the todo list

The same green box becomes a work queue the moment a **processor** consumes it to issue commands.
This is mandatory explicitness, not convenience — the connection grammar forbids the shortcut:
*"you cannot have an event kickoff a command directly that's the most common mistake"* (Copenhagen
DDD talk, machine transcript). Three mechanics distinguish the fate, and all three must be
present:

1. **A processor issues commands from it.** The gear works the list; the list exists *for* the
   gear.
2. **A check-off back-channel.** The list also subscribes to the completion events its own
   commands produce, so done work leaves the list. Dilger gives the pattern's two governing
   questions: *"How does the Processor get a new task to execute?"* and *"What needs to happen
   for this task to get checked off the Processor-Todo-List?"* — and draws the check-off feed as
   a dotted line outside the flow.
3. **A two-part specification.** The canonical article: *"The specification for this is always
   done in 2 parts: The Given-When-Then for creating the to-do list and the Given-When-Then for
   executing the command."*

The actor's species is explicitly not the line: *"you can turn any gear into a mechanical turk"*
(workshop, machine transcript) — a human working a screen-rendered list is the same pattern.

**FnEmail example:** outbound delivery, and nothing inbound. `MessageAccepted` opens a task;
a delivery or bounce event checks it off; the processor retries until something closes it —
Dilger's refund-processor walkthrough is the exact template, dead-letter escape included. The
model document already names this by pattern: the delivery agent *works a todo list*. Dymitruk
would extend the pattern all the way down: even a network send buffer is *"just a buffer
somewhere in memory a little index increasing"* — a todo list handed to a robot (YOW 2022,
machine transcript).

## Fate 3 — a command handler reads it: the decision model, which has no box

Here is the fact this document exists to record: **the fold a command validates against is
deliberately not on the model.** The connection grammar has four legal edges — *"command to
event, event to state view, state view to UI or processor, UI/processor to command"* — and
view-to-command is not among them. Verified structurally in Dymitruk's own drawio conference
model: 109 edges, none from a green box into a command.

Where does precondition state live instead?

- **On the model:** in the Given. Dilger's definition of a command scenario's Given is *"A set of
  events that brings the system into a specific state."* The preconditions are events, listed by
  the slice that needs them — dependency pointing backward, no box drawn.
- **In the implementation:** in the handler's own fold. *"The sole purpose of the command handler
  is to make a decision whether a command can be processed by validating the existing state of
  the system"* (the book's implementation chapters) — folded from events, never queried from a
  view. Dymitruk coined a separate name precisely to keep this out of the view category:
  *"projection of history"* (podcast 24, machine transcript), elsewhere *"the decision model"*,
  whose events are *"not necessarily the same events that are connected to the screen"*
  (podcast 38, machine transcript).
- **When outside data is genuinely needed:** a processor fetches it and *puts it on the command
  as a parameter*. The handler still never reads a view.

**FnEmail example:** `TransactionState`. The model defined it as a view answering *Open? Reverse
path? How many recipients?* for `RcptTo`, `BeginData` and `SubmitContent` — command validation
wearing a view's name. The walk exposed the truth by accident of discipline: under the
minimal-dependency rule, **no step ever consults it** — every command's `Given` cites events
directly, which is exactly where the corpus says decision state belongs. What remains of
`TransactionState` on the page is only its rendered role — drawing `250` OK — whose dataset is
empty, a question parked with the walk's step 6.

---

## The tests, compactly

| The fold is a… | when its consumer is… | and its signature is |
|:--|:--|:--|
| state view | an actor completing a task | dataset = what the task needs; passive; Given/Then |
| todo list | a processor issuing commands | check-off back-channel; two-part GWT; items leave |
| decision model | a command handler deciding | **no box** — events in the `Given`, fold in the handler |

Two boundary clarifications the corpus makes, both of which bit this project:

**Progress is not work.** Even for one long-running process, Dymitruk splits the fates: the work
is *"encapsulated in to-do lists"*, while where-the-process-stands *"is simply a read model for
progress"* (workshop, machine transcript). An accumulating count — `recipient_count` — is
progress-state; Dilger files calculated aggregations under the *Logic Read Model* pattern, never
the todo list.

**The reservation pattern does not override the consumer test.** Dymitruk assigns a
pending-reservations view to the todo-list side — *"it'll act as a to-do list which will be done
in the sequence"* (workshop, machine transcript) — but only because a **processor** executes the
second phase. SMTP's transaction has the reservation *shape* (`RCPT TO` reserves, the final dot
commits), yet the second phase is executed by the **client**, an external actor. No processor, no
todo list — the shape alone decides nothing.

---

## A verified gap, and why this document exists

Request/response protocol conversation state — a server tracking where a dialogue stands so it
can validate the next request — is **addressed nowhere in the corpus**. Swept: 47 podcast
episodes, every recorded talk, both sites. The nearest statements are the two analogs above (the
send buffer as todo list; *"your preconditions are really … all the events to the left"*). A
protocol server is off the method's map, and this three-fates reading is the map fragment FnEmail
drew to cover it: replies are state views rendered to the peer, autonomous work behind the
responsibility boundary is a todo list, and conversation position is decision-model state that
belongs in `Given` rows, not in boxes.

---

## Sources

Method quotations follow rule 7: brief, attributed, never reproduced wholesale. The canonical
article is public — [What is Event Modeling?](https://eventmodeling.org/posts/what-is-event-modeling/)
(grammar edge rule; two-part specification; views cannot reject). Book quotations are from
Dilger, *Understanding Eventsourcing* — the Processor-Todo-List pattern chapter (its two
questions, the back-channel), the Given/When/Then chapter (the Given as a set of events), the
planning chapter (automation = view + change + gear), the Logic Read Model pattern chapter, and
the implementation chapters (the command handler's sole purpose). Spoken quotations are
**machine transcripts** — lowercase, unpunctuated, no speaker identity — from the Copenhagen DDD
talk, the YOW talks, the workshop recording, and podcast episodes 24 and 38; irregularities
belong to the transcriber. All quotes were verified verbatim against the mirrored corpus in the
private research repository (rule 7: the mirror never enters this one), with file paths and
offsets recorded in the session that produced this document, 2026-08-08.
