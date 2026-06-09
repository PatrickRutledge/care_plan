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
   *Status: RESOLVED for all five workflows.* Re-anchored to conformant C-CDA
   using **verified** templateIds (confirmed against HL7's published C-CDA
   template registry + Goal Observation / Outcome Observation examples):
   Care Plan `…1.15`, Health Concerns `…2.58`, Goals `…2.60`, Interventions
   `…21.2.3`, Health Status Evaluations & Outcomes `…2.61`, Health Concern Act
   `…4.132`, Goal Observation `…4.121`, Outcome Observation `…4.144`,
   Intervention Act `…4.131`, Planned Intervention `…4.146`, Entry Reference
   `…4.122`. Files: `careplan-{establish,complete,followup,assess,referral}.ccda.xml`,
   all schema-valid + rendered. WF3 uses the purpose-built `GEVL` goal-evaluation
   link for outcomes. *Note:* validation is base-CDA XSD; full C-CDA conformance
   also needs Schematron (a separate step not run here).

3. **Goal-as-measurable-target in WF1 / WF3 / WF4.** Those earlier examples set
   the goal as an A1c value/threshold (`code` = A1c, `value` = 7.0%) — the C-CDA
   Goal Observation pattern. It is **valid HL7**, but it encodes the *objective*,
   and the narrative called it "the goal," reproducing the very goal/objective
   conflation we later corrected. *Proposed fix:* a consistency pass so the
   goal/objective language is uniform across all workflows.

---

## 4. NOTES (correct, but worth stating plainly)

- `sdtc:inFulfillmentOf1` is **documented in the HL7 CDA-core-sd build
  (`InFulfillmentOf1`) and used in IGs such as QRDA Category I R3** — but it is a
  *niche* SDTC class, **not** part of the commonly-cited SDTC extension set
  (`sdtc:raceCode`, `sdtc:deceasedInd`, …). Calling it simply "a real SDTC
  extension" overstated its profile. Used correctly (`FLFS`) in the base-CDA
  example; the **conformant `.ccda` set uses the C-CDA Entry Reference idiom
  instead**, so nothing load-bearing depends on it. (Softened 2026-06-09 per
  external audit.)
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

## 5. External cross-check (GitHub Copilot + Grok, 2026-06-09)

Two independent AI audits of the repo were run. Both landed ~70% "directly
supported by HL7," with the same three red flags. Each finding was **re-verified
against the FHIR R4/R5 spec** before acting — and verification showed the external
fixes themselves were partly off:

| External finding | Verified verdict & action |
|---|---|
| `Task.reasonReference → Goal` mismatch (`cda-to-fhir-mapping.md`) | **Confirmed.** But the proposed fix (`activity.detail.goal`) is wrong for our Task-based plans: R4 constraint **cpl-3** makes `activity.detail` mutually exclusive with `activity.reference → Task`, and R5 **deleted** `activity.detail`. **Correct fix applied:** goal linkage is **plan-level `CarePlan.goal`**; no per-task goal field exists. Instances + docs updated. |
| "CDA REFR stronger than FHIR" (`wf3`) "backwards" | **Partly.** Backwards only for *inline* R4 activities. For Task-based plans (ours), FHIR has **no** per-task goal link (R4 *and* R5), so CDA's inline REFR is more *granular* (generic, but on the act). Reworded to the precise, version-aware statement rather than a blunt "stronger." |
| moodCode `PRP→INT→RQO→EVN` as one act's lifecycle (`state-machine`) | **Confirmed.** moodCode is a fixed attribute; these are *separate linked acts*. Diagram relabeled as a teaching abstraction. |
| `sdtc:inFulfillmentOf1` "real SDTC extension" overstated | **Confirmed-minor.** Softened (CDA-core-sd / QRDA Cat I; niche). `.ccda` set uses Entry Reference instead. |
| `Task.basedOn → CarePlan`, `ServiceRequest.supportingInfo → Goal` | **Fair.** Switched to idiomatic `CarePlan.activity.reference → Task` and plan-level `CarePlan.goal`. |

**What the cross-check confirmed about the process, not just the content:** an AI
audit can be confidently wrong the same way the original author was about `REFR`.
Verifying each claim against the spec (here, to the cpl-3 constraint and the R4→R5
deletion) produced a *more* correct result than any single audit — including
catching that both external tools oversimplified the goal-linkage fix.

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
| Relationship codes verified | ✅ (RSON, SPRT, FLFS, COMP, GEVL correct) |
| `REFR` overstatement | ✅ swept — `cda-to-fhir-mapping`, `wf3`, `wf5`, `discovery-summary`, `state-machine` corrected |
| FHIR per-service→goal linkage | ✅ corrected to plan-level `CarePlan.goal` (R4 cpl-3 + R5 deletion verified); instances updated |
| `sdtc:inFulfillmentOf1` profile | ✅ softened; `.ccda` set uses Entry Reference |
| Patient-as-performer | ⬜ flagged, fix proposed (still open) |
| templateIds | ✅ resolved — all five workflows re-anchored to verified C-CDA templateIds |
| Goal/objective consistency pass (WF1/3/4) | ✅ uniform C-CDA Goal Observation form |
| External cross-check reconciled | ✅ Copilot + Grok findings verified, applied, or refuted |
