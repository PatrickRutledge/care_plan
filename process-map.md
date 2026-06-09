# Care-Plan Process Map (operational, data-anchored)

A BPMN-style specification where every step names the **CDA data object** it
acts on and the **state transition** it causes. Build your swimlane diagram from
these tables. The "Data object" and "State in → State out" columns are the
**stable contract**; the "Lane (who clicks)" column is the **volatile
choreography** you can redesign at will.

Legend for state notation: `moodCode / statusCode`.

---

## Workflow 1 — Establish a new care plan with tasks

| # | Step (business action) | Lane (who clicks) — *volatile* | Data object — *stable* | State in → State out | Key fields written |
|---|---|---|---|---|---|
| 1 | Open new plan | Care coordinator | `ClinicalDocument` | — → draft | `id`, `code=18776-5`, `recordTarget`, `effectiveTime` |
| 2 | State the goal | Provider | `observation moodCode=GOL` | — → `GOL / active` | `id=G1`, `code`, target `value`, `effectiveTime` window |
| 3 | Create internal task | Provider | `act moodCode=RQO` | — → `RQO / new` | `id=T1`, `code`, `text`, due `effectiveTime`, `priorityCode` |
| 4 | Assign internal owner | Provider | `performer typeCode=PRF` on T1 | `RQO / new` → `RQO / active` | `assignedEntity`, `representedOrganization=OUR-ORG` |
| 5 | Create external task | Coordinator | `act moodCode=RQO` | — → `RQO / new` | `id=T2`, `code`, `text`, `effectiveTime`, `priorityCode` |
| 6 | Assign external owner | Coordinator | `performer` + `participant typeCode=REFT/CALLBCK` on T2 | `RQO / new` → `RQO / active` | `representedOrganization=OTHER-ORG`, callback contact |
| 7 | Link tasks to goal | System | `entryRelationship typeCode=REFR` | (no status change) | T1→G1, T2→G1 |
| 8 | Sign / attest plan | Provider | `legalAuthenticator` | draft → attested | `time`, `signatureCode=S`, `assignedEntity` |

**Robustness note for steps 3–6:** internal and external tasks are the *same*
data object. Only the `representedOrganization` (and `id` root namespace) differ.
You may merge, split, or reorder these steps in the UI without touching the model.

---

## Workflow 2 — Complete tasks in an existing care plan

| # | Step (business action) | Lane (who clicks) — *volatile* | Data object — *stable* | State in → State out | Key fields written |
|---|---|---|---|---|---|
| 1 | Pick up the task | Assignee (internal or external) | `act moodCode=RQO id=T1` | `RQO / active` | (read `id`, `code`, due window) |
| 2 | Pause if blocked | Assignee | same act, `statusCode` | `RQO / active` → `RQO / suspended` | `statusCode=suspended` (+ optional `RSON` reason link) |
| 3 | Resume | Assignee | same act, `statusCode` | `RQO / suspended` → `RQO / active` | `statusCode=active` |
| 4 | Perform & record completion | Assignee | **new** `act moodCode=EVN` | — → `EVN / completed` | `id=T1-DONE`, actual `effectiveTime`, `performer` |
| 5 | Reconcile to the order | System | `sdtc:inFulfillmentOf1 (FLFS)` | links EVN → original `RQO id=T1` | `actReference` to T1 |
| 6 | Roll progress up to goal | System | `entryRelationship typeCode=REFR` | (no status change) | T1-DONE → G1 |
| 7 | Assess goal outcome | Provider | `observation moodCode=EVN` | — → `EVN / completed` | measured `value` vs. goal target |
| 8 | Close or revise plan | Provider | `relatedDocument typeCode=RPLC` (if revised) | attested → superseded | new version `id`, replaces prior |

**Robustness note for steps 4–5:** completion is *additive* — a new `EVN` act
that points back to the order by `id`. It does not mutate the original order
in place. Any system, in any order, on any screen can produce this event and it
will still reconcile. That is the property that makes the path non-load-bearing.

**Note on `REFR` (the "Link tasks to goal" / "Roll progress up to goal" rows):**
in CDA, `entryRelationship typeCode="REFR"` means only **"refers to"** — a generic
reference, *not* a built-in "serves the goal" semantic. The goal-direction intent
is a convention layered on it (explicit in FHIR via plan-level `CarePlan.goal`;
FHIR has no per-task goal field for Task-based plans). Where an act *evaluates* a
goal (an outcome), the conformant `.ccda` files use the purpose-built `GEVL`
("evaluates goal") code instead. See [`hl7-conformance-audit.md`](hl7-conformance-audit.md).

---

## Other recognized workflows (same atoms, different configuration)

| Workflow | Data pattern | Distinguishing field(s) |
|---|---|---|
| Referral / external hand-off | `act moodCode=RQO`, performer in other org | `participant typeCode=REFT` / `REFB` |
| Appointment scheduling | `encounter moodCode=ARQ → APT → EVN` | `moodCode` progression |
| Conditional / triggered task | `act` + `precondition` | `precondition` criterion |
| Plan revision / versioning | new doc, `relatedDocument typeCode=RPLC` | `RPLC` link |
| Review & sign-off | `authenticator` / `legalAuthenticator` | `signatureCode` |
| Reason / justification | `entryRelationship typeCode=RSON` → diagnosis | `RSON` link |
| Notify / cc a party | `participant typeCode=NOT` / `IRCP` / `PRCP` | participant `typeCode` |

---

## How to use this with your BPMN tool

1. Put the **Data object** column values on the BPMN canvas as *data objects*
   (the document-with-folded-corner shape), one per row.
2. Put the **Lane** column as your *swimlanes* — these are the only things you
   redraw when the process changes.
3. Put the **State in → State out** column on the *sequence-flow gateways*.
4. When someone says "the workflow changed," confirm: did a **Data object** or a
   **State transition** change (real change — update the contract), or only a
   **Lane / step order** (cosmetic — the model is unaffected)? That triage
   question is the payoff of anchoring to the data standard.
