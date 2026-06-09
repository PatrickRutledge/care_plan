# HL7 Conformance Audit

Purpose: verify **every** pattern in this repo against authoritative HL7
definitions, and flag anything invented, overstated, or confused. Triggered by
the goal/objective correction — if that confusion existed, others might too.

Authoritative sources used: the repo's own normative vocabulary
(`online-navigation/infrastructure/vocabulary/ActRelationshipType.htm`, `ActMood.htm`),
the CDA SDTC schema, and the FHIR R4 Goal/CarePlan resource definitions + official
examples.

---

## 0. Provenance of the examples ("was the example in the HL7 pages?")

The **field structure** is HL7's, verified against HL7's official FHIR Goal
example (a weight-loss goal): `description` = "Target weight is 160 to 180 lbs",
`target.measure` = LOINC Weight Measured, `target.detailRange` = 160–180, `dueDate`.
Our goal example uses the **identical shape** (`description` + `target.measure` +
`detailRange`) with diabetes content.

- **From HL7:** the pattern/fields. No HL7 example was copied; the shape was followed.
- **From us:** the clinical scenario (diabetes, A1c 6.5–7.5%, the specific tasks).
- Note: HL7's example restates the number in `description`; ours puts the
  qualitative aim there. Both are valid — `description` is a free CodeableConcept.

---

## 1. VERIFIED CORRECT (grounded in the repo's normative vocabulary)

| Code | HL7 definition (from repo vocab) | Our use | Verdict |
|---|---|---|---|
| `RSON` "has reason" | "the reason or rationale for a service" | task `RSON`→ problem / outcome | ✅ correct |
| `SPRT` "has support" | "an existing service is suggesting evidence for a new observation" | problem `SPRT`→ assessment obs | ✅ correct |
| `FLFS` "fulfills" | "source act fulfills (in whole or in part) the target act; source mood ≥ target" | `EVN` `inFulfillmentOf` `RQO` | ✅ correct |
| `COMP` "has component" | composition relationship | (only where genuinely compositional) | ✅ valid |
| moodCodes `GOL`/`RQO`/`INT`/`EVN`/`APT` | per ActMood | as used | ✅ correct |

---

## 2. OVERSTATED — corrected

### `REFR` does **not** mean "serves the goal"

This is the important one, because the **"intentional direction of services"**
narrative leaned on it.

- **What HL7 actually says** (repo `ActRelationshipType.htm`): `REFR` = **"refers
  to"** — *"A relationship in which the target act is referred to by the source
  act. This permits a simple reference relationship."* It is **generic and weak**.
- **What we claimed:** that a task's `REFR`→ goal link encodes "this service
  *serves* the goal" — the backbone of the intentional-direction story.
- **The reality:**
  - In **CDA**, the service→goal link rides on the generic `REFR` ("refers to").
    It does *not* assert "serves" — that is our interpretation layered on top.
  - In **FHIR**, there **is** an explicit field: `CarePlan.activity.detail.goal`
    = *"Goals this activity contributes toward."* That is a real "serves"
    semantic — first-class, not inferred.
- **Correction:** the *intentional direction of services* is a sound **design
  principle**, and it is **explicit in FHIR**. In **CDA** it is a **convention**
  on the generic `REFR` link, not a first-class HL7 meaning. Docs that said
  "`REFR` (serves)" are being corrected to "`REFR` (refers to); the explicit
  serves-the-goal link is FHIR `activity.detail.goal`."

---

## 3. QUESTIONABLE — flagged for resolution (not yet fixed)

1. **Patient as `performer`/`assignedEntity` (goal-model example).** CDA
   `assignedEntity` implies a *provider* role; modeling the **patient** as a
   performer is a stretch. FHIR `Task.owner = Patient` **is** explicitly allowed.
   *Proposed fix:* in CDA, represent patient-expected actions as patient
   instructions or a patient-authored act, and reserve `performer` for the team;
   keep `Task.owner = Patient` in FHIR.

2. **C-CDA `templateId`s asserted from memory.** Several document- and
   entry-level templateIds (e.g. the Care Plan, Progress Note, Referral Note IDs)
   were added illustratively and **not verified** against the C-CDA Implementation
   Guide. They do **not** affect base-CDA schema validity (a `templateId` is just
   an `II`), but a wrong one is misleading.
   *Status: in progress.* **WF1 re-anchored** in `careplan-establish.ccda.xml`
   using **verified** C-CDA templateIds (Care Plan `…1.15`, Health Concerns
   `…2.58`, Goals `…2.60`, Interventions `…21.2.3`, Goal Observation `…4.121`,
   Planned Intervention `…4.146`, Entry Reference `…4.122`), confirmed against
   HL7's published C-CDA template registry and Goal Observation example. WF2–WF5
   to follow this template. *Note:* validation is base-CDA XSD; full C-CDA
   conformance also needs Schematron (a separate step not run here).

3. **Goal-as-measurable-target in WF1 / WF3 / WF4.** Those earlier examples set
   the goal as an A1c value/threshold (`code` = A1c, `value` = 7.0%) — the C-CDA
   Goal Observation pattern. It is **valid HL7**, but it encodes the *objective*,
   and the narrative called it "the goal," reproducing the very goal/objective
   conflation we later corrected. *Proposed fix:* a consistency pass so the
   goal/objective language is uniform across all workflows.

---

## 4. NOTES (correct, but worth stating plainly)

- `sdtc:inFulfillmentOf1` is a real **HL7 SDTC extension**, not base CDA. Used
  correctly (`FLFS`), but it is an extension, not core.
- `ParticipationType` codes `REFB` / `REFT` / `CALLBCK` are **valid schema
  enumerations** (confirmed in `voc.xsd`); their exact directional semantics
  (referred-by vs referred-to) should be confirmed against the vocabulary before
  any production use.
- The accountability split is **genuinely HL7-native**: `Task.status` /
  `activity.detail.status` (team) vs `Goal.achievementStatus` (nature) are
  distinct fields by design.

---

## The teaching statement (keep — verified true)

> **The clinician who conflates "task done" with "goal met" is fighting the data
> model. The one who keeps them apart is using it as intended.**

This is not rhetoric — it is grounded in the standard: HL7 places *task/activity
status* and *goal achievement* in **separate fields on purpose**. Holding them
apart in your head matches the data as it actually is.

---

## Audit status

| Item | State |
|---|---|
| Example provenance documented | ✅ |
| Relationship codes verified | ✅ (RSON, SPRT, FLFS, COMP correct) |
| `REFR` overstatement | ⬜ correcting in docs (centerpiece done first) |
| Patient-as-performer | ⬜ flagged, fix proposed |
| templateIds | ⬜ flagged, verify-or-remove |
| Goal/objective consistency pass (WF1/3/4) | ⬜ flagged |
