# Sequence diagram orchestration plan

## Purpose and agreed scope

This is the agreed handoff for the next agent to orchestrate creation and validation of the ambulance rostering sequence diagrams. The user approved the plan and specifically requires consistent boundary/controller design across all use cases.

Produce one standalone UML 2.5 sequence diagram for each of UC-01 to UC-14. Each diagram must cover its main success scenario and all documented alternative scenarios, and must be delivered as PlantUML source, PNG and SVG. The required diagram deliverables total 42 files.

The class table is substantially finalised. The class diagram is still being developed and must not block this work or override the class table. The resulting sequence interactions and proposed operations can inform the eventual class diagram.

This handoff was saved on 7 October 2026, Asia/Singapore. Planning and read-only source review have been completed; no new sequence diagrams have been created under this plan. Historical diagrams exist under `old/` and are not the current authority.

The next agent should execute the agreed plan when asked to continue it, without asking the user to approve the same scope or subagent arrangement again. The current request was only to save this handoff.

## Source authority

Workspace: `/Users/zhongjun88/Codex fun stuff/Intro to SE`.

| Source | Authority and use |
|---|---|
| [AGENTS.md](AGENTS.md) | Applicable project instructions, including UML 2.5 and sequence-diagram rules. Read it again when execution begins. |
| [Final Use Case.docx](Final%20Use%20Case.docx) | Behaviour: actors, prerequisites, processing order, validations, main and alternative scenarios, outcomes and use-case relationships. |
| [Main Class Table.docx](Main%20Class%20Table.docx) | Domain baseline: class names, responsibilities and attributes. |
| [Current domain class diagram](class-diagrams/main-class-table/domain-class-diagram.puml) | Supporting structural reference. It is unfinished and does not override the class table. |
| [Current model rules](class-diagrams/main-class-table/rules.md) | Supporting explanations and documented gaps. Distinguish source-backed business rules from provisional class-diagram modelling choices. |
| `Revised class tables.docx`, earlier diagrams and `old/` outputs | Historical/supporting material only; do not silently substitute these for the agreed baseline. |

Read the current source files rather than relying only on this summary or old extracted text. If a source changes during execution, identify which diagrams are affected and revalidate those diagrams.

Where a use case and the class table appear inconsistent, record the exact discrepancy and its impact. Do not silently rewrite either source. Refer to the project description or requirements register only when necessary to resolve a source-backed business rule, and record that supplementary evidence.

## Domain baseline

The class table contains these twelve domain classes:

`UserAccount`, `Staff`, `StaffAvailability`, `JobPreference`, `WeeklyRoster`, `Shift`, `RosterSlot`, `Assignment`, `Ambulance`, `AmbulanceAvailability`, `Station`, and `AuditEntry`.

Use these names and their recorded responsibilities and attributes consistently. The simplified class diagram intentionally omits operations. New message signatures are proposed detailed design derived from the class responsibilities, not pre-existing operations in the class table.

Adding boundary and controller classes is part of the agreed sequence-design work because the table covers domain entities. A genuine change to a domain class responsibility, attribute or relationship must be reported separately. Updating the class table or finalising the class diagram is not an automatic part of this deliverable.

## Shared boundary and controller design

Before independent use-case drafting, the orchestrator must establish one shared interaction contract. All author and validator subagents receive this contract and use it as the common design baseline.

The contract must contain:

- A participant catalogue with canonical class names, instance labels, UML roles, responsibilities and supported use cases.
- A boundary/controller responsibility map that identifies which participants are reused across related use cases.
- An operation register with owning class, operation name, parameters, result type or values, and calling use cases.
- Shared state and result conventions, especially assignment approval, prevention, cancellation, pending confirmation and confirmation.
- A consistent database mapping for illustrative SQL, including parameter placeholders and any proposed identifiers or tables absent from the domain model.
- Common visual conventions, including participant order, labels, fonts, fragment guards and message wrapping.

Design boundaries around coherent actor-facing interfaces and controllers around coherent workflow responsibilities. Reuse them where responsibilities align. Do not automatically introduce one new boundary and one new controller for every use case, and do not create an all-purpose controller with unrelated responsibilities.

Boundary/controller names have not yet been selected. The next orchestrator should make these routine design choices centrally, check them against the use cases, and test them in the UC-04/UC-13 pilot. They do not require another general approval round.

Once the pilot is validated, treat the catalogue as the shared baseline. Authors may propose additions or changes, but the orchestrator must reconcile them centrally before they appear in final diagrams. If a shared operation changes, update and revalidate every affected use case.

Actors send requests through boundaries. Controllers coordinate workflows. Entities carry domain behaviour consistent with their class-table responsibilities. Actor-facing success, warnings, errors and notifications also pass through boundaries. Distinguish the Staff actor from an entity lifeline such as `staff:Staff`.

## Diagram conventions

- Follow UML 2.5 notation and semantics and all current `AGENTS.md` instructions.
- Include relevant actor, boundary, controller and entity participants. Do not include all twelve entities in every diagram merely for completeness.
- Include the database explicitly wherever it is a secondary actor, showing requests and results. Thirteen use cases list Database; UC-13 lists no secondary actor.
- Use SQL on database-directed messages where available. Since no final SQL schema has been supplied, label proposed SQL and schema mappings as illustrative assumptions and use input placeholders.
- Disable automatic message numbering and omit manual numeric prefixes on message arrows. Use-case IDs and source-step references may appear in titles, comments or the coverage report.
- Use guarded `alt`, `opt` and `loop` fragments and interaction references where appropriate. Preserve the actual source order and outcomes.
- Trace every exit and retry. A rejection or cancellation must not fall through to later success-only validations, writes, audit-success events or confirmations. Shared continuation is valid only for every branch reaching it.
- Do not add speculative failure/retry/rollback scenarios or delivery guarantees as if the sources require them.
- Each `.puml` must be independently renderable. Keep essential definitions and styling in the file rather than depending on unavailable external includes.
- UC-13 must have its own diagram and be referenced from UC-04, UC-06 and UC-08. Its inclusion must not duplicate its checks or manager warning/decision in the base diagram.

## Complete use-case inventory

| ID | Exact use-case name | Required coverage emphasis |
|---|---|---|
| UC-01 | Manage User Account | Create, edit, deactivate, incomplete input, duplicate staff ID, retained records, audit and manager notification about existing assignments. |
| UC-02 | Manage Availability | Five-week planning period, individual 12-hour shifts, Wednesday cut-off per affected week, on-time and late submissions, persistence, audit, manager notification and unchanged published assignments. |
| UC-03 | Set Job Preference | Shift and station preferences for a week, including blank/no preference. |
| UC-04 | Build Weekly Roster | Retrieve/create draft, eligible ambulance selection, candidate display, UC-13, save/audit, repeated allocation, removal/replacement, no eligible ambulance, rejected proposal, published-roster redirect and unavailable ambulance warning. |
| UC-05 | Publish Weekly Roster | Readiness of every shift, successful publication/audit/visibility, or prevention with affected shifts, missing roles and ambulance shortfall. |
| UC-06 | Amend Published Roster | Direct reassignment, removal with reason, no candidate, UC-13 prevention/cancellation, audit, affected-staff notifications, standby extension and ambulance unavailability while remaining published. |
| UC-07 | Reject Assignment | Discussion warning, confirmation/reason, cancellation, missing reason, rejection/audit, manager notification and resulting shortfall. |
| UC-08 | Find Standby Replacement | Eligibility filtering, candidate comparison, withdrawal of prior request, no candidate, UC-13, pending persistence/audit, confirmation request and vacancy remaining unfilled. |
| UC-09 | View Own Roster | Own weekly assignments and crew details, no assignments, calendar-month hours and own monthly assignment view. |
| UC-10 | Review Staff Workload | Weekly hours, lowest-workload list including cutoff ties, over-40-hour highlighting and calendar-month hours. |
| UC-11 | View Ambulance Utilisation | Individual scheduled utilisation, fleet utilisation by shift, missing roles and staff required, or all slots fully staffed. |
| UC-12 | Manage Ambulance Record | Add/edit/deactivate/reactivate, shift unavailability, audit, allocation exclusion, affected assignments and routing to draft/published amendment. |
| UC-13 | Check Assignment Rule | Qualification, staff conflict, ambulance conflict, rest, weekly-hours warning, allowed/prevented/cancelled results and manager decision through a boundary. |
| UC-14 | Confirm Replacement Assignment | Display pending request, confirmation, decline, already-taken error, persistence/audit, vacancy state and manager notification. |

This table is a checklist, not a replacement for reading every scenario and postcondition in the DOCX.

## Execution order

1. Extract and inspect the current use cases and class table. Establish a source snapshot, the shared interaction contract and an initial issue register.
2. Pilot UC-13 and UC-04 together. Resolve their calling relationship, shared operation signatures, boundary warning interaction and validation outcomes. Validate the pilot before propagating its conventions.
3. Draft and validate foundational use cases: UC-01, UC-02, UC-03 and UC-12.
4. Draft and validate publication and amendment: UC-05 and UC-06.
5. Draft and validate rejection and replacement: UC-07, UC-08 and UC-14.
6. Draft and validate views/reporting: UC-09, UC-10 and UC-11.
7. Perform a complete cross-diagram consistency pass, render final outputs and verify the delivery inventory.

Independent use cases within these stages may progress concurrently, subject to available agent capacity and a stable shared contract.

## Subagent orchestration

The user explicitly requested an Astra xhigh author alongside a validation subagent for each use case.

For every UC, assign an author using model `gpt-6-astra` with reasoning effort `xhigh`, plus a separate independent validator. No specific validator model was requested; use the available inherited/default model unless the user changes this preference.

When the collaboration tool requires it, use `fork_turns: "none"` or a supported bounded history value for the explicit author model override. Supply a self-contained task brief instead of relying on inherited conversation. Use subagents within the current task, not new user-owned chats.

At planning time the environment allowed four active agents including the orchestrator. Respect the capacity available at execution time. Schedule small batches; do not attempt to run all fourteen author/validator pairs simultaneously. A validator may review a completed draft while an author works on the next independent use case.

The orchestrator owns the shared contract and final integration. Give each author exclusive ownership of its assigned diagram files. Validators report findings rather than simultaneously editing the author's files. Shared-directory edits are visible to all agents, so changes to the central contract must be coordinated.

Each author brief must include:

- The exact assigned UC and current source paths or a verified complete excerpt, including all alternatives and postconditions.
- Applicable `AGENTS.md` instructions and the current shared contract.
- Related use-case contracts and dependencies.
- Exact output paths and scope of file ownership.
- Known source gaps relevant to that UC.
- A requirement to provide a source-step coverage map and identify proposed operations, participant additions and SQL assumptions.
- A requirement to return the draft for independent review and revise every substantiated finding.

Each validator brief must include the same authoritative sources and contract, plus the authored draft. The validator must independently read the original scenario rather than relying solely on the author's explanation or coverage map.

For each UC, repeat author draft, validator review, author correction and validator recheck until PASS. Keep unresolved source-policy questions separate from diagram defects. A diagram dependent on an unresolved material policy must not be reported as unconditionally final.

## Behavioural checks requiring particular care

- UC-13 is invoked by a base controller, never directly initiated by the Manager. Its manager warning and response still traverse the relevant boundary. Rule context can be loaded by the base use case and passed in; do not invent a required database secondary actor for UC-13.
- Preserve UC-13's order: qualification, staff same-shift conflict, ambulance same-shift conflict, minimum rest, then weekly hours. Blocking failures stop subsequent checks. Exceeding 40 hours warns and permits confirmation or cancellation; it is not itself a blocking rule. Base writes proceed only on an allowed result.
- UC-02 must persist late availability and the affected late flags before audit/notification, as required by its postconditions. Record the omitted save in alternative 5a as a source reconciliation.
- UC-04's published-roster branch ends or hands off to UC-06; it must not continue into draft allocation. Invalid selections and rejected proposals must follow their stated retry paths.
- UC-05 checks every shift in the week, including shifts with no slots. Missing coverage cannot be hidden by iterating only over existing slots.
- UC-06 remains published during amendment. If no candidate leads to removing the current assignee, route through the required reason, persistence, audit and notification before proceeding to UC-08.
- UC-07 cancellation preserves the assignment. Missing reason prevents submission. Shortfall handling follows an actual rejection, not the cancellation branch.
- UC-08 withdrawal removes the previous pending assignment and notifies that staff member before candidate selection continues. A new pending request does not fill the vacancy. Failed/cancelled UC-13 results create neither an assignment nor a confirmation request.
- UC-14's already-taken branch must not reach confirmation writes or success-only audit, notification and visibility outcomes. If using a conditional/atomic persistence operation to enforce this, label it as proposed design. Both confirmation and decline notify the manager according to the full scenario text.
- UC-01 includes a manager notification in its deactivation alternative despite listing only Database in its secondary-actor field. Include the required notification path through a boundary.
- UC-09 preserves access to the staff member's own data and assignment crew; do not expose a full team roster through the personal view.
- UC-10 includes all ties at the third-person workload cutoff. Do not confuse this rule with the separate up-to-three candidate display in allocation use cases.
- Do not count one duty twice merely because a qualified staff member fulfils two roles. Distinguish missing crew-role positions from additional people required.

## Source gaps and proposed design assumptions

Maintain `model-issues.md` with source location, issue, affected UCs, proposed treatment and resolution status.

Known items to recheck against current sources:

| Item | Treatment |
|---|---|
| UC-02 late alternative omits explicit save | Reconcile with required stored-availability postconditions; document the added persistence interaction. |
| Pending assignments and workload/rest | Their contribution remains unspecified. Do not silently choose a policy when implementing concrete calculations. Seek focused clarification if necessary while continuing independent work. |
| Candidate location for a date | Do not equate weekly station preference with actual dated location or invent a permanent station relationship. Record the data-source gap. |
| Incoming/outgoing crew derivation | Check the source evidence; retain any unresolved derivation as a documented detail rather than inventing roster relationships. |
| Overnight duty across a calendar-month boundary | Do not silently assume which month receives the hours. |
| Exact scheduled in-service hours and zero active fleet | Preserve the documented utilisation formulas and 11-hour cap; record unspecified calculation inputs or undefined denominator cases. |
| UC-04 unavailable-ambulance continuation | The alternative states a warning but does not fully define continuation. Do not invent a successful allocation outcome. |
| Boundary/controller classes and operation signatures | Proposed detailed design, maintained in the shared contract and validated against domain responsibilities. |
| SQL schema and confirmation concurrency mechanism | Illustrative/proposed mappings and implementation choices, not established source facts. |
| Relationships in the unfinished class diagram | Check against the class table and use cases. Record suggested reconciliation without automatically editing the diagram. |

The user has already approved routine modelling work and subagent orchestration. Ask only focused questions about decisions that materially change required behaviour; do not ask for a blanket approval of the same plan again.

## Validation and completion gates

Each use case must pass the following checks:

1. Source coverage: every main/alternative step within scope maps to a message, guard, loop, interaction reference or explicit constraint.
2. Ordering and branch safety: all source-order dependencies, rejection, cancellation, missing-input, no-candidate and retry paths are correct, with no invalid continuation.
3. Participant consistency: boundaries/controllers match the shared catalogue, domain lifelines match the class table, and actors never bypass boundaries.
4. Operation consistency: owning lifeline, operation name, parameters and results match the shared register, including reused operations across UCs.
5. Persistence and state: database requests/results, audit requirements, confirmation states, publication states and notifications match the documented outcomes.
6. UML and source quality: correct fragment meaning, readable guards and lifeline types, no message numbering, and valid standalone PlantUML.
7. Visual quality: both PNG and SVG render correctly, without clipping, unreadable messages, missing content or ambiguous fragment nesting.

Record PASS/FIX findings per UC in `validation-report.md`, including resolved findings and any remaining source assumptions. After individual passes, compare all fourteen diagrams for shared participant, operation, state, SQL mapping and included-use-case consistency.

## Rendering and file delivery

Use a new batch directory under `sequence-diagrams/`, named using the current Asia/Singapore date and time. Preserve historical work.

Use these canonical filename stems:

```text
uc-01-manage-user-account
uc-02-manage-availability
uc-03-set-job-preference
uc-04-build-weekly-roster
uc-05-publish-weekly-roster
uc-06-amend-published-roster
uc-07-reject-assignment
uc-08-find-standby-replacement
uc-09-view-own-roster
uc-10-review-staff-workload
uc-11-view-ambulance-utilisation
uc-12-manage-ambulance-record
uc-13-check-assignment-rule
uc-14-confirm-replacement-assignment
```

Every stem must have matching `.puml`, `.png` and `.svg` files. Also deliver:

- `README.md`: index linking each use case's three outputs, source baseline and rendering instructions.
- `interaction-contract.md`: shared participant catalogue, boundary/controller responsibility map, operation register and illustrative database mappings.
- `validation-report.md`: source-step coverage, individual review outcomes and cross-diagram review result.
- `model-issues.md`: reconciliations, assumptions, unresolved policies and suggested class-diagram follow-up.

Local PlantUML and Java were available during planning. `plantuml -version` reported version 1.2026.8 and successful installation checks. Recheck availability at execution time. The observed default image size limit was 4096; ensure long diagrams are exported completely rather than accepting truncation or shrinking text until unreadable.

Compile every source, export both formats, and visually inspect every final diagram. Re-render after any source change and verify the PNG/SVG correspond to the final `.puml`. Successful compilation alone does not establish semantic correctness or readability.

For DOCX inspection, use the available documents skill and resolve bundled runtimes with `load_workspace_dependencies` as required by the current environment. During planning the bundled Python executable was `/Users/zhongjun88/.cache/codex-runtimes/codex-primary-runtime/dependencies/python/bin/python3`; rediscover it if necessary. Do not edit or re-export the source Word documents merely to extract their contents.

Completion means all fourteen use cases have their three matching files, every diagram has passed independent validation and visual inspection, the shared design is consistent, and any material unresolved issue is clearly disclosed rather than represented as resolved.
