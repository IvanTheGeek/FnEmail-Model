# Exploring — what a 🟩 View is: informing, not deciding

**Asked and answered against the corpus, 2026-08-11. Nothing here is ruled.** The question was
Ivan's, in two parts: *does Dymitruk indicate or demonstrate that a View makes a decision, or does
it inform an actor who then decides* — and *does a View only filter?* Both have clear answers in
the corpus, and both bear on questions this repo already has open. This file is the reasoning
record and the citation trail; it changes no document and settles no FnEmail form. What it feeds is
named at the end.

This is a **corpus survey**, so it is exploration material under
[`AGENTS.md`](../AGENTS.md) rule 14 — read for its trail, and corrections go in place under rule 4.

---

## Scope, and what a locator means here

Searched: the canonical article and posts on eventmodeling.org, 35 talk transcripts, 47 podcast
transcripts, and the method reference's own survey of Dilger's kit and book. All paths below are
relative to the private research repo, prefixed `event-modeling-research:`, and every quotation
was opened at its locator rather than recalled (rule 3).

⚠️ **Every talk and podcast transcript is a machine transcript** — lowercase or auto-punctuated,
with transcription errors. They are quoted exactly, errors included, and labeled at each use
(rule 2). Each transcript file is a **single line**, so a locator is a **character offset**, not a
line number. Offsets below were measured in this session; where an earlier report gave a different
one, the measured value is used.

The one thing this survey cannot do is attribute the podcast voice. The episodes carry **no
speaker labels**, so every podcast citation is *"identified as Dymitruk by anchors"* at best —
`event-modeling-research:research/GWT-FINDINGS.md` §1 sets out that reasoning and rates it high
confidence, not verified. The written article carries `author: adymitruk` in its front matter and
is the only first-party written source; the talks are first-party by voice. **The load below is
carried by the article and the talks.** The podcast is corroboration.

---

## 1. The definition, in Dymitruk's own written words

`event-modeling-research:research/archive/eventmodeling-org/posts/what-is-event-modeling/index.md`,
front matter `author: adymitruk`. The View is introduced as one half of a pair, line 61:

> *"there are 2 very fundamental pieces that must be added to the blueprint which show 2 core
> features of any information system: Empowering the user and informing the user."*

🟦 Commands are the empowering half. 🟩 Views are the other, line 80:

> *"The second part of any information system is the ability to inform the user about the state of
> the system. Our hotel guest should know about what days are available for certain types of rooms
> they are interested in staying in."*

Line 85 says what one is made of — *"A view into the facts already in the system has been changing
as these new events were being stored"* — and line 152 says where it goes: *"we now have to link
information accumulated by storing events back into the UI via views (aka read-models)."*

**So the definition has three parts, and none of them is behavior:** a View is (a) a fold of stored
🟧 events, (b) into a shape that (c) informs a consumer, placed where that consumer is. Dilger's
one-line gloss — *"Making sense of what happened in the system"* — is the same thing said shorter,
and is Dilger's wording, not Dymitruk's.

---

## 2. Does the View decide? No — five independent lines of evidence

### 2.1 The passivity clause is definitional, not incidental

Article line 87, immediately after saying a View is specified like a command:

> *"Specifying how a view behaves is very similar to the way we specify how we accept commands with
> one difference. The views are passive and cannot reject an event after it's been stored in the
> system."*

Rejection is the only decision the method models. A View is defined by not having it.

### 2.2 The reason a View Slice has no `When` is always "there is nothing to decide"

Five sessions across six years, each giving a *different* reason for the same omission — and every
reason is a version of *no decision here*. The chronology is
`event-modeling-research:research/GWT-FINDINGS.md` §1; the two clearest, both machine transcripts:

**Dymitruk, 2020-10-30** —
`extracted/talk-transcripts/podcast-exploring-axon-episode-8-event-modeling-with-adam---BZk35QFjYmg.txt`
@33587:

> *"the state view is really given when then without the command so it's a given then because you
> can't reject state it is what it is"*

…and @33695, on what is left once rejection is gone:

> *"given that I've had these events happen to my system all I can do is make a conclusion in my
> test"*

**Dymitruk, YOW! 2023** —
`extracted/talk-transcripts/event-modeling-from-beginner-to-expert-adam-dymitruk-yow-2--Pin_B-AbdXE.txt`
@22723:

> *"same thing with these State views except they don't have that uh when part because they're
> passive"*

**A conclusion is not a decision.** The 🟦 Command spec asserts a transition and may reject; the
🟩 View spec asserts a shape and may not.

### 2.3 He narrates the handoff — informed, *then* the actor acts

**Dymitruk, workshop 2021-11-19** —
`extracted/talk-transcripts/event-modeling-workshop-adam-dymitruk--gyhR5Wey6_s.txt`
@34730, machine transcript:

> *"we get into a pattern of waves meaning you do an action you get a response from the action a
> recording of it you get informed about what's next … and then the user has more information that
> they can now click the checkout button and continue on to the next piece"*

The View ends at *"more information"*. The click is a separate element. Same shape in Copenhagen,
`extracted/talk-transcripts/copenhagen-ddd-event-modeling-with-adam-dymitruk--U_MwAEf8V_A.txt`
@18339, machine transcript: *"those events are used in projections or views you then show that
information back to the user and it's that's kind of enabling people to do something giving them
feedback informing them to do the next thing with more information"*.

### 2.4 The connection rule makes the deciding actor structurally unavoidable

Article line 46:

> *"command to event, event to state view, state view to UI or processor, UI/processor to command.
> Connecting an event directly to a command is not allowed."*

A 🟩 View can never reach a 🟦 Command. A ⬜ screen or a processor always stands between, and that
intermediary is where the deciding happens. Copenhagen @70969, machine transcript, on why:

> *"you cannot have an event kickoff a command directly that's the most common mistake you need a
> reed model that to do lists of what a processor is supposed to kickoff for you automatically"*

(`reed model` is the transcription error for *read model*.) The stated motive at the same offset is
disclosure, not computation: *"we want to know why and how it got kicked off with using what
information"*.

### 2.5 A View may be empty and the model still runs

`extracted/podcast-transcripts/episode-04.txt` @14849, machine transcript, voice identified as
Dymitruk by anchors: *"even if these State views are empty that's fine"*, and @15084 he calls a
low-scenario one *"a dumb information pump"*. **A decision-maker cannot be empty; an information
channel can.** This is the cheapest test of the whole question and it comes out unambiguously.

---

## 3. The one place it looks like the View decides — and why it still doesn't

The **to-do list**. Copenhagen @71626, machine transcript:

> *"these to-do lists are views for processors to understand what they're to do next in terms of
> automation"*

Here the 🟩 View's rows *are* the work, and the processor issues 🟦 commands off them with no human
judgment anywhere. That is the strongest counter-case in the corpus. Three things blunt it:

- **The Automation slice is specified as two Given/When/Thens, and the decision is in the second.**
  Article line 107: *"The specification for this is always done in 2 parts: The Given-When-Then for
  creating the to-do list and the Given-When-Then for executing the command."*
- **The same read model serves a human who decides and a machine that doesn't.**
  `extracted/podcast-transcripts/episode-46.txt` @8763: *"the automation's done by a human. And so,
  the instrument that's keeping track if you're finished yet is the same read model um like a
  to-do list that you know, an automation would have"*, and @8962: *"can I replace the automation
  with a human and vice versa"*. If the box is unchanged when the decider changes, the deciding is
  not in the box.
- **A processor is an actor**, occupying the same slot as a screen. The View informs it; it acts.

The near-miss on the human side is UI affordance — Copenhagen @73195, machine transcript: *"you can
gray out the things that are not at play"*. The View supplies the state that grays the control. It
does not reject the command if the control is used anyway; that is still the 🟦 Command Slice's
`Then`.

---

## 4. Does a View only filter? No — it folds and derives; filtering is a late, contested extra

### 4.1 The formula makes it a fold over the whole history

**Dymitruk, 2019-07-10** —
`extracted/talk-transcripts/11-event-modeling-with-adam-dymitruk--9EQiozD9wgQ.txt`
@53364, machine transcript:

> *"the new events are equal to the function of the command and all the previous events and likewise
> the state is just a function of all all events in the system"*

### 4.2 It computes, in his own worked examples

| Where | What the View does that is not filtering |
|:--|:--|
| Article line 80, 85 | Hotel calendar — availability computed from room inventory against bookings |
| Article line 85 | Cleaning list — *"which rooms are ready to be cleaned"*, a derived work-list |
| YOW! 2023 @22723 | Invoice — applies *"a discount code"* and *"items that have different taxation rules"* |
| `MbkDDnqUm90` @46613 | *"something that's doing aggregation uh across multiple entities"* |
| `episode-18.txt` @45137 | *"that's a state view where you have a calculation you know based on who the current user is"* |
| `episode-17.txt` @24040 | reducers *"not just being pure calculations but also lookup tables and other entities"* |

All machine transcripts except the article rows. YOW! 2023 @6353 states the job outright: *"the
fact that a whole bunch of stuff happened we still got to make sense of it to in order to do
something"*.

### 4.3 Calculation is not merely allowed in a View — it is *required* to live there

Dilger's kit is categorical and the method reference records it: totals, averages and counts are
read models, **not** events; *"If a value changes multiple times, it's a read model."* Read-model
field mappings admit `latest:`, `aggregate:` and `derived:` forms. See
`event-modeling-research:research/METHOD-REFERENCE.md` → *Field metadata* and *Elaborate
Scenarios*. This is **Dilger's kit stating it**, and it is consistent with Dymitruk's own examples
above rather than an addition to them.

### 4.4 A View may even be the thing that informs a Command's check

**Dymitruk, YOW! 2023** @36852, machine transcript, on a per-till limit across many cash registers:

> *"you can simply bring in some of the information from a state view into the command to say at
> this point I know that the outstanding balance on all of them is this and therefore it can be
> used as part of the calculation as I'm calculating the state of that single till to make sure
> that the total isn't over a th"*

Read the shape precisely, because it is the whole answer to part one in miniature: **the View
supplies the number; the Command does the comparison and the rejecting.** Corroborated at
`extracted/podcast-transcripts/episode-38.txt` @34430 — *"an extra read model in front of your
command to to rein in information from from events that are outside of your aggregate"* — where it
is priced as a cost, not offered as a place to put a rule.

### 4.5 Where filtering *does* appear, it is the recent `When`, and it is not settled

The filtering claim traces to exactly three podcast episodes, and there it is a **query with a key
over the read model's output rows** — the 🟧 events stay in the `Given`. `episode-18.txt` @55186:
*"And it's our filtration."* `episode-24.txt` @30581: *"there's a a sneaky way to put in a when in
there"*, for *"how you filter on what you want to output"*. `episode-43.txt` @58765 puts its own
status plainly: *"the standard I'd like to push forward"* — an aspiration, and it has not reached
the talks or eventmodeling.org.

⚠️ **The word reversed roles.** In 2019, `9EQiozD9wgQ` @53480, *filtration* names the thing being
**excluded** from the specification — *"obviously you optimize that you don't actually do that
there's filtration and all sorts of other things to make it more you know easier to program"*, an
implementation optimization outside the spec. In the podcast it *is* the `When`. Same word, opposite
role, and worth knowing before leaning on either.

---

## 5. Positions, with their stance (rule 16)

| Position | What the corpus does | What we do |
|:--|:--|:--|
| A 🟩 View informs an actor; the actor decides | **Prescribes** — the passivity clause and the connection rule are both prescriptive, and every worked example enacts it | **Adopt** |
| A 🟩 View is a fold that may aggregate and derive, not a filter | **Prescribes** (Dilger's kit, explicitly) and **enacts** (Dymitruk's examples); no source anywhere restricts a View to selection | **Adopt** |
| Row selection belongs in a View `When` | **Divided** — three podcast episodes against six years of talks, the article, and the author's own stated aspiration-not-standard | **Open** — already flagged, see below |

The corpus is **silent** on protocol-level modeling generally, so applying any of this to SMTP is
an extension of the method's demonstrated range — the standing caveat in
`event-modeling-research:research/METHOD-REFERENCE.md` → *When it does not fit*.

---

## 6. What this does not settle, and where it lands

**It rules nothing.** Three things it bears on, all owned elsewhere and all still open:

- **The consulted-view question**
  ([`paths/EXPLORE-view-slice.md`](paths/EXPLORE-view-slice.md), and the ⚠️ on `SessionState` in
  [`event-model.md`](event-model.md)). This survey supports the shape rather than deciding the
  form: `RecipientDirectory` informs `RcptTo`, and `RcptTo` decides `250` or `550`. Putting the
  accept/reject rule *inside* the view would be the corpus violation — consulting one is ordinary.
- **The view-`When` form and its missing key** — `event-model.md` ships `WHEN Query(session)`, whose
  only support is those three episodes, and which names no key. §4.5 above sharpens why that is
  uncomfortable rather than resolving it: episode 24's own complaint is about an *implicit
  requirement*, which is exactly what an unnamed key is.
- **The `Where` row question** in [`paths/EXPLORE-view-slice.md`](paths/EXPLORE-view-slice.md). That
  file already reached, from the walk alone, the distinction this survey reaches from the corpus:
  a `Given` **enumerates** and a `Where` **selects**. Two routes to one place is worth recording;
  it still does not decide whether the row belongs.

**Not in scope and deliberately not touched:** whether any of this should flow to the method repo
under rule 11. The generic half — what a View is — is method material and belongs there if it goes
anywhere; the SMTP consequences stay here. Nobody asked for the move, so nothing is moved.

---

## Citation index

Every entry opened at its locator in this session. Research-repo paths, prefixed
`event-modeling-research:`. **Talk and podcast files are machine transcripts and locators are
character offsets** (`@`); article locators are line numbers.

| Source | Locator | Carries |
|:--|:--|:--|
| `research/archive/eventmodeling-org/posts/what-is-event-modeling/index.md` (`author: adymitruk`) | 46 | the connection rule |
| *ibid.* | 61 | empowering / informing |
| *ibid.* | 80, 85 | the View's job; hotel calendar; cleaning list |
| *ibid.* | 87 | views are passive, cannot reject |
| *ibid.* | 107 | automation specified in 2 parts |
| *ibid.* | 152, 163, 167 | views link information back to the UI; one spec per command or view; origin and destination |
| `extracted/talk-transcripts/11-...--9EQiozD9wgQ.txt` (2019-07-10) | @53364, @53480 | the two formulas; *filtration* as an excluded optimization |
| `extracted/talk-transcripts/podcast-exploring-axon-...--BZk35QFjYmg.txt` (2020-10-30) | @33587, @33695 | cannot reject state; only a conclusion |
| `extracted/talk-transcripts/copenhagen-ddd-...--U_MwAEf8V_A.txt` | @18339, @70969, @71626, @73195 | the cycle; no event to command directly; to-do lists are views for processors; graying out |
| `extracted/talk-transcripts/event-modeling-workshop-...--gyhR5Wey6_s.txt` (2021-11-19) | @34730 | the wave, and the user clicking after being informed |
| `extracted/talk-transcripts/event-modeling-from-beginner-to-expert-...--Pin_B-AbdXE.txt` (YOW! 2023) | @6353, @7513, @22723, @36852 | make sense of it; empowering; passive, no `When`; state view feeding a command's calculation |
| `extracted/talk-transcripts/microservices-meetup-munich-...--MbkDDnqUm90.txt` | @46613 | aggregation across entities |
| `extracted/podcast-transcripts/episode-04.txt` | @14849, @15084 | empty state views are fine; dumb information pump |
| `extracted/podcast-transcripts/episode-17.txt` | @24040 | reducers beyond pure calculations |
| `extracted/podcast-transcripts/episode-18.txt` | @45137, @55186 | a state view with a calculation; *"our filtration"* |
| `extracted/podcast-transcripts/episode-24.txt` | @30581 | the *"sneaky way"* to add a `When` |
| `extracted/podcast-transcripts/episode-38.txt` | @34430 | a read model in front of a command, priced as cost |
| `extracted/podcast-transcripts/episode-43.txt` | @58765 | *"the standard I'd like to push forward"* |
| `extracted/podcast-transcripts/episode-46.txt` | @8763, @8962 | same read model for human or automation |
| `research/GWT-FINDINGS.md` | §1, §2, §3 | the no-`When` chronology; the `When` as query not event filtering; the fold-versus-transition role split |
| `research/METHOD-REFERENCE.md` | *Field metadata*, *Column types*, *When it does not fit* | mapping forms; the two column shapes; the protocol-modeling caveat |
