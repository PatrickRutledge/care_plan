# The Clinical Care-Planning Process (the spine)

> **Sequencing decision:** the clinically-correct process is defined **here, first**,
> anchored to HL7 data. A vendor (Athena) is an *implementation target* mapped in
> later — never the source of the design. Athena's care-plan fields exist but are
> not connected to an intentional model of care, so they cannot teach us the
> direction of services. We define that direction; then we bind it to Athena.

This document is the **comparison to clinical process** that every workflow in
this repo is measured against. Workflows are not invented for software
convenience — each one realizes a phase of the recognized clinical care-planning
cycle, and each phase is carried by a specific HL7 data atom.

---

## 1. The care-planning cycle

The clinical process for care planning is a closed loop (the nursing-process
"ADPIE" lineage, generalized for team-based, longitudinal care). Each phase
answers a clinical question, consumes the previous phase's output, and is carried
by a data atom that does not change when a UI changes.

| # | Phase | Clinical question it answers | HL7 data atom (the carrier) | Realized by workflow |
|---|---|---|---|---|
| 1 | **Assess** | What is going on with this patient? | `observation moodCode="EVN"`, `Condition`/problem obs | (planned) Assess & intake |
| 2 | **Identify problem/need** | What problem are we addressing? | problem `observation` / `Condition` | (planned) Problem identification |
| 3 | **Set goal** | What outcome are we aiming for, by when? | `observation moodCode="GOL"` (`Goal`) | **WF1 – Establish** |
| 4 | **Direct services** | What services will move us toward the goal? | `act/procedure/encounter moodCode="RQO"/"INT"` | **WF1 – Establish** |
| 5 | **Implement** | Who does it, internal or external? | `performer` + `representedOrganization` | **WF1 / WF2** |
| 6 | **Monitor / follow up** | Is it happening; is it on track? | `statusCode` transitions; interim `observation EVN` | **(next) WF3 – Follow-up** |
| 7 | **Evaluate** | Did the services move the goal? | `observation moodCode="EVN"` value vs. `GOL` target | **(next) WF3 – Follow-up** |
| 8 | **Revise or close** | Continue, change, or complete? | `statusCode`; `relatedDocument typeCode="RPLC"` | **WF2 – Complete** + (next) Revise |
| → | **loop** | back to Assess with new information | the same `id`s carried forward | — |

The arrows between phases are **data dependencies**, and that is the whole point
of the next section.

---

## 2. Why the temporal order cannot break

Each phase **consumes the data the previous phase produced**. This is a stronger
guarantee than "we agreed on a step order" — the order is enforced by the
information itself:

```
Assessment data  ─▶  Problem      ─▶  Goal        ─▶  Service (RQO)
   (observations)     (condition)      (GOL)           justified by ▲ goal
                                                          │
                                          Evaluation (EVN value) ◀─ performed
                                                          │
                                          compared to ▲ goal target
                                                          │
                                                   Revise / Close
```

- You **cannot evaluate** (phase 7) without a goal target to compare against
  (phase 3).
- You **cannot justify a service** (phase 4) without a goal it serves (phase 3).
- You **cannot set a meaningful goal** (phase 3) without an identified problem
  (phase 2).

So "establish → follow-up → complete" isn't a UI sequence somebody can
re-order — **one step literally builds on the data output of the prior step.**
Change the clicks all you want; the dependency graph is invariant because it's a
graph of *information*, not of screens.

---

## 3. The intentional direction of services (the core idea)

This is the thing Athena will not give us, and the reason to model it ourselves.

> **A care plan is not a list of tasks. It is a set of goals that *direct*
> services. Every service is justified by its link back to a goal.**

The directionality is a loop encoded entirely in data:

```
        directs                evaluated against
 GOAL ───────────▶ SERVICE ───────────────────▶ GOAL
 (GOL)             (RQO/INT)   (EVN value vs. target)
   ▲                                               │
   └───────────────  revise based on outcome  ◀────┘
```

How the data carries the intent:

| Intentional relationship | The data that encodes it |
|---|---|
| This service exists **to serve this goal** | `entryRelationship typeCode="REFR"` (act → `GOL`) |
| This service is **because of this problem** | `entryRelationship typeCode="RSON"` (act → condition) |
| This evaluation **measures this goal** | `observation EVN` value compared to `GOL` target |
| This revision **replaces** the prior intent | `relatedDocument`/`entryRelationship typeCode="RPLC"` |

**Auditable consequence:** a service with *no* `REFR` link to a goal is, by this
model, an **unjustified service** — it appears in the plan with no intent behind
it. The data model makes "are our services actually directed at our goals?" a
*queryable* question rather than a chart-review opinion. That is the operational
power of anchoring intent in the standard.

---

## 4. Workflow roadmap (each workflow = a phase of the cycle)

| Workflow | Cycle phases | Status |
|---|---|---|
| **WF4 — Assess & identify problems** | 1 Assess, 2 Problem | ✅ built (feeds WF1) |
| **WF1 — Establish care plan** | 3 Set goal, 4 Direct services, 5 Implement | ✅ built |
| **WF2 — Complete tasks** | 5 Implement, 8 Close | ✅ built |
| **WF3 — Follow-up & evaluate** | 6 Monitor, 7 Evaluate, 8 Revise | ✅ built |
| **WF5 — Referral / external hand-off** | 4 Direct services (external) | ✅ built |

**All 8 cycle phases are now realized, and the material is fully exhausted.** The
discovery process is complete end-to-end: assess (WF4) → set goal & direct
services (WF1) → implement & complete (WF2) → monitor, evaluate, revise (WF3) →
loop, with external service direction and closed-loop referral covered by WF5.
Every workflow has a schema-valid CDA instance, a FHIR mirror, and a process map.

This closes the HL7-material discovery phase. The agreed next phase (a separate
effort) is **comparison** — reconciling this model against actual clinical
practice and external frameworks (e.g. CMS, AHRQ). That comparison is deliberately
deferred so the discovery stands on its own first.

---

## 5. Only after this: Athena

Once the cycle, the temporal dependencies, and the goal→service→evaluation intent
are specified and validated against clinical practice, the Athena step is purely
a **binding exercise**: map each data atom above to an Athena API field/endpoint
through a thin adapter (see the planned `athena-mapping` column). Because the
intent lives in the data model — not in Athena — Athena's immature/disconnected
care-plan fields constrain *what we can store today*, but not *what the workflow
means*. As Athena connects those fields, the workflow is already correct and
waiting.
