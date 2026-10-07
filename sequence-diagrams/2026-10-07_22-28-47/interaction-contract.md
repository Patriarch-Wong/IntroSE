# Shared interaction contract — three-use-case regeneration

Version 1.2 — UC-04 retry revision; UC-02 and UC-13 retain the 2026-10-07_21-35-15 baseline. This centrally reconciled contract replaces historical pilot conventions for this batch. Behaviour is from `Final Use Case.docx`; domain responsibilities are from `Main Class Table.docx`; notation and presentation follow the current `AGENTS.md`. Current source hashes and GitHub baseline are recorded in `source-snapshot.json`. No source document was edited.

## Participants and operations

All boundary/Control classes, operation signatures, DTOs and persistence identifiers below are proposed detailed design, not operations already defined by the domain class table. `Id`, `Week`, `Hours`, `Records`, `Selection`, `Proposal`, `Context`, `Policy`, `View` and outcome types are design value types. Human actors are distinct from entity lifelines.

| Class or formal role | Responsibility and operations |
|---|---|
| StaffPlanningBoundary | Own-availability form: `openAvailability()`, `selectAvailableShifts(shiftIds)`, `submitAvailability(submission)`. Presents the results of the corresponding Control requests. |
| StaffPlanningControl | `openAvailability(staffId): AvailabilityView`; `saveAvailability(staffId, submission): SaveResult`; internal `checkCutoff(week, submittedAt, cutoffPolicy): Boolean`. Coordinates reads, preparation, persistence, audit and late notification. |
| StaffAvailability | `prepareWeek(week, selectedShifts, isLate): ShiftAvailabilityRecords`. Records selected/unselected availability and the week's late flag. |
| NotificationBoundary | `notify(managerId, event, staffId, lateWeeks)`. Presents the required late-change notification to the Manager. |
| RosterBoundary | `openAllocation(week)`, `selectSlot(shiftId, slotSelection)`, `removeAssignedRole(assignmentId)`, `chooseRoleOrReplacement(ambulanceId, roles, replacedAssignmentId?)`, `selectCandidate(staffId)`, `saveDraft(week)`; UC-13 `requestHoursDecision(projectedHours)` and `submitHoursDecision(decision)`. |
| RosterControl | `openWeek(week): RosterView`; `selectSlot(shiftId, slotSelection): AmbulanceRecords`; `removeDraftAssignment(assignmentId): RemovalResult`; `listCandidates(selection): CandidateView`; `assignCandidate(staffId, selection): AssignmentOutcome`; `saveDraft(week): DraftSaveResult`. |
| AssignmentRuleControl | `checkAssignment(proposal, context): RuleResult`. Performs ordered checks through Staff/Ambulance and requests the Manager's hours decision through RosterBoundary. |
| baseControl | Formal caller role in UC-13, bound to `roster:RosterControl` in UC-04/06 or the proposed `assignments:AssignmentControl` in UC-08. It is not an added domain class. Its surrounding execution is outside the detail of UC-13. |
| Staff | `isQualified(roles): Boolean`; `hasShiftConflict(proposal, context): Boolean`; `hasMinimumRest(proposal, context): Boolean`; `projectedWeeklyHours(proposal, context): Hours`; `assignedHours(week, assignmentRecords, policy): Hours`. The lifeline denotes the staff member currently evaluated, then the selected member. |
| Ambulance | `hasShiftConflict(proposal, context): Boolean`. Its UC-04 participation is inside the UC-13 reference. |
| RosterSlot | `staffingStatus(assignments, crewPolicy): StaffingStatus(missingRoles, staffRequired, isFullyStaffed)`. These are derived values, not extra persisted columns. |
| Assignment | `remove(reason?): ChangeData`; `prepare(proposal, confirmed): AssignmentData`. Both prepare domain changes; explicit Control-to-database SQL performs persistence. |
| AuditEntry | `recordChange(event, actor, time, previousValue, newValue, reason): AuditData`. Prepares the audit payload; Control inserts it. The event is a descriptive design input, carried in the value payload where required, not an assumed domain attribute. |

`Selection` identifies the week, shift, existing slot or proposed row, ambulance, roles and optional replaced assignment. RosterControl retains the retrieved working context and updates it after each supported successful write; boundary input does not replace authoritative retrieved records. Before/new audit values and remaining slot assignments are derived from that context and the accepted changes. No extra reload is asserted.

## UC-13 and continuation

The UC-04 `ref` represents the entire UC-13 invocation and return: `result = checkAssignment(proposal, context)`. It covers all shared lifelines, with the caller bound to `baseControl`. There is no duplicate standalone call in UC-04. UC-13 returns `allowed`, `prevented(reason)` or `cancelled`; these are transient results, not persistent assignment states.

The context includes qualifications, relevant commitments, shift timing, same-shift ambulance slots and workload. Exclude the replaced assignment for checks where appropriate, without discarding other commitments; ambulance conflict concerns another slot, not the selected slot itself. The caller retrieves the records before the reference, so UC-13 adds no database query. Pending-request contributions remain a supplied, unresolved PolicyContext (I02).

UC-13's nested valid operands preserve qualification → staff conflict → ambulance conflict → 12-hour rest → weekly-hours order. Every blocking result leaves all subsequent checks outside its selected operand. Above 40 hours prompts rather than blocks. The calling boundary supplies the eventual actor-facing save, rejection or cancellation outcome.

UC-04 retries candidate retrieval/display after prevented/cancelled results. Only `allowed` enters the save-or-unavailable-warning choice. An unavailable warning suppresses subsequent allocation iterations and final draft save; its unknown continuation is not replaced with a successful outcome. The published branch cannot enter draft-only continuation. Choosing another slot after no eligible ambulance takes a new outer iteration.

## Illustrative SQL mapping

The source documents supply no schema or SQL. Messages use **illustrative PostgreSQL**, with named placeholders bound by a query API; `:name` is a diagram/client placeholder, not raw PostgreSQL server syntax. This mapping is proposed to make reads and writes reviewable, not a claim that these physical tables/views exist. PostgreSQL identifiers shown without quotes are treated consistently case-insensitively. Collection/value payloads may use arrays or JSONB.

| Relation | Assumed key/columns and use |
|---|---|
| Shift | Proposed `shiftID`; source date, shiftType, startTime, endTime. Calendar shifts exist for the planning window. |
| StaffAvailability | Unique `(staffID, shiftID)`, `available`, `lateFlag`; absent records are unset form values. A submitted week is expanded into explicit true/false records for its displayed shifts. UPSERT changes only submitted weeks and never Assignment. |
| WeeklyRoster | `rosterWeek` key and `status` (`draft`/`published`). New draft is stored before its details are presented. |
| Ambulance | `ambulanceNumber`, `activeStatus`; `'active'` is a proposed status literal. |
| AmbulanceAvailability | `(ambulanceNumber, shiftID)`, `available`. For this SQL mapping, a row supplies explicit availability for every relevant ambulance/shift; no missing-row default is assumed. |
| RosterSlot | Proposed `slotID`; foreign keys `rosterWeek`, `shiftID`, `ambulanceNumber`. A proposed UI row is not a stored slot until accepted allocation. Control supplies a new ID before its INSERT. |
| Assignment | Proposed `assignmentID`, `slotID`, `staffID`, `fulfilledRoles` as an array, `confirmationStatus`. Direct allocation is confirmed. Replacement updates the existing assignment record only after approval; removal deletes it while the audit retains prior data. This is a persistence design choice, not a new rejection status. |
| AuditEntry | Proposed generated audit key (omitted from INSERT), `actingUser`, `timestamp`, `previousValue`, `newValue`, `reason`. Parameters come from AuditData. |
| WeekPlanningView | Proposed read projection returning one week record with status, slot/assignment arrays, assigned ambulance/staff details and staff availability, including an empty slot array for a new draft. It aggregates the relevant domain relations and is not a new domain class. |
| CandidateOption | Proposed read projection exposing staffID, availability, assignmentRecords, jobPreference and datedLocation for week/shift/ambulance/requestedRoles. It represents the source's suitable-candidate retrieval. SQL caps results at three; no ranking or tie policy is asserted. Selection/suitability details are not silently implemented as extra UC-13 checks. |
| StaffRuleContext | Proposed read projection of Staff qualifications and relevant Assignment/RosterSlot/Shift data, keyed by staffID and rosterWeek. Includes the weekly commitments and adjacent shifts across week boundaries needed for rest checks. It preserves status information for PolicyContext. |

The proposed projections abbreviate required reads without pretending to have final view definitions. In particular, `CandidateOption.datedLocation` is a **logical supplied field with an unresolved physical source (I03)**. It is not inferred from station preference and the SQL is not implementation-ready until that source is defined. Pending commitments' impact on candidate workload and UC-13 stays abstract (I02).

INSERT/UPDATE/DELETE replies describe the confirmed persisted business effect, never a bare row count or generic success. Examples: `availability and late flags saved`, `assignment stored`, `assigned staff replaced`, `assignment removed`, `availability-change audit entry stored`, `roster retained as draft`. No returned records are implied unless SQL supplies them. Queries return records/arrays. No unrequested database-failure branch, transaction, lock, rollback or atomic-save guarantee is introduced. Domain changes are saved incrementally; the final draft UPDATE persists the draft status after those saves. Its affected-row count does not imply that the value changed or that previous writes formed a transaction.

## Notifications, assumptions and source gaps

- **I01:** UC-02's late alternative omits its save, while its postconditions require storage. Both on-time and late records are explicitly persisted before audit/notification.
- **I02:** Pending assignments' contribution to conflict/rest/hours remains unresolved; no recommended policy has been adopted by this revision.
- **I03:** Date-specific staff location lacks a defined source. The proposed candidate read projection records that gap rather than inventing a permanent station association.
- **I07:** UC-04 alternative 9a supplies an unavailable-ambulance warning without detection, resumption or persistence details. The branch models the warning at the documented point and stops without a persisted-success claim. No polling, callback source, scheduler or recovery process is invented.
- **I08:** Boundary/Control operations, DTOs, identifiers and mapping choices are proposed detailed design. No new domain attributes are asserted.
- UC-02 manager notification and the UC-13 on-screen prompt are shown as asynchronous presentations through boundaries. This assumes a presentation event, not email, durable delivery or acknowledgment. Manager notification is not an audit or database reply.
- The exact Wednesday cutoff time and candidate selection/ranking policy are not supplied. They are policy inputs; the diagrams do not select concrete values. All guards refer to the retrieved context or the source's stated condition.

## Rendering conventions

Standalone PlantUML; Arial 14; rectangular stereotyped boundary/Control/entity heads; actor figures and database cylinder; no message numbering. Synchronous calls use `->`, asynchronous messages use `->>`, and replies use `-->>`. Activation bars are balanced within each branch, with shared closure after alternatives where the same invocation remains active. No execution spans an unrelated later user action. Guards are written without literal outer brackets because the renderer supplies them.

## Canonical participant catalogue

Use these instance labels consistently; aliases are implementation shorthand only. UML classes appear as rectangular `participant` heads with explicit stereotypes.

| UML role | Instance label | Alias | Scope / reuse |
|---|---|---|---|
| actor | Staff | staffActor | UC-02 primary actor |
| actor | Manager | manager | UC-04 primary actor; UC-02 late recipient; UC-13 hours decision |
| boundary | planning:StaffPlanningBoundary | planningUI | UC-02; coherent future reuse for UC-03 |
| control | planning:StaffPlanningControl | planning | UC-02; coherent future reuse for UC-03 |
| boundary | rosterUI:RosterBoundary | rosterUI | UC-04/13; future allocation/amendment interface |
| control | roster:RosterControl | roster | UC-04; future UC-05/06 |
| control | rules:AssignmentRuleControl | rules | UC-13; included from UC-04, future UC-06/08 |
| control formal caller | baseControl:BaseControl | baseControl | UC-13 formal role, bound to roster:RosterControl for UC-04, not a new concrete class |
| entity | staff:Staff | staff | UC-04/13 selected staff; UC-04 candidate loop substitutes evaluated candidate |
| entity | availability:StaffAvailability | availability | UC-02 |
| entity | slot:RosterSlot | slot | UC-04 selected/evaluated slot |
| entity | assignment:Assignment | assignment | UC-04 change preparation |
| entity | ambulance:Ambulance | ambulance | UC-13 and UC-04 reference |
| entity | audit:AuditEntry | audit | UC-02/04 |
| database | db:Database | db | UC-02/04 |
| boundary | notifications:NotificationBoundary | notifications | UC-02; reusable actor notifications |

`AssignmentProposal` identifies staffId, week, shiftId, slotId (or proposed row), ambulanceId, roles and optional replacedAssignmentId. `RuleContext` contains retrieved qualifications, commitments (including adjacent-week timing), same-shift ambulance slots and `PolicyContext`. No extra UC-13 database query is needed. `RuleResult = allowed | prevented(reason) | cancelled`. The UC-04 reference covers Manager, RosterBoundary, RosterControl, AssignmentRuleControl, Staff and Ambulance; no duplicate call/checks outside the reference. Use the exact UC-13 title and input/result binding on that reference, wrapping it for readability.

Actor-to-boundary operations return actor-facing outcomes through dashed open-arrow replies. The hours prompt is an asynchronous boundary presentation and the manager's decision is a new boundary request; the rule invocation remains pending because that decision is part of its execution. UC-02 late notification is asynchronous, with no transport/delivery guarantees implied.

Each SQL arrow starts with a brief `-- Purpose` comment on its own line directly above SQL. Keep SQL and parameter placeholders visible and wrapped. Record schema limitations in documentation/comments only. Every confirmed write effect has a descriptive dashed open-arrow reply; query results are records/lists/arrays. The final draft UPDATE acknowledges `roster retained as draft` when the value was already draft.

No automatic/manual message numbering; no legends, authoring notes or source-coverage captions in diagrams. Preconditions and behaviour notes are permitted. The use-case ID and exact name form each title. Default Arial 14 and `maxMessageSize 350`, white background, dark arrows, subtle boundary/control/entity colours, `hide footbox`. No scale directive. Render with PLANTUML_LIMIT_SIZE=30000.

Any signature/mapping adjustment proposed by an author must be reconciled here by the orchestrator before final validation. Individual diagram author files and coverage notes are exclusively owned by the assigned author; validators report findings rather than changing them.

## Candidate retry presentation

UC-04 has one candidate-attempt loop containing steps 6–9. Its guard is true for the first attempt following role/replacement selection and thereafter only when UC-13 returned prevented(reason) or cancelled. Candidate retrieval, assigned-hours derivation and display occur once in the source, at the beginning of this loop. After an unsuccessful proposal, the boundary presents the reason/outcome asynchronously and the loop repeats from step 6 without another role selection. This is an on-screen presentation event, not a separate notification service.

There is exactly one pending boundary execution at each loop entry: chooseRoleOrReplacement on first entry, or selectCandidate after prevention/cancellation. The candidate-display reply closes that pending actor request before the Manager selects a candidate. The new selectCandidate execution closes after the saved/warning reply when the loop ends; on rejection it remains pending only through the related candidate refresh at the next iteration. RosterControl's assignCandidate execution always returns and closes before any refresh call. No execution spans a later unrelated user action.

Assignment persistence has one combined guard: result is allowed AND the ambulance has not been made unavailable. This replaces the baseline nested allowed/available guards without changing accepted, prevented, cancelled or warning paths. All SQL, write acknowledgments, audit behaviour and the UC-13 binding are preserved; only the duplicate candidate query is removed from the drawn source, and it still executes on every retry. The unavailable-warning stop remains the baseline's documented I07 assumption and is not settled by this revision.

## Operation signatures and results

Parameters below are proposed design types. `Selection` and `Proposal` are the previously defined records; `Context`, `PolicyContext`, `View`, `Records`, `AuditData` and `ChangeData` are transient values, not added domain classes. UI input operations return presentation outcomes. Only the database arrows perform persistence.

| Owner | Operation and parameters | Result | Calling UC |
|---|---|---|---|
| StaffPlanningBoundary | openAvailability() | saved availability and selectable shifts | 02 |
| StaffPlanningBoundary | selectAvailableShifts(shiftIds: Id[]) | selected shifts | 02 |
| StaffPlanningBoundary | submitAvailability(submission: AvailabilitySubmission) | availability saved | 02 |
| StaffPlanningControl | openAvailability(staffId: Id) | AvailabilityView | 02 |
| StaffPlanningControl | saveAvailability(staffId: Id, submission: AvailabilitySubmission) | SaveResult | 02 |
| StaffPlanningControl | checkCutoff(week: Week, submittedAt: DateTime, cutoffPolicy: CutoffPolicy) | Boolean isLate | 02 |
| StaffAvailability | prepareWeek(week: Week, selectedShifts: Id[], isLate: Boolean) | ShiftAvailabilityRecords[] | 02 |
| NotificationBoundary | notify(managerId: Id, event: String, staffId: Id, lateWeeks: Week[]) | asynchronous presentation; no acknowledgment assumed | 02 |
| RosterBoundary | openAllocation(week: Week) | roster view or published handoff | 04 |
| RosterBoundary | selectSlot(shiftId: Id, slotSelection: SlotSelection) | eligible ambulances or no eligible ambulance outcome | 04 |
| RosterBoundary | removeAssignedRole(assignmentId: Id) | removed assignment and manpower status | 04 |
| RosterBoundary | chooseRoleOrReplacement(ambulanceId: Id, roles: Roles, replacedAssignmentId: Id?) | candidate view | 04 |
| RosterBoundary | selectCandidate(staffId: Id) | accepted allocation, refreshed candidates after prevention/cancellation, or unavailable warning | 04 |
| RosterBoundary | saveDraft(week: Week) | draft saved | 04 |
| RosterBoundary | requestHoursDecision(projectedHours: Hours) | Decision = confirm or cancel | 13 |
| RosterBoundary | submitHoursDecision(decision: Decision) | decision received | 13 |
| RosterControl | openWeek(week: Week) | RosterView | 04 |
| RosterControl | selectSlot(shiftId: Id, slotSelection: SlotSelection) | AmbulanceRecords[] | 04 |
| RosterControl | removeDraftAssignment(assignmentId: Id) | RemovalResult | 04 |
| RosterControl | listCandidates(selection: Selection) | CandidateView | 04 |
| RosterControl | assignCandidate(staffId: Id, selection: Selection) | AssignmentOutcome | 04 |
| RosterControl | saveDraft(week: Week) | DraftSaveResult | 04 |
| AssignmentRuleControl | checkAssignment(proposal: Proposal, context: RuleContext) | RuleResult | 13, invoked by 04 ref |
| Staff | isQualified(roles: Roles) | Boolean | 13 |
| Staff | hasShiftConflict(proposal: Proposal, context: RuleContext) | Boolean | 13 |
| Staff | hasMinimumRest(proposal: Proposal, context: RuleContext) | Boolean | 13 |
| Staff | projectedWeeklyHours(proposal: Proposal, context: RuleContext) | Hours | 13 |
| Staff | assignedHours(week: Week, assignmentRecords: Records[], policy: PolicyContext) | Hours | 04 |
| Ambulance | hasShiftConflict(proposal: Proposal, context: RuleContext) | Boolean | 13 |
| RosterSlot | staffingStatus(assignments: Records[], crewPolicy: CrewPolicy) | StaffingStatus(missingRoles, staffRequired, isFullyStaffed) | 04 |
| Assignment | remove(reason: String?) | ChangeData | 04 |
| Assignment | prepare(proposal: Proposal, confirmationStatus: ConfirmationStatus) | AssignmentData | 04 |
| AuditEntry | recordChange(event: String, actor: Id, time: DateTime, previousValue: Value?, newValue: Value?, reason: String?) | AuditData | 02, 04 |

Final palette: boundary `#EAF4FB`, Control `#FFF3DA`, entity `#EDF6EC`, arrows `#263238`, lifelines `#607D8B`, participant borders `#455A64`. Database remains a cylinder with a neutral fill. All classes use explicit stereotypes and canonical instance labels.

UML interaction semantics were checked against the [OMG UML 2.5 specification](https://www.omg.org/spec/UML/2.5/PDF), clauses 17.4, 17.6 and 17.7. Nested valid alternatives are used to prevent invalid continuation. Interaction references bind every common lifeline and their inputs/result.

`AssignmentOutcome = saved(staffingStatus) | notAccepted(result) | invalidAmbulanceWarning(affectedSlot)`. RosterControl's assignCandidate execution returns exactly one of these outcomes and closes before any separate candidate-refresh call. `notAccepted` wraps a prevented reason or cancellation from UC-13; it is not a persisted status. The boundary's original actor request remains active through the related outcome presentation and candidate refresh, then closes.

## Baseline and revision status

This version adapts the independently reviewed three-use-case baseline for the focused UC-04 retry edit. Current checks are recorded in validation-report.md; this revision does not claim a new independent-agent review. Subsequent changes to a shared signature, result, participant binding or SQL mapping require revalidation of every affected diagram. Source-step coverage and final rendered artifact checks are in validation-report.md.
