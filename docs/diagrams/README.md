# Diagrams — FnEmail inbound event model

Mermaid renderings of `../event-model.md` v0.3. GitHub renders these inline; the Android app shows
the source, so read this page on GitHub or desktop.

**Colors** — Event **orange** · Command **blue** · Read Model **green** · Screen **white** ·
external **yellow** · hotspot **red**.

**Terminology.** *Slice*, not column — Adam's own word. The two slice types are named here as this
project prefers them, with the corpus synonyms in parentheses:

| This project | Corpus synonyms |
|---|---|
| **Command Slice** | state change · write column · `state-change` |
| **View Slice** | state view · read model · query · `state-view` |

**Rendering vocabulary.** Settled empirically in
[`EXPERIMENT-block-labels.md`](EXPERIMENT-block-labels.md) — the Mermaid docs specify none of it.
One `classDef` per element type carries **color, left alignment and height together**; labels use
`<b>` for the title and `\n` for breaks; markdown never works; boxes never wrap.

**No arrows.** Meaning is carried by row position and left-to-right time. Rows are fixed:

```
row 1   Actor / Screen
row 2   Command (blue)  ·or·  Read Model (green)   ← never both; that is the slice type
row 3   Event (orange)
```

A Command Slice reads **top to bottom**. A View Slice reads **bottom to top**. Same three rows.

---

## 1. Command Slice

*(aka state change, write column)*

Slice 4, `MailFrom`.

```mermaid
block
  columns 1
  actor["<b>Remote client</b>\nMAIL FROM:&lt;Smith@bar.com&gt;"]
  cmd["<b>MailFrom</b>\nreverse_path"]
  evt["<b>MailTransactionStarted</b>\nreverse_path"]

  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:70px
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:70px
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:70px
  class actor screen
  class cmd command
  class evt event
```

### One event per command

**The target is exactly one.** More than one is legal, but it usually signals that several
responsibilities have been folded into a single command, and it should be investigated rather than
accepted.

This is a stronger rule than the corpus states. Dilger names the shape as an anti-pattern —
*"left chair"*, one command to many events — but calls the four anti-patterns *"not red flags, but
something to keep an eye on."* Here it is a design target with a stated reason: **multiple events
usually means multiple responsibility.**

It already earned its keep. In v0.1 `SubmitContent` emitted three event types plus a per-recipient
fan-out. Scoping to inbound removed the fan-out and the completeness check removed a redundant
event, leaving one command, one event — and the model got simpler, not poorer.

```mermaid
block
  columns 2
  ok["<b>✅ one command → one event</b>"] bad["<b>⚠️ one command → many events</b>\n'left chair'"]
  c1["<b>Command</b>"] c2["<b>Command</b>"]
  e1["<b>Event</b>"] e2["<b>Event</b> · <b>Event</b> · <b>Event</b>"]

  classDef okh fill:#eef7ee,stroke:#4a4,stroke-width:1px,color:#000,text-align:left,height:50px
  classDef badh fill:#fdf1e7,stroke:#c85,stroke-width:1px,color:#000,text-align:left,height:50px
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:50px
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:50px
  classDef warn fill:#f5a04f,stroke:#c85,stroke-width:2px,stroke-dasharray:5 3,color:#000,text-align:left,height:50px
  class ok okh
  class bad badh
  class c1,c2 command
  class e1 event
  class e2 warn
```

**`RcptTo` does not trip this rule.** `RecipientAccepted` and `RecipientRejected` are
*alternatives* — accepted **or** rejected, never both — not a fan-out.

---

## 2. View Slice

*(aka state view, read model, query)*

Slice 3, `SessionState`. Same three rows, read **bottom to top**.

```mermaid
block
  columns 1
  consumer["<b>consumed by</b>\nMailFrom · RcptTo · BeginData"]
  rm["<b>SessionState</b>\nidentified\ntransaction_open"]
  evts["<b>ConnectionAccepted</b>\n<b>ClientIdentified</b>\n<b>SessionReset</b>"]

  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:70px
  classDef readmodel fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:70px
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:70px
  class consumer screen
  class rm readmodel
  class evts event
```

A View Slice may draw on events from **anywhere earlier** on the timeline. It is placed at its
**consumer**, not next to its sources — which is why `SessionState` sits at slice 3 while its
sources are slices 1 and 2.

Many events feeding one read model is the *"right chair"* shape. Expected for a view that
accumulates; worth watching if it grows without a matching growth in the question it answers.

---

## 3. Both slice types, side by side

The shared row structure, and why the two never mix. Row 2 holds a command **or** a read model —
that choice *is* the slice type.

```mermaid
block
  columns 2
  h1["<b>COMMAND SLICE</b>\nread top → bottom"] h2["<b>VIEW SLICE</b>\nread bottom → top"]
  a1["Actor / Processor"] a2["Actor / Processor"]
  b1["<b>Command</b>"] b2["<b>Read Model</b>"]
  c1["<b>Event</b>"] c2["<b>Event(s)</b>"]

  classDef hdr fill:#e8e8e8,stroke:#444,stroke-width:1px,color:#000,text-align:left,height:55px
  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:55px
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:55px
  classDef readmodel fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:55px
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:55px
  class h1,h2 hdr
  class a1,a2 screen
  class b1 command
  class b2 readmodel
  class c1,c2 event
```

Every slice in the model is one of these two. Automation and Translation are **compositions** — a
View Slice feeding a Command Slice — which is why Dymitruk specifies an automation as *two*
Given-When-Thens rather than one.

---

## 4. Inbound timeline — session establishment

Slices 1–3. Columns are slices; rows are element types; `space` marks a row a slice does not use.

```mermaid
block
  columns 3
  h1["<b>1 · AcceptConnection</b>"] h2["<b>2 · Helo</b>"] h3["<b>3 · SessionState</b>"]
  s1["tcp connect"] s2["HELO bar.com"] s3["<b>consumed by</b>\nslices 4, 6, 8"]
  k1["<b>AcceptConnection</b>"] k2["<b>Helo</b>"] r3["<b>SessionState</b>\nidentified"]
  e1["<b>ConnectionAccepted</b>\npeer_address"] e2["<b>ClientIdentified</b>\nclaimed_domain"] space

  classDef hdr fill:#e8e8e8,stroke:#444,stroke-width:1px,color:#000,text-align:left,height:45px
  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:70px
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:70px
  classDef readmodel fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:70px
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:70px
  class h1,h2,h3 hdr
  class s1,s2,s3 screen
  class k1,k2 command
  class r3 readmodel
  class e1,e2 event
```

---

## 5. Inbound timeline — the transaction

Slices 4–7. `RecipientDirectory` is the Translation boundary onto the Directory context (H3
resolved), so its source is outside this model — shown **yellow**.

```mermaid
block
  columns 4
  h4["<b>4 · MailFrom</b>"] h5["<b>5 · RecipientDirectory</b>"] h6["<b>6 · RcptTo</b>"] h7["<b>7 · TransactionState</b>"]
  s4["MAIL FROM"] s5["<b>consumed by</b>\nslice 6"] s6["RCPT TO"] s7["<b>consumed by</b>\nslices 6, 8, 9"]
  k4["<b>MailFrom</b>"] r5["<b>RecipientDirectory</b>\nis_local"] k6["<b>RcptTo</b>"] r7["<b>TransactionState</b>\nrecipient_count"]
  e4["<b>MailTransactionStarted</b>\nreverse_path"] x5["<b>Directory context</b>\ntranslated — external"] e6["<b>RecipientAccepted</b>\n<b>RecipientRejected</b> 550"] space

  classDef hdr fill:#e8e8e8,stroke:#444,stroke-width:1px,color:#000,text-align:left,height:45px
  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:75px
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:75px
  classDef readmodel fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:75px
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:75px
  classDef external fill:#f7e463,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:75px
  class h4,h5,h6,h7 hdr
  class s4,s5,s6,s7 screen
  class k4,k6 command
  class r5,r7 readmodel
  class e4,e6 event
  class x5 external
```

The **yellow** cell is what distinguishes Translation from Automation — the source events belong to
another context.

---

## 6. Inbound timeline — content and close

Slices 8–12. `DataPhaseEntered` is **red**: hotspot **H1**, on trial because its only candidate
consumer is the transcript.

```mermaid
block
  columns 5
  h8["<b>8 · BeginData</b>"] h9["<b>9 · SubmitContent</b>"] h10["<b>10 · Reset</b>"] h11["<b>11 · Quit</b>"] h12["<b>12 · SessionTranscript</b>"]
  s8["DATA"] s9["content · then dot"] s10["RSET"] s11["QUIT"] s12["<b>Operator</b>\nreads transcript"]
  k8["<b>BeginData</b>"] k9["<b>SubmitContent</b>"] k10["<b>Reset</b>"] k11["<b>Quit</b>"] r12["<b>SessionTranscript</b>"]
  e8["<b>DataPhaseEntered</b>\nH1 — consumer?"] e9["<b>MessageAccepted</b>\nqueue_id · received_at"] e10["<b>TransactionAborted</b>"] e11["<b>SessionClosed</b>"] space

  classDef hdr fill:#e8e8e8,stroke:#444,stroke-width:1px,color:#000,text-align:left,height:45px
  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:75px
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:75px
  classDef readmodel fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:75px
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000,text-align:left,height:75px
  classDef hotspot fill:#f4a0a0,stroke:#a33,stroke-width:2px,color:#000,text-align:left,height:75px
  class h8,h9,h10,h11,h12 hdr
  class s8,s9,s10,s11,s12 screen
  class k8,k9,k10,k11 command
  class r12 readmodel
  class e9,e10,e11 event
  class e8 hotspot
```

`SessionTranscript` is the **"right chair"** shape — one read model fed by every event above.
Expected here; worth watching if it grows.

---

## 7. Worked paths as sequences

Sequence diagrams, not blocks — a path is a conversation over time, which is what this diagram
type is for.

**Path 1** — [`../paths/helo-multi-recipient.md`](../paths/helo-multi-recipient.md). Three
recipients, one rejected mid-flow. 13 steps over 11 slices.

```mermaid
sequenceDiagram
    participant C as bar.com
    participant S as foo.com (us)
    S->>C: 220 foo.com Service Ready
    C->>S: HELO bar.com
    S->>C: 250 foo.com
    C->>S: MAIL FROM Smith at bar.com
    S->>C: 250 OK
    C->>S: RCPT TO Jones at foo.com
    S->>C: 250 OK
    C->>S: RCPT TO Green at foo.com
    S->>C: 550 No such user here
    C->>S: RCPT TO Brown at foo.com
    S->>C: 250 OK
    C->>S: DATA
    S->>C: 354 Start mail input
    C->>S: content then dot
    S->>C: 250 OK queued as q7F3A21
    C->>S: QUIT
    S->>C: 221 closing channel
```

**Path 2** — [`../paths/helo-single-recipient.md`](../paths/helo-single-recipient.md). One
recipient, no errors. 11 steps over 11 slices.

```mermaid
sequenceDiagram
    participant C as foo.com (relay)
    participant S as xyz.com (us)
    S->>C: 220 xyz.com Service Ready
    C->>S: HELO foo.com
    S->>C: 250 xyz.com is on the air
    C->>S: MAIL FROM JQP at bar.com
    S->>C: 250 OK
    C->>S: RCPT TO Jones at XYZ.COM
    S->>C: 250 OK
    C->>S: DATA
    S->>C: 354 Start mail input
    C->>S: headers, body, then dot
    S->>C: 250 OK queued as x91B4C7
    C->>S: QUIT
    S->>C: 221 closing channel
```

Outcome is relative to the actor. Path 1 is a complete success for the server, **partial** for the
sender, and a failure for `Green`. Path 2 succeeds for everyone. `Reset` — slice 10 — remains the
only slice no path has touched.
