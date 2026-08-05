# Diagrams — FnEmail inbound event model

Mermaid renderings of `../event-model.md` v0.3. GitHub renders these inline; the Android app
shows the source, so read this page on GitHub or desktop.

**Colours** — Event **orange** · Command **blue** · Read Model **green** · Screen **white** ·
hotspot **red**. See `../research/UPSTREAM-DEFECTS.md` for the one upstream file that disagrees.

**On the arrows.** Mermaid needs edges to stack nodes; they are a rendering necessity, not Event
Modeling semantics. Meaning is carried by lane position and left-to-right time.

---

## 1. A write column (state change)

One command, one event. This is slice 4, `MailFrom`.

```mermaid
flowchart TD
  S["Screen — remote client<br/>250 foo.com<br/>MAIL FROM Smith at bar.com"]
  C["MailFrom"]
  E["MailTransactionStarted<br/>reverse_path"]
  S --> C
  C --> E

  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
  class S screen
  class C command
  class E event
```

---

## 2. A read column (state view)

Events in, projection out, consumed by whatever sits in that column. This is slice 3,
`SessionState` — note its sources are from **earlier** columns, because a read model is placed at
its consumer, not at its source.

```mermaid
flowchart TD
  E1["ConnectionAccepted"]
  E2["ClientIdentified"]
  E3["SessionReset"]
  R["SessionState<br/>identified, transaction_open"]
  U["consumed by<br/>MailFrom, RcptTo, BeginData"]
  E1 --> R
  E2 --> R
  E3 --> R
  R --> U

  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
  classDef readmodel fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000
  classDef plain fill:#f0f0f0,stroke:#999,stroke-dasharray:4 3,color:#000
  class E1,E2,E3 event
  class R readmodel
  class U plain
```

---

## 3. The two column types

The four patterns reduce to these. Automation and Translation are compositions of both.

```mermaid
flowchart LR
  subgraph W["WRITE column"]
    direction TB
    ws["Trigger / Processor"]
    wc["COMMAND"]
    we["EVENT(s)"]
    ws --> wc
    wc --> we
  end

  subgraph R["READ column"]
    direction TB
    re["EVENT(s)"]
    rr["READ MODEL"]
    rs["Trigger / Processor"]
    re --> rr
    rr --> rs
  end

  classDef screen fill:#ffffff,stroke:#444,stroke-width:2px,color:#000
  classDef command fill:#8ecafc,stroke:#444,stroke-width:2px,color:#000
  classDef event fill:#f5a04f,stroke:#444,stroke-width:2px,color:#000
  classDef readmodel fill:#a8d98a,stroke:#444,stroke-width:2px,color:#000
  class ws,rs screen
  class wc command
  class we,re event
  class rr readmodel
```

---

## 4. Inbound timeline — session establishment

Slices 1–3.

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
resolved), so its source is outside this model.

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

Note `x5` is **yellow** — an external event, which is what distinguishes Translation from
Automation.

---

## 6. Inbound timeline — content and close

Slices 8–12. `DataPhaseEntered` is red: **H1**, an event on trial because its only candidate
consumer is the transcript.

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

  subgraph c1112["10–11 · Reset / Quit"]
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

`SessionTranscript` is the **"right chair"** shape — one read model fed by many events. Expected
here; worth watching if it grows.

---

## 7. The first worked path

`../paths/helo-multi-recipient.md`, as a sequence. 13 steps over 11 columns; `RcptTo` walked
three times.

```mermaid
sequenceDiagram
    participant C as Remote client (bar.com)
    participant S as FnEmail (foo.com)
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

Outcome is **relative to the actor**: complete success for the server, partial for the sender,
failure for `Green`. See the path document for why that matters to the golden-path question.
