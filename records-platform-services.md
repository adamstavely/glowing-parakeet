# Service and endpoint map

Headless legal records and matter management platform. Companion to the platform overview, revision C.

Path parameters are written `:id` rather than `{id}` because braces are unreliable inside Mermaid node labels. The API contract uses `{id}`.

Service boundaries below are a decomposition proposal, not a settled deployment topology. Several of these could reasonably collapse into one service (record, filing, and file are a strong candidate for a single records service, since they share invariants and transactions). Split them at the seams where team ownership and scaling actually differ, not because the diagram drew a box.

---

## 1. Request path

How any call moves through the platform. Every transition on every resource passes through one action endpoint and one guard chain, which is what makes the invariants centrally enforceable.

```mermaid
flowchart LR
  C["Client<br/>UI, bot, integration"] --> GW["API Gateway<br/>authn, tenant, idempotency"]
  GW --> ACT["Action Gateway<br/>POST /:resource/:id/actions<br/>GET available transitions"]
  GW --> Q["Query services<br/>/search, /reports"]
  ACT --> G["Guard and Policy Engine<br/>existence, authz, state,<br/>invariants, routing"]
  G -->|routine| DOM["Domain services<br/>records, filings, files, matters"]
  G -->|controlled| WF["Workflow Service<br/>/workflows, /corrections"]
  WF --> WL["Worklist Service<br/>/worklists"]
  WF -->|on approval| DOM
  DOM --> SOR[("System of Record<br/>temporal state, truth")]
  SOR --> EV["Event Log<br/>/events, hashes only"]
  EV --> AN["Anchor Service<br/>/anchors"]
  AN --> ANS[("Anchor Store<br/>object lock")]
  SOR -->|rebuild| Q
```

---

## 2. Service inventory by plane

The full decomposition with the endpoints each service owns. Dependencies are drawn in diagrams 1 and 3 rather than here, so this stays readable as a reference.

```mermaid
flowchart TB
  subgraph EDGE["Edge and derived"]
    SRCH["Search Service<br/>/search"]
    REP["Reporting Service<br/>/reports"]
    CLS["Classification Service<br/>scored filing proposals"]
    IMP["Import Service<br/>/imports"]
    EXP["Export Service<br/>/exports"]
  end

  subgraph ACTP["Activity plane"]
    ACT["Action Gateway<br/>/:resource/:id/actions"]
    GRD["Guard and Policy Engine"]
    WF["Workflow Service<br/>/workflows, /corrections"]
    WL["Worklist Service<br/>/worklists"]
    EV["Event Log Service<br/>/events"]
    ANC["Anchor Service<br/>/anchors"]
  end

  subgraph DATA["Data plane"]
    PARTY["Party Graph Service<br/>/entities, /identifiers, /assertions,<br/>/sources, /relationships"]
    CONF["Conflicts Service<br/>/conflict-checks<br/>returns a decision, not the graph"]
    MAT["Matter Service<br/>/matters, /matters/:id/team,<br/>/matters/:id/events"]
    DOC["Document Service<br/>/documents<br/>working, declared, transient"]
    REC["Record Service<br/>/records"]
    FIL["Filing Service<br/>/filings, /filings/:id/events<br/>carries the series"]
    FILE["File Service<br/>/files, /files/:id/lifecycle"]
    HOLD["Hold Service<br/>/holds"]
    RENG["Retention Engine<br/>projection per filing"]
    DISP["Disposition Service<br/>/dispositions"]
  end

  subgraph CTL["Control plane"]
    FP["File Plan Service<br/>/series, /schemas"]
    VOC["Vocabulary Service<br/>/vocabularies, /terms"]
    RET["Retention Rules Service<br/>/retention-schedules"]
    WALL["Walls Service<br/>/walls, /policies"]
    NUM["Numbering Service<br/>/numbering-schemes"]
  end

```

---

## 3. Domain dependencies

The write side of the data plane, showing where the series attaches and how an obligation reaches disposition.

```mermaid
flowchart LR
  DOC["Document Service<br/>/documents"] -->|declare| REC["Record Service<br/>/records"]
  REC -->|file| FIL["Filing Service<br/>/filings"]
  SER["File Plan Service<br/>/series"] -->|classifies the filing| FIL
  FIL -->|placed into| FILE["File Service<br/>/files"]
  FILE -->|opened under| MAT["Matter Service<br/>/matters"]
  MAT -->|clearance at open| CONF["Conflicts Service<br/>/conflict-checks"]
  CONF -->|privileged read| PARTY["Party Graph Service<br/>/entities"]
  RUL["Retention Rules<br/>/retention-schedules"] --> RENG["Retention Engine<br/>one projection per filing"]
  FIL -->|projects a date| RENG
  HOLD["Hold Service<br/>/holds"] -->|suppresses| RENG
  RENG -->|latest projection| DISP["Disposition Service<br/>/dispositions"]
  DOC --> CAS[("Content Store<br/>content-addressed")]
  DISP -->|destroys content| CAS
```

---

## 4. Declare and file a document

The write path. Note that the series is applied by the filing, and that the event carries a hash rather than content.

```mermaid
sequenceDiagram
  autonumber
  participant C as Client
  participant A as Action Gateway
  participant G as Guard and Policy
  participant D as Document Service
  participant R as Record Service
  participant F as Filing Service
  participant N as Retention Engine
  participant S as System of Record
  participant E as Event Log

  C->>A: POST /documents/:id/actions (declare)
  A->>G: evaluate guards in order
  G->>G: existence, authz and walls, state, invariants
  G-->>A: routine, execute
  A->>D: pin version by content hash
  D->>R: create record (bytes and declaration fixed)
  R->>S: write record state
  S->>E: record.declared (id, version, hash)

  C->>A: POST /records/:id/actions (file)
  A->>G: evaluate guards
  G-->>A: routine, execute
  A->>F: create filing (series, position, primary)
  F->>F: enforce one primary per record
  F->>N: request projection for this filing
  N-->>F: disposition date from series trigger
  F->>S: write filing state and projection
  S->>E: filing.created (record, file, series)
  A-->>C: 200 with new state
```

---

## 5. Disposition run

The read-check-destroy path. The hold re-check inside the destroying transaction is the load-bearing step.

```mermaid
sequenceDiagram
  autonumber
  participant T as Scheduler
  participant N as Retention Engine
  participant P as Disposition Service
  participant W as Workflow Service
  participant H as Hold Service
  participant S as System of Record
  participant X as Content Store
  participant E as Event Log

  T->>N: sweep for eligible records
  N->>N: latest projection across all filings
  N-->>P: candidate set
  P->>H: any hold on any filing
  H-->>P: clear
  P->>W: open destruction review
  W-->>P: approved by records manager
  P->>S: begin transaction
  S->>H: re-check holds inside transaction
  H-->>S: still clear
  S->>X: destroy content
  S->>S: leave tombstone (ids, dates, authority)
  S->>E: record.destroyed (hash, certificate ref)
  P->>P: issue certificate with external timestamp
```

---

## 6. File lifecycle

Segments rather than rewind. Reopening opens a new segment and suspends the retention clock.

```mermaid
stateDiagram-v2
  [*] --> Open: open (conflicts cleared)
  Open --> Open: file, cross-file, transfer
  Open --> Closing: close requested
  Closing --> Open: denied
  Closing --> Closed: approved
  Closed --> Open: reopen (new segment)
  Closed --> Eligible: every filing projection matured
  Eligible --> Held: hold attaches
  Held --> Eligible: hold released
  Eligible --> Disposing: destruction approved
  Disposing --> Disposed: certificate issued
  Disposed --> [*]

  note right of Held
    Hold supremacy: no path
    from Held to Disposing
  end note
```

---

## 7. Filing lifecycle

Filings are never deleted. Withdrawal leaves a residual obligation so the ratchet survives.

```mermaid
stateDiagram-v2
  [*] --> Proposed: classifier below threshold
  [*] --> Active: filed by a person or high confidence
  Proposed --> Active: reviewer accepts
  Proposed --> Rejected: reviewer declines
  Active --> Active: reclassify series (reviewed)
  Active --> Superseded: refiled elsewhere
  Active --> Withdrawn: filed in error (reviewed)
  Superseded --> [*]
  Withdrawn --> [*]
  Rejected --> [*]

  note right of Withdrawn
    Residual obligation persists,
    provenance points at this filing
  end note
```

---

## Notes on the decomposition

**The Conflicts Service returns a decision, not the graph.** This is the shape that answers the open question about party graph readability. If callers can query `/entities` freely, the platform leaks who the firm represents against whom. Routing conflicts through a privileged service keeps the graph closed while still answering the question that matters at matter open.

**The Action Gateway is a chokepoint, deliberately.** Every transition on every resource passes through it, which is what makes the guard order and the invariants centrally enforceable. It is also a single point of catastrophic failure, so it needs its own change control, staged policy promotion, and dry-run against historical actions before any rule touching disposition goes live.

**The Retention Engine is a service, not a library.** Projections recompute when any trigger fires anywhere, including on files the caller was not touching. Embedding that logic in each domain service guarantees it drifts.

**Record, filing, and file could be one service.** They share transactions and invariants, and the primary-filing uniqueness constraint and the hold re-check both want to be transactional. Splitting them means distributed transactions or sagas for operations that are conceptually atomic. My recommendation is one records service with three resource families until scale forces otherwise.

**The Event Log is downstream of the system of record, never upstream.** Nothing reads from the log to serve a request. Search and reporting rebuild from the system of record, so a destroyed record cannot reappear in an index.
