# UC-02 and UC-04 under the updated AGENTS.md

| Use case | Diagram | Source |
|---|---|---|
| UC-02 Manage Availability | [SVG](uc-02-manage-availability.svg) · [PNG](uc-02-manage-availability.png) | [PlantUML](uc-02-manage-availability.puml) |
| UC-04 Build Weekly Roster | [SVG](uc-04-build-weekly-roster.svg) · [PNG](uc-04-build-weekly-roster.png) | [PlantUML](uc-04-build-weekly-roster.puml) |

Both follow the AGENTS.md revision on main (`b723731`): short business-operation labels on database arrows, the key SQL in side notes, supporting SQL in source comments, no message numbering, at most three levels of nesting and roughly 20–35 messages per diagram. Both also keep to the team's request for no nested loops.

## Review notes

| Diagram | Messages | Max nesting | Deepest path |
|---|---|---|---|
| UC-04 Build Weekly Roster | 23 | 3 | loop → opt (candidates shown) → alt (UC-13 result) |
| UC-04 Prepare Allocation | 9 | 1 | break (no eligible ambulance / removal) |
| UC-04 Remove Draft Assignment | 6 | 0 | none |
| UC-02 Manage Availability | 18 | 1 | loop (each submitted week); opt (late) |

Traced paths: UC-04 published (break ends the interaction before allocation and draft save), no eligible ambulance (only another slot selection is offered next), fill, replace, remove (via the referenced interaction), UC-13 allowed / prevented / cancelled (retry from the candidates shown), ambulance made unavailable, and draft save after the loop. UC-02 on-time and late both save before the late-only notification and reach the same confirmation. Activation bars close on every branch; both `ref`s cover every lifeline they share with the caller.

## UC-04 Build Weekly Roster

**Scope:** main success scenario and alternatives 1a, 4a, 5a, 8a and 9a, across three diagrams, laid out in the use case's step order 1→10. Steps 1–2 open the roster; 1a is a top-level `break` straight after it. Inside the one allocation loop, steps 3–6 are the referenced [UC-04 Prepare Allocation](uc-04-prepare-allocation.svg) ([source](uc-04-prepare-allocation.puml)), which also holds 4a (`break`: no eligible ambulance) and 5a (`break` to the referenced [UC-04 Remove Draft Assignment](uc-04-remove-assignment.svg), or continue to candidates when replacing). Steps 7–9 follow in an `opt [candidates are shown]`, so 4a and removal never reach candidate selection. After 8a the candidates stay shown, so the next pass skips Prepare Allocation and returns to candidate selection (step 6/7) without a second loop. Step 10 saves the draft after the loop. The main diagram keeps the UC-13 reference, every assignment outcome and the three core database operations.

| Visible database operation | Behaviour represented | SQL |
|---|---|---|
| `retrieveOrCreateWeeklyRoster(week)` | Read the week's roster, create an empty draft if absent, read planning data | WeeklyRoster SELECT / INSERT in side note; WeekPlanningView SELECT in source comment |
| `removeStaffAssignment(assignmentId, auditData)` (5a, referenced diagram) | Delete the assignment and record the removal | DELETE and AuditEntry INSERT in side note |
| `saveStaffAssignment(proposal, auditData)` | Store or replace the assignment, store the slot's ambulance, record the change | Assignment upsert in side note; RosterSlot upsert and AuditEntry INSERT in source comments |
| `saveDraftAndAudit(week, auditData)` | Retain draft status and record the draft save | UPDATE in side note; AuditEntry INSERT in source comment |

Supporting reads (eligible ambulances, candidates with assigned hours, rule context with the ambulance's current availability) are not drawn as round trips; their SQL is in source comments beside `selectSlot` and `listCandidates` (Prepare Allocation) and `assignCandidate` (main diagram).

| Source step | Diagram |
|---|---|
| 1–2 | `openAllocation`, `retrieveOrCreateWeeklyRoster` |
| 1a | `break [roster status is published]`; use case ends |
| 3–6, 4a, 5a | `opt [starting a new allocation]` → `ref` UC-04 Prepare Allocation (4a and removal are `break`s inside it; removal refs UC-04 Remove Draft Assignment) |
| 7–8 | `opt [candidates are shown]`; `ref` UC-13 |
| 9 / 9a / 8a | Inner `alt`: saved / invalid ambulance warning / not saved |
| 10 | Loop repeats; `saveDraft` after it |

## UC-02 Manage Availability

**Scope:** main success scenario and alternative 5a (late submission). One loop checks the Wednesday cut-off and prepares records per submitted roster week. Both paths save availability with late flags and an audit entry, then reach the same confirmation; only the late path notifies the Manager through `NotificationBoundary` (asynchronous, on-screen; no email assumed). The late-branch save reconciles 5a with the postconditions (I01).

| Visible database operation | Behaviour represented | SQL |
|---|---|---|
| `retrieveAvailability(staffId, window)` | Read the staff member's saved availability and the five-week shifts | SELECT in side note |
| `saveAvailabilityAndAudit(records, auditData)` | Upsert each shift's availability and late flag; record the change | StaffAvailability upsert in side note; AuditEntry INSERT in source comment |

Simplifications from the [previous UC-02](../2026-10-07_21-35-15/uc-02-manage-availability.puml): the boundary-only shift selection is folded into `submitAvailability(selectedShifts)`, the per-record save loop and the `AuditEntry` entity call are grouped under `saveAvailabilityAndAudit`.

## Assumptions and open issues

- Combined operations name logical persistence calls; they do not imply a transaction or atomicity.
- SQL is illustrative PostgreSQL; projections and mappings follow the [previous interaction contract](../2026-10-07_22-28-47/interaction-contract.md). `CandidateOption` supplies `assignedHours`; `StaffRuleContext` supplies the ambulance's current availability.
- 9a (I07): unavailability is assumed to be detected from the availability read at assignment time; only the warning is shown.
- Wednesday cut-off time/timezone (I13), pending-assignment policy (I02) and date-specific location (I03) remain open, see [model issues](../2026-10-07_22-28-47/model-issues.md).

Rendered with PlantUML 1.2025.4 (`plantuml -tpng` / `-tsvg`).
