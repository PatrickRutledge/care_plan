# Goal, Target, and Activity — using ONLY HL7-defined care-plan fields

> **Correction.** An earlier draft of this file invented a nested "objective"
> entity (a child observation under the goal). **HL7 defines no such entity.**
> This version uses only fields HL7 actually specifies. The whole point of this
> repo is to bring clarity to practitioners by grounding them in the standard —
> so we may not add structure the standard does not have. If HL7 has no field for
> it, it does not go in the care plan.

## What HL7 actually defines for a care plan (authoritative)

Source: FHIR R4 [CarePlan](https://hl7.org/fhir/R4/careplan.html) and
[Goal](https://hl7.org/fhir/R4/goal.html) resource definitions.

**A care plan has exactly these first-class parts:**

| HL7 field | What it is |
|---|---|
| `CarePlan.addresses` → Condition | the problems/concerns the plan manages |
| `CarePlan.goal` → Goal | the goals of the plan |
| `CarePlan.activity` → Task / ServiceRequest / MedicationRequest / Appointment | the actions (tasks/interventions) |
| `CarePlan.careTeam` → CareTeam | who is involved |

**A Goal has exactly these parts** (the relevant ones):

| HL7 field | Cardinality | What it is |
|---|---|---|
| `Goal.description` | 1..1 CodeableConcept | the desired **objective** — i.e. the *aim* ("well-controlled") |
| `Goal.target.measure` | 0..1 CodeableConcept | the parameter being tracked (e.g. Hemoglobin A1c) |
| `Goal.target.detailQuantity` / `detailRange` / `detailCodeableConcept` | 0..1 | the **target value** (a number, a range, or a coded state) |
| `Goal.target.dueDate` / `dueDuration` | 0..1 | the deadline |
| `Goal.lifecycleStatus` | 1..1 | proposed / active / completed … (the goal's own state) |
| `Goal.achievementStatus` | 0..1 | in-progress / improving / **sustaining** / achieved / not-achieved … |
| `Goal.category` | 0..* | treatment / dietary / behavioral … |
| `Goal.addresses` → Condition | 0..* | the problem the goal addresses |
| `Goal.outcomeReference` → Observation | 0..* | what actually changed (the measured result) |

## The three words practitioners use → HL7's actual fields

This crosswalk *is* the clarity. The clinician's mental model is right; it just
maps onto fewer HL7 records than people expect.

| Practitioner says | It is NOT a separate record — it is this HL7 field | FHIR | CDA |
|---|---|---|---|
| **Goal** (the aim: "well-controlled", "maintain ADL") | the goal's description | `Goal.description` | Goal Observation `code` / narrative |
| **Objective** (the measurable range/value) | **the goal's target — a field *inside* the goal** | `Goal.target.measure` + `target.detail[x]` | Goal Observation `value` |
| **Task** (the completable action) | an activity of the plan | `CarePlan.activity` → `Task`/`ServiceRequest` | `act moodCode="RQO"` |

**The headline for confused practitioners:** *the objective is not a thing you
create — it is the target field of the goal.* One Goal record carries both the
aim (`description`) and the measure (`target`). You do not make a goal, then make
an objective under it. You make **one goal that has a target.**

## A real difference between the two HL7 representations (clarity point)

Reading the definitions exposed an asymmetry worth knowing:

- **FHIR Goal separates the aim from the measure cleanly.** `description` holds
  the qualitative aim ("diabetes well-controlled") and `target.measure` +
  `target.detail` holds the metric ("A1c 6.5–7.5%"). So FHIR lets a clinician
  record the goal as a *state* and the objective as a *target* — exactly the
  distinction good clinicians make — **in one Goal, with no invention.**
- **CDA's Goal Observation is a single observation** (`code` + `value`). It tends
  to encode the measurable target directly (`code` = the measure, `value` = the
  target), so the qualitative aim has to ride in the `code` concept or the
  narrative. CDA collapses what FHIR separates.

Neither adds an "objective" record. The practitioner's "objective" is `Goal.target`
(FHIR) or the Goal Observation's `value` (CDA), in both cases **a field of the
goal.**

## The accountability separation — already built into HL7's fields

The earlier finding (team controls task completion; nature controls the outcome)
is not something we impose — **HL7 already separates these into different fields:**

| Question | Who controls it | HL7 field |
|---|---|---|
| Was the action done? | the team | `CarePlan.activity.detail.status` / `Task.status` |
| Did the measure move? | nature | `Goal.achievementStatus` + `Goal.outcomeReference` |

Activity status and goal achievement are *distinct fields by design*. So the
standard itself says: track "did we do the work" separately from "did the goal
get met." A clinician who conflates them is fighting the data model; one who
keeps them apart is using it as intended.

## Goal types map to existing fields too (no new structure)

| Goal type | Success condition | HL7 fields used |
|---|---|---|
| Achievement | cross a threshold | `target.detailQuantity` + `achievementStatus = achieved` |
| Maintenance | stay in range | `target.detailRange` + `achievementStatus = sustaining` |
| Prevention | event does not occur | `target.detailRange`/measure of events + `achievementStatus = sustaining` |
| Risk reduction | score trends down | `target.measure` (risk score) + `achievementStatus = improving` |

Most chronic-care goals are maintenance/prevention — and HL7's `achievementStatus`
has a literal **`sustaining`** code for "maintenance goal still being met." The
fields already exist; the discipline is using the right one.

## What we will not do

Add fields HL7 has not defined. No "objective" record, no nested target
observation, no invented relationships. The care plan contains exactly what HL7
specifies — that constraint is the source of the clarity, not a limit on it.

See [`careplan-goal-model-example.xml`](careplan-goal-model-example.xml) (CDA)
and [`careplan-goal-model-example.fhir.json`](careplan-goal-model-example.fhir.json)
(FHIR) for instances that use only these fields.
