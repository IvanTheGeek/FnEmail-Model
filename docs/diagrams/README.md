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

---

## 1. Command Slice

*(aka state change, write column)*

Three rows, one slice: **actor → command → event**. This is slice 4, `MailFrom`.

```mermaid
block-beta
  columns 1
  actor["Remote client · MAIL FROM Smith at bar.com"]
  cmd["MailFrom"]
  evt["MailTransactionStarted"]

  style actor fill:#ffffff,stroke:#444,stroke-width:2px,color:#000
  style cmd fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000
  style evt fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
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
block-beta
  columns 2
  ok["✅ one command · one event"] bad["⚠️ one command · many events"]
  c1["Command"] c2["Command"]
  e1["Event"] e2["Event · Event · Event"]

  style ok fill:#eef7ee,stroke:#4a4,stroke-width:1px,color:#000
  style bad fill:#fdf1e7,stroke:#c85,stroke-width:1px,color:#000
  style c1 fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000
  style c2 fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000
  style e1 fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
  style e2 fill:#f5a04f,stroke:#c85,stroke-width:2px,stroke-dasharray:5 3,color:#000
```

---

## 2. View Slice

*(aka state view, read model, query)*

Rows run the other way: **events → read model → actor**. This is slice 3, `SessionState`.

```mermaid
block-beta
  columns 1
  evts["ConnectionAccepted · ClientIdentified · SessionReset"]
  rm["SessionState · identified, transaction_open"]
  consumer["consumed by MailFrom, RcptTo, BeginData"]

  style evts fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
  style rm fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000
  style consumer fill:#ffffff,stroke:#444,stroke-width:2px,color:#000
```

A View Slice may draw on events from **anywhere earlier** on the timeline. It is placed at its
**consumer**, not next to its sources — which is why `SessionState` sits at slice 3 while its
sources are slices 1 and 2.

Many events feeding one read model is the *"right chair"* shape. Expected for a view that
accumulates; worth watching if it grows without a matching growth in the question it answers.

---

## 3. Both slice types as a grid

The two shapes side by side. Read each slice top to bottom; read the board left to right.

```mermaid
block-beta
  columns 2
  h1["COMMAND SLICE"] h2["VIEW SLICE"]
  a1["Actor / Processor"] a2["Event(s)"]
  b1["Command"] b2["Read Model"]
  c1["Event"] c2["Actor / Processor"]

  style h1 fill:#e8e8e8,stroke:#444,stroke-width:1px,color:#000
  style h2 fill:#e8e8e8,stroke:#444,stroke-width:1px,color:#000
  style a1 fill:#ffffff,stroke:#444,stroke-width:2px,color:#000
  style b1 fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000
  style c1 fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
  style a2 fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
  style b2 fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000
  style c2 fill:#ffffff,stroke:#444,stroke-width:2px,color:#000
```

Every slice in the model is one of these two. Automation and Translation are **compositions** — a
View Slice feeding a Command Slice — which is why Dymitruk specifies an automation as *two*
Given-When-Thens rather than one.

> ⚠️ **Syntax note.** Sections 1–3 were written using `block-beta`. As of Mermaid **11.16.1** the
> keyword is plain **`block`** — the beta suffix is gone. These render today, so the old form is
> still being accepted, but they should be migrated. Label formatting is being settled first in
> [`EXPERIMENT-block-labels.md`](EXPERIMENT-block-labels.md), then this page gets rebuilt from
> whatever wins.

---

## 4. Inbound timeline — session establishment

Slices 1–3. Flowchart syntax, as a rendering control against §3.

```mermaid
flowchart LR
  subgraph c1["1 · AcceptConnection"]
    direction TB
    s1["tcp connect"]
    k1["AcceptConnection"]
    e1["ConnectionAccepted<br/>peer_address"]
    s1 --> k1
    k1 --> e1
  end

  subgraph c2["2 · Helo"]
    direction TB
    s2["HELO bar.com"]
    k2["Helo"]
    e2["ClientIdentified<br/>claimed_domain"]
    s2 --> k2
    k2 --> e2
  end

  subgraph c3["3 · SessionState"]
    direction TB
    r3["SessionState<br/>identified"]
  end

  e1 --> r3
  e2 --> r3

  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
  classDef readmodel fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000
  class s1,s2 screen
  class k1,k2 command
  class e1,e2 event
  class r3 readmodel
```

---

## 5. Inbound timeline — the transaction

Slices 4–7. `RecipientDirectory` is the Translation boundary onto the Directory context (H3
resolved), so its source is outside this model — shown **yellow**.

```mermaid
flowchart LR
  subgraph c4["4 · MailFrom"]
    direction TB
    s4["MAIL FROM"]
    k4["MailFrom"]
    e4["MailTransactionStarted"]
    s4 --> k4
    k4 --> e4
  end

  subgraph c5["5 · RecipientDirectory"]
    direction TB
    x5["Directory context<br/>translated — external"]
    r5["RecipientDirectory<br/>is_local"]
    x5 --> r5
  end

  subgraph c6["6 · RcptTo"]
    direction TB
    s6["RCPT TO"]
    k6["RcptTo"]
    e6["RecipientAccepted"]
    e6b["RecipientRejected<br/>550"]
    s6 --> k6
    k6 --> e6
    k6 --> e6b
  end

  subgraph c7["7 · TransactionState"]
    direction TB
    r7["TransactionState<br/>recipient_count"]
  end

  e4 --> r7
  e6 --> r7
  r5 --> k6

  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
  classDef external fill:#f7e463,stroke:#444,stroke-width:2px,color:#000
  classDef readmodel fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000
  class s4,s6 screen
  class k4,k6 command
  class e4,e6,e6b event
  class x5 external
  class r5,r7 readmodel
```

`RcptTo` shows two outcomes, which is **not** the one-event-per-command warning from §1. They are
alternatives — accepted **or** rejected, never both — not a fan-out.

---

## 6. Inbound timeline — content and close

Slices 8–12. `DataPhaseEntered` is red: **H1**, on trial because its only candidate consumer is
the transcript.

```mermaid
flowchart LR
  subgraph c8["8 · BeginData"]
    direction TB
    s8["DATA"]
    k8["BeginData"]
    e8["DataPhaseEntered<br/>H1 — consumer?"]
    s8 --> k8
    k8 --> e8
  end

  subgraph c9["9 · SubmitContent"]
    direction TB
    s9["content then dot"]
    k9["SubmitContent"]
    e9["MessageAccepted<br/>queue_id, received_at"]
    s9 --> k9
    k9 --> e9
  end

  subgraph c1011["10–11 · Reset / Quit"]
    direction TB
    k11["Reset / Quit"]
    e11["TransactionAborted<br/>SessionClosed"]
    k11 --> e11
  end

  subgraph c12["12 · SessionTranscript"]
    direction TB
    r12["SessionTranscript<br/>operator view"]
  end

  e8 --> r12
  e9 --> r12
  e11 --> r12

  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
  classDef hotspot fill:#f4a0a0,stroke:#a33,stroke-width:2px,color:#000
  classDef readmodel fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000
  class s8,s9 screen
  class k8,k9,k11 command
  class e9,e11 event
  class e8 hotspot
  class r12 readmodel
```

---

## 7. Worked paths as sequences

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
sender, and a failure for `Green`. Path 2 succeeds for everyone. `Reset` remains the only slice no
path has touched.
