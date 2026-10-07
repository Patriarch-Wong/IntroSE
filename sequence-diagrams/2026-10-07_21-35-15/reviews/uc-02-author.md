# UC-02 Manage Availability author review

The diagram covers the complete main success scenario, late-submission alternative 5a and all postconditions. It models one logged-in staff member opening and submitting their own availability within a five-week planning period. All submitted weeks are classified before persistence; every prepared shift record is saved before the change is audited and any late notification is presented.

## Source verification

Read `AGENTS.md`, `SEQUENCE_DIAGRAM_ORCHESTRATION_PLAN.md`, this batch's `interaction-contract.md` and `model-issues.md`. Independently extracted the original UC-02 table in `Final Use Case.docx` and the complete `Main Class Table.docx` using the bundled Python `docx` library, without modifying or re-exporting either source. The UC-02 excerpt and class table match this batch's supplied evidence files. The documents skill was used for the read-only inspection. The previous UC-02 source was inspected only as a layout/design reference; the current source rules and central contract govern this regenerated diagram.

## Source coverage

| Source item | Diagram coverage |
|---|---|
| Description and precondition | Behaviour note identifies the logged-in staff member, own availability and five weeks of 12-hour day/night shifts. `staffId` comes from the authenticated session. |
| Trigger and main 1 | Staff calls `openAvailability()` through `planning:StaffPlanningBoundary`. |
| Main 2 | `StaffPlanningControl` queries dated shifts and that staff member's saved values within the five-week bounds. The database returns `shiftAvailabilityRecords[]`; the view and actor-facing outcome follow the completed read. |
| Main 3 | `selectAvailableShifts(shiftIds)` updates the boundary form and replies with the selected shifts. This execution ends before the later submit action. |
| Main 4 | `submitAvailability(submission)` calls `saveAvailability(staffId, submission)`. |
| Main 5 | The first loop calls `checkCutoff(week, submittedAt, cutoffPolicy)` for each submitted roster week. `isLate = false` produces false late flags; the later notification guard is false when none are late. |
| Main 6 | The second loop explicitly UPSERTs every prepared shift's availability and late flag. Each write returns `shift availability and late flag saved`. No Assignment or roster data is written. |
| Main 7 | `AuditEntry.recordChange(...)` prepares audit data; Control inserts it and receives `availability-change audit entry stored`. The resulting visibility note describes manager access to the persisted availability. |
| Main 8 | Control returns `availability saved` through the boundary to Staff using dashed open-arrow replies. |
| Alternative 5a and 5a1 | The per-week cutoff result identifies every affected late week. `prepareWeek(..., isLate)` gives only those weeks a true late flag. `lateWeeks` is the set of weeks whose result is true. Mixed submissions are supported. |
| Alternative 5a2 | Late records pass through the same complete persistence loop before the shared audit and manager-visibility result. This includes the save omitted from 5a's prose but required by its postconditions. |
| Alternative 5a3 | Only the guarded late branch asynchronously calls `NotificationBoundary.notify(...)` and presents the affected staff member and late weeks to Manager for consideration. |
| Alternative 5a4 | The guarded notification rejoins the common saved confirmation; no separate success path bypasses persistence. |
| Stored availability postcondition | Per-shift UPSERT uses the authenticated staff identity and prepared values from submitted weeks within the displayed five-week window. |
| Audit postcondition | A separate explicit audit INSERT completes before confirmation. |
| Late flags and notification postcondition | Per-week flags are persisted; only late submissions trigger the boundary-mediated Manager notification. |
| Published-assignment postcondition | All writes target StaffAvailability or AuditEntry. A resulting-state note confirms published assignments remain unchanged. UC-06 is a future authorized amendment, not an interaction performed in UC-02; no `ref` invokes it. |

## Participants and operations

The source defines StaffAvailability and AuditEntry responsibilities but no operations. Both entity lifelines perform the proposed operations recorded in the central contract. `prepareWeek` prepares individual-shift availability and late flags; `recordChange` prepares the audit payload. Control alone requests database persistence. There is no unused Staff, Shift or Assignment entity lifeline: their data or non-modification constraints do not require domain executions here.

The diagram uses the contract's exact participant labels and aliases: Staff, `planning:StaffPlanningBoundary`, `planning:StaffPlanningControl`, `availability:StaffAvailability`, `audit:AuditEntry`, `db:Database`, `notifications:NotificationBoundary`, and Manager. The planning interface and Control are reusable with UC-03. No new contract operation or domain attribute is introduced.

## SQL and design assumptions

- SQL is illustrative PostgreSQL using named client-bound placeholders. No physical schema was supplied. Shift has a proposed `shiftID`; StaffAvailability is unique on `(staffID, shiftID)`. This follows the central mapping.
- The read uses the authenticated staff identity in the LEFT JOIN, so saved availability belongs only to that staff member. The five-week window is expressed as inclusive `windowStart` and exclusive `windowEnd`. Shift rows are assumed to represent the source's dated 12-hour day/night shifts.
- Form submission contains only displayed weeks and shifts. `selectedShifts` in the first loop is the current week's selection. `prepareWeek` expands each submitted week to explicit true/false availability values for its displayed shifts; absent saved rows are initially unset values. Unsubmitted weeks are not changed.
- `isLate` means the submission timestamp is after the Wednesday cutoff for the current week. The exact time and timezone remain policy inputs (I13). The diagram chooses no concrete cutoff.
- `lateWeeks` is accumulated from the true cutoff results; `preparedAvailability` collects all prepared shift records. Before audit values come from the retrieved availability filtered to submitted weeks. No speculative reload, lock, transaction, rollback or atomic-save guarantee is added.
- Each write reply describes the confirmed persisted business effect and does not claim returned rows or changed-value counts. Persisting an already equal availability value still leaves that requested value saved.
- Manager visibility is a result of storing availability in the shared relation used by manager planning reads. No new manager request or view-persistence operation is invented in this scenario.
- The recipient `managerId` is supplied application routing context. The late notification is an asynchronous presentation event through NotificationBoundary (I15); no email, scheduler, durable delivery, receipt acknowledgment or transport guarantee is implied.
- I01 is explicitly reconciled: the late alternative's omitted save is supplied before audit and notification to satisfy the source postconditions and current project rule. This explanation appears only here and in non-rendered comments.
- Boundary/Control operations, value payloads, identifiers and SQL mapping remain proposed detailed design (I08). The domain table and use-case DOCX files remain unchanged.

## UML and presentation checks

Each detailed boundary, Control, entity and database execution has a matching activation/deactivation. The cutoff self-call uses a nested Control activation. Opening the page and selecting shifts finish before later user actions. The submit execution spans its own required preparation, persistence, audit, optional notification and reply. The late fragment is `opt` because both on-time and late values share the same valid save path and notification is the conditional addition; the per-week Boolean result controls stored flags without duplicated branches.

Calls use solid filled arrows (`->`), asynchronous presentations use solid open arrows (`->>`), and every reply uses dashed open arrows (`-->>`). Human interactions traverse boundaries. Rectangular participants carry explicit stereotypes; human actors use figures and the database uses a cylinder. SQL messages begin with a short purpose comment immediately above the SQL. No numbering, caption, footer legend, source mapping or authoring commentary is rendered. Arial 14 and message wrapping remain enabled without scaling.

Standalone compilation passed with `PLANTUML_LIMIT_SIZE=30000 plantuml -checkonly` (PlantUML 1.2026.8, exit 0). Independent validation is pending at initial author handoff. The parent orchestrator owns final PNG/SVG rendering and visual verification.

Parent visual QA found that the opening precondition note extended beyond the left canvas edge when anchored across Staff and Control. The source now anchors that same note across `planningUI` and `planning`; its wording and all interaction semantics are unchanged. This resolves the source placement issue for the parent's rerender and visual recheck. Standalone syntax validation passed again after the correction.
