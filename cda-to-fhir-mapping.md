# CDA ↔ FHIR mapping (same atoms, two layers)

`careplan-example.xml` (CDA R2.0 — the legal snapshot) and
`careplan-example.fhir.json` (FHIR R4 — the live workflow) describe the **same
care plan**. This table is the Rosetta stone. Your business-process map is drawn
once, over the **concepts** in the middle column; each layer is just a
serialization of those concepts.

| Concept (the stable atom) | CDA R2.0 | FHIR R4 |
|---|---|---|
| The plan | `ClinicalDocument` (Care Plan template) | `CarePlan` |
| Patient | `recordTarget/patientRole` | `Patient` |
| Goal | `observation moodCode="GOL"` | `Goal` (+ `CarePlan.goal`) |
| Goal target value | `observation/value` | `Goal.target.detailQuantity` |
| A task (assigned) | `act moodCode="RQO"` | `Task` `intent=order` |
| Task identity (the thread) | `act/id` | `Task.identifier` / `Task.id` |
| Task lifecycle state | `moodCode` + `statusCode` | `Task.status` |
| Task is part of the plan | `entryRelationship` / section membership | `Task.basedOn → CarePlan` |
| Task serves the goal | `entryRelationship typeCode="REFR"` → goal | `Task.reasonReference → Goal` |
| Who performs it | `performer typeCode="PRF"` | `Task.owner` |
| Who requested it | `author` / requester | `Task.requester` |
| Who it's for | `subject` | `Task.for` |
| **Internal vs. external** | `performer/.../representedOrganization` | `Task.owner` + `CareTeam...onBehalfOf Organization` |
| Care team | participations across acts | `CareTeam` |
| Due / execution window | `effectiveTime` | `Task.executionPeriod` / `Task.restriction.period` |
| Urgency | `priorityCode` | `Task.priority` |
| **Task completion** | **new `act moodCode="EVN"`, `statusCode="completed"` + `sdtc:inFulfillmentOf1` → order** | **same `Task`, `status` → `completed`** |
| Referral to external | `participant typeCode="REFT"` | `CareTeam` member `onBehalfOf` external org |
| Sign-off / attestation | `legalAuthenticator` | `Provenance` (separate resource) |
| Plan revision | `relatedDocument typeCode="RPLC"` | `CarePlan.replaces` |

## The one genuinely different mechanic — task completion

This is the most instructive row, and it shows *why* anchoring to concepts (not
to either serialization) is what keeps your process portable:

- **CDA** is document/event-sourced: completion is an **additive** new act
  (`EVN`) that points back to the order (`RQO`) by `id` via `inFulfillmentOf`.
  The original order is never mutated — you get an immutable audit trail by
  construction. Fits CDA's "persistent, attestable snapshot" nature.
- **FHIR** is resource/state-machine: the **same** `Task` resource advances its
  `status` field `requested → in-progress → completed`. The `id` is still the
  thread; history is kept in `Task` versions / `Provenance`.

Same concept ("the task got done, and it reconciles to the order by identity"),
two mechanics. **Your process map says "task reaches Completed and reconciles to
its order."** It does not say "append an EVN act" *or* "PATCH status=completed."
That sentence is the contract; the mechanic is an implementation detail of the
layer you happen to be on. Swap CDA for FHIR — or swap vendors within either —
and the process step is unchanged.

## When to use which

| Use CDA when… | Use FHIR when… |
|---|---|
| You need a persistent, legally-attestable snapshot of the plan at a point in time | You need live, queryable task state that updates as work happens |
| Exchanging a complete signed document between organizations | Driving assignment, status, and notifications in an app/EHR |
| Regulatory submission / records of care | Real-time care coordination and dashboards |

Many real systems do **both**: run the live workflow on FHIR `Task`/`CarePlan`,
then emit a CDA Care Plan document as the signed snapshot at milestones. Because
both map to the same atoms in this table, that round-trip is mechanical.
