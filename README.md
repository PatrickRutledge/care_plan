# Care-Plan Workflow — data-anchored business process map

An operational view of two care-plan workflows, built on **CDA R2.0 data atoms**
so the process **survives changes to the critical path**. The core idea:

> A care-plan task is not "the button the nurse clicks." It is a CDA `act`/
> `observation` in a specific `moodCode`, with a stable `id`, `participant`s, a
> `statusCode`, and `entryRelationship` links. The clicks are just how a human
> moves that data object through its states. Anchor the process map to the
> **data state machine** and the **information contract** instead of the screens,
> and re-sequencing the path can no longer break it.

## Design sequence (important)

The **clinically-correct process is defined first**, anchored to HL7 data; a
vendor (Athena) is mapped in **last**, as an implementation target — never the
source of the design. See [`clinical-process.md`](clinical-process.md) for the
governing care-planning cycle and the *intentional direction of services* that
every workflow realizes.

## What's here

| File | What it gives you |
|---|---|
| [`clinical-process.md`](clinical-process.md) | **The spine.** The clinical care-planning cycle, the temporal dependency graph, and the goal→service→evaluation intent. **Start here.** |
| [`process-map.md`](process-map.md) | BPMN-style swimlane spec for WF1 + WF2. Each step names its data object + state transition. |
| [`wf3-followup-evaluate.md`](wf3-followup-evaluate.md) | WF3 process map: monitor → evaluate → revise, with the revise-vs-close decision gateway. |
| [`wf4-assess-identify.md`](wf4-assess-identify.md) | WF4 process map: assess → identify problem → set goal. The start of the arc and the justification root. |
| [`wf5-referral-handoff.md`](wf5-referral-handoff.md) | WF5 process map: external referral with closed-loop reporting (CHI/community hand-off). |
| [`state-machine.md`](state-machine.md) | Mermaid diagrams of the `moodCode` / `statusCode` state machine (renders on GitHub). |
| [`careplan-example.xml`](careplan-example.xml) | A concrete, **schema-valid** CDA Care Plan instance: 1 goal, 1 internal task, 1 external task, 1 completion event. |
| [`careplan-example.rendered.html`](careplan-example.rendered.html) | The CDA instance run through the HL7 CDA stylesheet — the human-readable, attestable view. Open in a browser. |
| [`careplan-example.fhir.json`](careplan-example.fhir.json) | FHIR R4 mirror of the same plan (Patient, CareTeam, Goal, CarePlan, 2 Tasks) — the live-workflow representation. |
| [`cda-to-fhir-mapping.md`](cda-to-fhir-mapping.md) | The Rosetta-stone table: same data atoms, CDA column vs. FHIR column. |

## Validation & rendering (all reproducible)

| Check | Result |
|---|---|
| `careplan-example.xml` vs. `CDA_SDTC.xsd` (SDTC-extended CDA R2.0) | **SCHEMA-VALID** |
| Stylesheet render (`schema/normative/infrastructure/cda/CDA.xsl`) | **OK** → `careplan-example.rendered.html` |
| `careplan-example.fhir.json` well-formedness + cross-references | **OK** (10 resources, transaction Bundle) |

> The XML validates against the **SDTC-extended** schema (not base `CDA.xsl`)
> because it uses the `sdtc:inFulfillmentOf1` fulfillment link. The unzipped
> normative schemas live in `.normative-schema/` (extracted from
> `schema/normative/cda_r2_normativewebedition2010.zip`) — a working artifact you
> can delete and re-extract anytime.
>
> Note on the render: the CDA stylesheet shows the **narrative** (`section/text`)
> — the legally-attested human-readable layer — by design, not the structured
> entries. That separation of narrative vs. machine-readable entries is itself a
> CDA robustness feature.

## The two workflows

1. **Establish a new care plan with tasks for internal & external care team.**
   Goal = `observation moodCode="GOL"`. Each task = `act moodCode="RQO"`.
   Internal vs. external is *only* the `representedOrganization` behind the
   `performer` — the same atom, one differing field.

2. **Complete tasks in an existing care plan.**
   Completion = a *new* `act moodCode="EVN"`, `statusCode="completed"`, tied back
   to the order by `sdtc:inFulfillmentOf1`. Additive, not destructive — any
   system in any order can produce it and it still reconciles by `id`.

3. **Follow-up & evaluate (WF3).** Measure the outcome (`observation moodCode="EVN"`
   value), compare to the goal target, then **close** (goal met) or **re-direct**
   (new `act moodCode="RQO"` linked `REFR`→goal and `RSON`→outcome). The
   re-direct-vs-close branch is a data comparison, not a UI rule. See
   [`wf3-followup-evaluate.md`](wf3-followup-evaluate.md).

## The stability guarantee

| Layer | Examples | Changes when…? |
|---|---|---|
| **Volatile choreography** | screens, click order, swimlanes, vendor | the UX is redesigned — freely |
| **Stable contract** | `id`, `moodCode` lifecycle, `statusCode`, `inFulfillmentOf` / `entryRelationship` links, participant roles | the *clinical meaning* changes — rarely |

The path was never load-bearing; the **identity and state of the information
object** is. That is the decoupling that keeps the process from breaking.

## Provenance

The `moodCode`, `statusCode`, participation/relationship type codes, and the
`inFulfillmentOf` linkage used here were extracted directly from the schema in
this repository:

- `schema/extensions/SDTC/infrastructure/cda/POCD_MT000040_SDTC.xsd`
- `schema/extensions/SDTC/processable/coreschemas/voc.xsd`

> Note on layering: base CDA R2.0 (this repo) is a *document* standard — a
> persistent, attestable snapshot. For **live** task management (assign, track,
> notify), the same model maps cleanly onto the **C-CDA Care Plan** IG, or onto
> **FHIR `CarePlan` + `Task` + `CareTeam`** resources, which are designed as
> workflow state machines. Use CDA when the artifact must be a legal snapshot;
> use FHIR when you need the live workflow. The data atoms and state machine in
> these files translate to either.
