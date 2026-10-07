# UC-04 author review

Status: draft ready for independent validation. Scope is the complete UC-04 Build Weekly Roster interaction: main steps 1–10, alternatives 1a, 4a, 5a, 8a and 9a, and all postconditions. The shared UC-13 invocation is an interaction use, not a second rendering of its rule checks.

## Source verification

Independently opened `Final Use Case.docx` and `Main Class Table.docx` using bundled Python and python-docx in read-only mode. Extracted the original UC-04 and UC-13 tables directly and read every domain-class row. They agree with the batch evidence. Read current `AGENTS.md`, `SEQUENCE_DIAGRAM_ORCHESTRATION_PLAN.md`, the batch contract and issue register. No source DOCX was edited. The earlier diagram was inspected as a historical design reference and its authoring captions, issue notes, legends, generic database write replies and activation arrangement were not retained.

## Source-step coverage

| Source | Diagram realization |
|---|---|
| Preconditions and trigger | Opening behaviour note records logged-in Manager and beginning/continuing planning. `openAllocation(week)` initiates the interaction. The selected week is constrained to the five-week planning period. |
| Main 1 | Manager requests `openAllocation(week)` through RosterBoundary; boundary invokes `openWeek(week)`. |
| Main 2 | Explicit WeeklyRoster SELECT; if absent, draft INSERT and creation audit; WeekPlanningView SELECT; each existing draft slot derives its staffing status. The response exposes slots, ambulance/staff details, staff availability and manpower needs. |
| Main 3 | Each outer allocation iteration starts with `selectSlot(shiftId, slotSelection)`. |
| Main 4 | SQL joins AmbulanceAvailability and filters active, available, and no other same-shift slot; eligible records return to Control, boundary and Manager. The selected stored slot is excluded from the conflict subquery. |
| Main 5 | `chooseRoleOrReplacement(ambulanceId, roles, replacedAssignmentId?)`; the role choice supports either an unfilled role or replacement. |
| Main 6 | CandidateOption SELECT is capped at three records. Staff derives assigned hours from retrieved assignment records and supplied policy. The displayed candidates include availability, assigned hours, job preference and dated location. |
| Main 7 | `selectCandidate(staffId)` is a new actor request after the candidate-list request has completed. |
| Main 8 | Control retrieves StaffRuleContext and same-shift ambulance slots, then performs one UC-13 `ref`, including inputs, result and `baseControl = roster:RosterControl`. No qualification, conflict, rest, hours or hours-decision check is duplicated here. |
| Main 9 | Only `result is allowed` can enter assignment preparation and persistence. If the ambulance has not become unavailable, the chosen slot is inserted or its ambulance updated as needed, then the confirmed assignment is inserted or replaced. Slot staffing is derived after persisted change; audit is prepared and explicitly inserted before saved outcome is returned. |
| Main 10 | Outer allocation loop repeats selection and allocation. With no invalid-ambulance warning, Manager invokes `saveDraft`; the status UPDATE explicitly acknowledges `roster retained as draft`; the save event is audited; Manager receives saved draft and manpower outcome. |
| 1a, 1a1–1a3 | The same initial roster queries retrieve published data. Published response names UC-06 Amend Published Roster. All subsequent interaction is inside `opt roster status is draft`, so the published handoff ends UC-04. No UC-06 `ref` claims the separate use case automatically executes here. |
| 4a, 4a1–4a2 | Empty eligible records produce the no-ambulance response. The entire removal/fill/replacement continuation is guarded by nonempty records; another outer iteration permits a different shift/slot. There is no candidate-allocation fallthrough for the invalid selection. |
| 5a, 5a1–5a2 | The remove operand selects an existing assignment, prepares domain removal, persists DELETE, derives newly missing roles and saves the removal audit. The actor outcome states change saved and released roles require manpower. |
| 5a3 | Replacement shares the ordinary candidate retrieval and UC-13 process. Existing assignment is retained until allowed persistence; the UPDATE targets its ID and slot. |
| 8a, 8a1 | Prevented/cancelled results cannot enter the allowed-only persistence fragment. The existing assignment, if any, remains untouched. |
| 8a2 | After the assignment request returns to the boundary, an asynchronous screen presentation gives the reason/cancellation outcome. The final candidate-list reply retains that outcome. |
| 8a3 | The boundary immediately invokes a separate `listCandidates(selection)` request, which repeats the candidate read, hours derivation and complete display. The candidate-selection loop then accepts another choice. Each Control request and boundary actor request completes before the next choice. |
| 9a, 9a1 | The documented unavailable condition takes the no-write operand at step 9. Control returns `invalidAmbulanceWarning(affectedSlot)` and boundary replies that the draft contains an invalid ambulance assignment. The inner loop ends on allowed UC-13 result; the warning disables later outer iterations and final draft save. Detection and continuation remain source issue I07. |
| Postcondition: stored draft | Initial creation persists an absent draft; accepted changes are stored incrementally; final status update retains draft. Published and warning paths do not claim the main-success postcondition. |
| Postcondition: eligible ambulance per slot | Selection enforces active/available/no-other-slot constraints and UC-13 rechecks ambulance conflict. The I07 warning path makes no valid-roster success claim. |
| Postcondition: valid staff assignments | UC-13 result gates all INSERT/UPDATE assignment SQL; direct assignments are stored confirmed against their slot and fulfilled roles. |
| Postcondition: unfilled roles | RosterSlot derives missingRoles, staffRequired and isFullyStaffed initially and after each removal/allocation. Actor replies identify remaining manpower. Derived staffing fields are not falsely stored as new columns. |
| Postcondition: available staff availability | WeekPlanningView supplies the week's staff availability; candidates also show their availability. No statement modifies StaffAvailability. |
| Postcondition: audit changes | Explicit audit INSERTs follow draft creation, assignment removal, assignment/slot change and final draft save. Each has a descriptive persisted-effect reply. |

## Shared design and proposed operations

All boundary and Control participants are the shared proposed detailed design. Canonical instance/class labels are `rosterUI:RosterBoundary`, `roster:RosterControl`, `rules:AssignmentRuleControl`, `staff:Staff`, `ambulance:Ambulance`, `slot:RosterSlot`, `assignment:Assignment`, `audit:AuditEntry` and `db:Database`. There are no additional domain classes or operations beyond the shared register.

The `ref` explicitly covers Manager, RosterBoundary, RosterControl, AssignmentRuleControl, Staff and Ambulance. Its concrete caller binds the referenced formal `baseControl`. Other shared lifelines already have identical canonical names. `proposal` supplies the selected staff, week, shift, slot/proposed row, ambulance, roles and optional replaced assignment. `context` supplies qualifications, commitments and statuses, selected/adjacent shift timings, same-shift ambulance slots and PolicyContext. The caller excludes the replaced assignment only where appropriate and retains unrelated commitments.

The orchestrator accepted two central clarifications during drafting: `AssignmentOutcome = saved(staffingStatus) | notAccepted(result) | invalidAmbulanceWarning(affectedSlot)`; and asynchronous boundary screen-progress presentation of a rejected/cancelled result before candidate refresh. Neither asserts email, durable delivery, a callback subsystem or a new persistent state. A common Control reply closes the assignment execution before the boundary makes a separate candidate-refresh request. Manager's selection execution closes after its final outcome/list reply. UC-13's nested hours decision remains part of the pending assignment request.

`staff:Staff` denotes each evaluated candidate in the hours loops and the chosen candidate at the UC-13 interaction use. `slot:RosterSlot` denotes each evaluated initial slot and the selected slot after a change. They are formal instance roles in these repeated interactions, not one permanently chosen database record throughout the complete use case.

## SQL and source assumptions

SQL is illustrative PostgreSQL with named query-API placeholders, as centrally mapped. The source supplies no physical schema. The proposed WeekPlanningView, CandidateOption and StaffRuleContext projections abbreviate required reads without claiming implementation-ready view definitions. CandidateOption is capped at three with no invented ranking/tie rule. Its datedLocation remains supplied but physically unresolved (I03), and it is never equated with station preference. Pending commitments' effects remain supplied PolicyContext (I02).

New draft and slot IDs, assignment IDs, role arrays and confirmation literals follow the shared mapping. New slots are persisted only with accepted allocation. Assignment removal is DELETE; replacement is UPDATE after approval. The Control retains authoritative retrieved context and updates it after successful writes, supplying remaining/updated assignment sets and before/new audit values without invented reloads. The final draft UPDATE is idempotent with respect to status, and its reply says retained rather than changed.

Every database request starts with a short plain-language `--` purpose on its own label line immediately above SQL. Every SELECT returns a record or record list. Every write reply acknowledges its confirmed persisted business effect without a row-count-only reply or implied returned record. Queries and writes are Control-to-database. No transaction, atomic-save guarantee, concurrent-change detection, rollback, polling or database-failure recovery has been invented.

I07 is a source gap: the unavailable-ambulance alternative supplies only a warning. Its branch has no extra persistence, invented trigger source, subsequent allocation iteration or draft-save success. Earlier saved changes are not rolled back or erased. A future implementation requires an explicit decision on detection and resumption; this diagram does not supply one.

## Author checks

- Original behaviour/class tables independently read; main/alternative/postcondition coverage checked as above.
- Single UC-13 reference; all six common lifelines covered; no duplicate invocation or checks.
- Published, no-eligible, removal, prevented, cancelled, allowed and unavailable outcomes traced through continuation.
- Every persistence request has SQL, purpose comment and corresponding explicit query/write reply.
- Boundary/Control/entity stereotype and naming conventions applied; Manager never bypasses the boundary; database replies go to the requesting Control.
- Synchronous calls use filled arrowheads, screen progress is asynchronous, and all replies use dashed open arrowheads. Activations close after their requests, with shared closure only for the same still-active request across alternatives.
- No automatic/manual message numbering, footer, legend, authoring caption, rendered issue IDs or schema/design commentary. The only opening note supplies use-case conditions.
- Standalone PlantUML `-checkonly` passed. Temporary SVG and PNG previews rendered at Arial 14 with size limit 30000. Inspected four overlapping crops of the complete 2037 × 5445 PNG; titles, heads, SQL, ref coverage, nested fragments, response arrows and final draft confirmation are readable without clipping. Final batch exports remain the orchestrator's responsibility.

Independent validation is still required. A diagram PASS establishes completeness and safe continuation at the declared abstraction; it does not resolve I02, I03 or I07 or make the illustrative SQL implementation-ready.

## Independent review correction

The validator and orchestrator found no behavioural defect. Resolved the notation cleanup by removing literal `(1,*)` from both allocation and candidate-selection loop labels, because PlantUML renders it within guard text rather than as UML loop bounds. The actual continuation guards are unchanged: `allocation requested and no invalid-ambulance warning`, and `first candidate selection or previous result prevented/cancelled`. Source coverage and branch semantics are unchanged. Re-ran the standalone PlantUML syntax check after this correction; final rendering and independent recheck use the corrected source.
