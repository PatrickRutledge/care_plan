# HL7 Standards Layers — progression and abstraction

Two questions get answered together here, because they are the same picture read
two ways:

1. **Progression** — how the HL7 standard *evolved over time*.
2. **Abstraction** — how it goes from *high-level (generic) to sub-level
   (specific)*.

The strategic decision for this repo: **build on the most extensive / most
current HL7 spec**, so we sit on the *edge of innovation* rather than perpetually
catching up to a standard that has already moved on.

---

## 1. Progression (the standard evolved)

| Era | Standard | What it is | Care-plan capability |
|---|---|---|---|
| 2005 (repo edition 2010) | **CDA R2.0** | RIM-based clinical *document* grammar | none defined — generic acts/observations only |
| 2012–2015 | **C-CDA** (R1.1 → R2.0 → **R2.1**, the `2015-08-01` templates) | a constraint/IG on CDA R2.0 — agreed templates | the **Care Plan document** (4 sections) — US-mandated |
| 2014–2019 | **FHIR** DSTU1 → DSTU2 → STU3 → **R4** | resource + REST API paradigm (new product family) | `CarePlan` + `Goal` + `Task`/`ServiceRequest` + `CareTeam` |
| 2023 | **FHIR R5** | current published release | refined CarePlan/Goal; more normative content |
| **2026 (in ballot)** | **FHIR R6** + **Clinical Reasoning** | `PlanDefinition` / `ActivityDefinition` targeted **Normative** | **computable, dynamic, reusable** care-plan definitions |

The arc: *generic document* → *agreed document template* → *live resources* →
*computable/dynamic definitions*. Each step did not replace the prior so much as
add capability on top.

---

## 2. Abstraction (high-level → sub-level)

The same material, stacked from most generic to most specific. Two stacks, one
per paradigm:

**CDA / document paradigm**
```
HIGH  CDA R2.0            the grammar — can express anything, defines no care plan
  │
SUB   C-CDA Care Plan     the agreed template — Health Concerns / Goals /
                          Interventions / Outcomes; fixed codes & cardinalities
  │
INST  a signed instance   one patient's care-plan document (a snapshot)
```

**FHIR / resource paradigm**
```
HIGH  PlanDefinition /    the DEFINITIONAL layer — a reusable, computable plan
      ActivityDefinition  "template" (the diabetes CCM protocol itself)
  │
SUB   CarePlan / Goal /   the REQUEST/INSTANCE layer — this patient's plan,
      Task / ServiceRequest  goals, and tasks, instantiated from the definition
  │
INST  live resources      updated continuously via the API
```

> **Version watch (verified 2026-06-09).** FHIR is still moving, and a care-plan
> detail changed between releases: **R4** `CarePlan.activity` could carry an inline
> `activity.detail.goal` (an explicit per-activity→goal link); **R5 deleted
> `activity.detail` entirely** (replaced by `plannedActivityReference`), so in R5
> goal linkage is **plan-level only** (`CarePlan.goal`). Practical effect: there is
> **no first-class per-task→goal link** when activities are Task/ServiceRequest
> resources, in R4 *or* R5. Building on "the most extensive spec" means tracking
> these shifts — the adapter must reconstruct per-service→goal intent itself, and
> not assume `activity.detail.goal` is available.

The crucial FHIR insight: **`PlanDefinition` → `CarePlan` is "template →
instance."** You author the care-plan logic *once* as a `PlanDefinition` (with
`ActivityDefinition`s for each action), host it on a registry, and *instantiate*
it into a patient-specific `CarePlan`/`Task` set. Dynamic values can be computed
via expressions/`Library`. **That is exactly "dynamic care plans hosted on a
registry."** It is not a future we have to invent — it is the frontier HL7 is
balloting into Normative now.

---

## 3. The three (really four) layers, side by side

| Layer | Paradigm | Role | Our use |
|---|---|---|---|
| **CDA R2.0** | document | the grammar / substrate | this repo (`cdaq-core-2.0`) — schema validation only |
| **C-CDA** | document | the *conformant* care-plan document (snapshot) | what our CDA examples should anchor to (the real templateIds) |
| **FHIR CarePlan/Goal/Task** | resource | the *live* care plan (API) | our "FHIR mirror" files |
| **FHIR PlanDefinition/ActivityDefinition** | resource (Clinical Reasoning) | the *computable, reusable* plan definition | the **edge** — reusable/dynamic plans on a registry |

> Base CDA is the alphabet. C-CDA is the standardized form. FHIR CarePlan is that
> form as a live API. PlanDefinition is the *blueprint that stamps out the form
> automatically* — the most extensive of the four.

---

## 4. Strategic decision for this repo

**Anchor to the most extensive / most current spec, so we innovate ahead instead
of catching up.** Concretely:

- **Substrate:** keep CDA R2.0 only as the base grammar (it is the repo we cloned).
- **Conformant document side:** anchor CDA examples to **C-CDA** (real templateIds,
  the four sections, the actual entry templates) — resolves the audit's
  templateId problem and makes each example a *conformant* care plan, not just
  schema-valid base CDA.
- **Live side:** keep the **FHIR R5** mirrors (R5 is current; design so R6 is a
  small step).
- **The edge:** introduce **`PlanDefinition` / `ActivityDefinition`** (FHIR
  Clinical Reasoning) as the reusable, computable definition layer — the
  technical form of *dynamic care plans on a registry*. This is where being
  "early" pays off: the resources are balloting toward Normative in **R6 (2026)**,
  so building here now puts us at the front, not behind.

The principle that has held all along still holds: anchor to the data standard,
and the process survives change. Choosing the **most extensive** layer of that
standard just means the change we survive includes the *next* standard, not only
the next UI.

Sources: [FHIR versions / R6 ballot](https://build.fhir.org/versions.html),
[FHIR Clinical Reasoning module](https://www.hl7.org/fhir/clinicalreasoning-module.html),
[PlanDefinition (R5)](http://hl7.org/fhir/plandefinition.html),
[ActivityDefinition (R5)](http://hl7.org/fhir/activitydefinition.html),
[C-CDA Care Plan template](https://hl7.org/ccdasearch/templates/2.16.840.1.113883.10.20.22.1.15.html).
