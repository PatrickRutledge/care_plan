# Discovery Phase — Summary (end of phase 1)

This closes the **HL7-material discovery phase**. The goal was to specify
care-plan workflows in terms of HL7 data types — so the process is robust to UI
("click") changes and to the eventual move into a vendor system (Athena) — and to
do it grounded strictly in the CDA/FHIR material, before any comparison to outside
frameworks. That is now done.

## What was established

1. **A workflow is a data state machine, not a screen flow.** Every care-plan
   step is a CDA `act`/`observation` in a specific `moodCode`, with a stable `id`,
   `participant`s, a `statusCode`, and relationship links. The clicks are just how
   a human moves that object through its states. (`clinical-process.md`)

2. **The clinical care-planning cycle, fully mapped to data.** All 8 phases
   (assess → identify → set goal → direct services → implement → monitor →
   evaluate → revise) are realized by a validated workflow, each phase carried by
   a named HL7 atom.

3. **Five workflows, each with three artifacts** — a schema-valid CDA instance, a
   FHIR R4 mirror, and a process map:

   | WF | Phase(s) | Closes on |
   |----|----------|-----------|
   | WF4 | assess → identify problem | the justification root |
   | WF1 | set goal → direct services | goal + internal/external tasks |
   | WF2 | implement → complete | fulfillment by `id` |
   | WF3 | monitor → evaluate → revise | value-vs-goal decision |
   | WF5 | external referral (closed-loop) | cross-org `inFulfillmentOf` |

4. **The intentional direction of services is encoded, not narrated.** Goals
   direct services; every service links back to the goal it serves (`REFR`) and
   the reason it exists (`RSON`); evaluation measures services against goal
   targets. A service with no goal link is *visibly* unjustified — an auditable,
   queryable property rather than a chart-review opinion.

5. **The temporal order cannot break, because it is a data dependency.** You
   cannot evaluate without a goal, justify a service without a goal, or set a goal
   without an evidenced problem. "Establish → follow-up → complete" is enforced by
   the information graph, not by step order in a UI.

## What was proven about robustness

- **Internal vs. external is one field.** A multi-party external referral (WF5) is
  the same object as an in-house task (WF1) with a different
  `representedOrganization`. A workflow can span organizational boundaries with no
  new process logic.
- **Even the decision gateways are data.** "Did we meet the goal? close or
  re-direct?" (WF3) is a `value` vs. `target` comparison — so the clinical
  judgment survives a UI change too.
- **Every example validates.** All five CDA instances are schema-valid against the
  SDTC-extended CDA R2.0 schema; all five FHIR bundles are well-formed with intact
  cross-references; all five render through the HL7 stylesheet.

## Open questions to carry into the next phase

1. **The CDA↔FHIR adapter gap (recurring).** CDA carries an inline
   `entryRelationship REFR` from the act to the goal — *granular* (which act →
   which goal) but semantically generic ("refers to"). FHIR has **no first-class
   per-service goal link** for Task/ServiceRequest-based plans (the explicit
   `CarePlan.activity.detail.goal` is inline-only, R4-only, removed in R5); goal
   linkage is **plan-level** (`CarePlan.goal`). So the two differ in *where* the
   link lives, not simply in strength. Conversely FHIR's `Goal.achievementStatus`
   is *more expressive* than CDA's goal `statusCode`. The adapter must reconstruct
   "which goal does this service serve?" and normalize achievement state across
   both. (Verified 2026-06-09 against FHIR R4/R5; documented in WF3, WF5,
   `cda-to-fhir-mapping.md`.)
2. **Static vs. dynamic care plans.** These artifacts model the care plan as a
   persistent, attestable *snapshot* (CDA's nature). A future direction is a
   *dynamic* care plan with a lifetime, hosted on a registry — to be examined in
   the comparison phase.
3. **Vendor binding (Athena).** Deferred by design. Athena's care-plan fields
   exist but are not yet connected to an intentional model, so binding is a final
   step: map each data atom to an Athena API field through the adapter, FHIR-first
   with the proprietary API as the write fallback. (See API discovery notes.)

## Deliberately deferred (the next phase)

Comparison. Reconciling this data-anchored model against **actual clinic
workflow** and external frameworks (e.g. **CMS**, **AHRQ**, and program models
such as CHI / PIN). That comparison is a separate effort, kept out of the
discovery so the model stands on its own first and gives the comparison something
solid to measure against.

---

*Status: discovery phase complete. Next: clinic-workflow comparison.*
