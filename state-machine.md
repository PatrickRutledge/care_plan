# Care-Plan Task State Machine (the stable spine)

These diagrams render on GitHub. They describe the **data state machine** that your
business process maps onto. The clicks/screens/order can change freely; these
transitions are the contract that does not break.

> Verified against the schema in this repo
> (`schema/extensions/SDTC/...`): the `moodCode` and `statusCode`
> vocabularies and the `inFulfillmentOf` / `entryRelationship` linkages are real
> CDA R2.0 constructs, not invented.

---

## 1. The `moodCode` lifecycle of a single task

The same act (same `code`, same `id`) is walked from "requested" to "happened".
That walk **is** task completion — independent of any UI.

```mermaid
stateDiagram-v2
    [*] --> PRP: proposed (moodCode=PRP)
    PRP --> INT: accepted into plan (moodCode=INT)
    INT --> RQO: assigned / ordered (moodCode=RQO)
    RQO --> EVN: performed (moodCode=EVN)
    EVN --> [*]

    note right of RQO
        RQO = the open task (an order)
        carries performer, due window, priority
    end note
    note right of EVN
        EVN = the done event
        links back via inFulfillmentOf
    end note
```

---

## 2. The `statusCode` lifecycle within a task

`moodCode` says *what kind of reality*; `statusCode` says *where in its life*.
A task in `RQO` mood moves through these states as work proceeds.

```mermaid
stateDiagram-v2
    [*] --> new
    new --> active: work begins
    active --> suspended: blocked / on hold
    suspended --> active: resumed
    active --> completed: fulfilled
    active --> aborted: cancelled / no longer needed
    completed --> [*]
    aborted --> [*]

    note right of completed
        Pairs with a moodCode=EVN act
        + inFulfillmentOf the order
    end note
```

---

## 3. End-to-end: both workflows over the same atoms

```mermaid
flowchart TD
    subgraph PLAN["Workflow 1: Establish plan"]
        G["GOAL<br/>observation moodCode=GOL<br/>id=G1, statusCode=active"]
        T1["TASK internal<br/>act moodCode=RQO id=T1<br/>performer org=OUR-ORG"]
        T2["TASK external<br/>act moodCode=RQO id=T2<br/>performer org=OTHER-ORG"]
        G -. "entryRelationship REFR" .- T1
        G -. "entryRelationship REFR" .- T2
    end

    subgraph DO["Workflow 2: Complete tasks"]
        D1["COMPLETION<br/>act moodCode=EVN id=T1-DONE<br/>statusCode=completed"]
        OUT["GOAL outcome<br/>observation moodCode=EVN<br/>measured value vs target"]
    end

    T1 == "sdtc:inFulfillmentOf1 (FLFS)" ==> D1
    D1 -. "entryRelationship REFR" .- OUT
    G  -. "entryRelationship REFR" .- OUT

    classDef stable fill:#e6f4ea,stroke:#2E8B57,color:#000;
    class G,T1,T2,D1,OUT stable
```

**Reading the green nodes:** every box is a CDA data atom with a stable `id`.
The arrows are the *contract* (fulfillment + reference links). A UI redesign
rearranges *how* a human creates these nodes; it cannot change *what* the nodes
are or how they reconcile.

---

## 4. Internal vs. external — same atom, one differing field

```mermaid
flowchart LR
    A["act moodCode=RQO<br/>(a task)"] --> P["performer typeCode=PRF"]
    P --> E["assignedEntity"]
    E --> O{"representedOrganization"}
    O -->|"= OUR-ORG"| INT["INTERNAL task"]
    O -->|"= OTHER-ORG"| EXT["EXTERNAL task"]
```

There is no separate "external task" object. Internal vs. external is a single
data fact (`representedOrganization` + the `id` root namespace). This is why a
process built on the data model doesn't fork into brittle internal/external
branches.
