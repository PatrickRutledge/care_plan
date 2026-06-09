# Goal / Objective / Task — the three-layer model

A refinement that came out of comparing the data-anchored model to real chronic-
care-management (CCM) practice. It corrects a flattening that most systems (and
our own early workflows) fall into: treating "A1c < 7.0%" as a *goal*. It is not.

> A good clinician does **not** set a goal of "A1c < 7.0%." They set a goal of
> **"diabetes well-controlled"** or **"maintain and improve ADLs"** — and then
> use a measurable **objective** (an A1c range, an ADL score) to indicate
> progress, and assign **tasks** (prescribe a regimen, refer to PT, schedule
> visits, call the patient) that the team can actually *complete*.

## The three layers

| Layer | Definition | Examples | Nature |
|---|---|---|---|
| **Goal** | the clinical *aim / state* being sought | "well-controlled diabetes"; "maintain & improve ADL"; "limit exacerbations"; "decrease risk of death" | qualitative; the destination |
| **Objective** | the *measurable* range/value that indicates the goal | A1c held in 6.5–7.5%; ADL score ≥ baseline; 0 exacerbations / 6 mo | quantitative; **measured, not completed** |
| **Task** | a concrete *completable action* assigned to a team member | prescribe medication regimen; refer to PT; schedule 4 appointments; call patient; arrange transportation | binary; **done / not done** |

A goal is a *direction*. An objective is the *yardstick*. A task is *work someone
can finish*.

## The accountability principle (why this separation is the whole point)

Three tiers of control, which the data must keep distinct:

```
  TEAM controls          PATIENT controls         NATURE controls
  ─────────────          ────────────────         ───────────────
  Task completion   →    Adherence/engagement →   Objective achievement
  (prescribe, refer,     (attend, take meds,      (does the A1c actually
   schedule, call)        follow through)          move; does decline stop)
  binary: done/not       influenceable, not        not anyone's "fault"
                          guaranteed
```

- A **task** must be completable by the assignee. "Schedule four appointments" is
  a task (the care manager can complete it). "Patient attends four appointments"
  is **not** a team task — it is a patient expectation. Model it as a task whose
  `performer`/owner is the **patient**, never as something the team "completes."
- The care manager's work on **SDOH barriers** (transportation, financial) *is*
  completable team work — removing obstacles on the pathway to care — even though
  it cannot guarantee the patient walks the path.
- **Task completion ≠ objective achievement.** All tasks can be done, the patient
  fully adherent, and the objective still unmet — because biology, not effort,
  decides. Never credit the team for outcomes they didn't control, and never
  blame the team for nature.

## How WF3 (evaluate) must change

The follow-up loop must evaluate **two independent things**, not one:

| Question | Measured by | If "no", the remedy is… |
|---|---|---|
| Did the team do its job? | **task completion** rate | fix execution / remove barriers (accountability) |
| Did the objective move? | **objective** value vs. range | revise the plan/approach — or accept it's biology |

The re-direct-vs-close decision (WF3 gateway) keys off the **objective**; the
team's performance review keys off **task completion**. Conflating them
mis-assigns blame and hides the real problem.

## Goal types in chronic care (mostly NOT "achievement")

Because chronic-care goals are usually about holding ground, the objective's
*success condition* differs by goal type:

| Goal type | Success condition | Objective shape | FHIR achievementStatus |
|---|---|---|---|
| **Achievement** | cross a threshold | value < / > target | `achieved` / `not-achieved` |
| **Maintenance** | stay in range | value within band | `sustaining` / `worsening` |
| **Prevention** | event does *not* occur | count of events over time = 0 | `sustaining` / `worsening` |
| **Risk reduction** | risk score trends down | score vs. prior | `improving` / `worsening` |

Our early examples used only **achievement**. Chronic care is mostly the other
three. The data model already supports them (a `target` can be a *range*, and
`Goal.achievementStatus` has `sustaining`), but the workflows must use them.

## Mapping to the standards

| Layer | CDA R2.0 | FHIR R4 |
|---|---|---|
| Goal (aim) | `observation moodCode="GOL"`, `value xsi:type="CD"` (a *state*, e.g. "controlled") | `Goal.description` + `Goal.category` |
| Objective (measure/range) | a `GOL` observation linked `entryRelationship typeCode="COMP"`, `value xsi:type="IVL_PQ"` (a range) | `Goal.target.measure` + `detailRange` |
| Task (action) | `act moodCode="RQO"` → `EVN` | `Task` (status = completion) |
| Patient-expected task | `act moodCode="RQO"`, `performer`/owner = **patient** | `Task.owner = Patient` |
| SDOH barrier removal | `act moodCode="RQO"` addressing an SDOH `observation` | `Task` with `reasonReference → Condition` (SDOH) |
| Task completed (team control) | `statusCode="completed"` + `inFulfillmentOf` | `Task.status="completed"` |
| Objective met (nature) | objective `observation value` vs range | `Goal.achievementStatus` |

**Key links in data:**
- Goal `=[COMP]=` Objective (the goal *comprises* a measurable objective)
- Task `=[REFR]=` Goal (the task *serves* the goal)
- Task `=[RSON]=` SDOH/problem (the task's *reason*)
- Objective `measured by` outcome observation (nature's verdict)

See [`careplan-goal-model-example.xml`](careplan-goal-model-example.xml) for a
schema-valid instance of all three layers with the accountability separation.
