# UC-04 sample following the supplied rough draft

Start with the [main flow SVG](uc-04-build-weekly-roster.svg), [PNG](uc-04-build-weekly-roster.png), or [editable PlantUML](uc-04-build-weekly-roster.puml).

The main flow follows the supplied rough draft's readable layout: Manager, boundary, Control, participating entities and database; short call/reply labels; repeated allocation; UC-13 validation; accepted/rejected outcomes; and final draft saving. It has three visible database requests. The key original SQL sits in three side notes, with supporting SQL retained in source comments. The maximum local control-flow nesting is three.

The project conventions continue to apply: rectangular stereotyped class heads, Control-to-database routing, meaningful actor outcomes, no message numbering, balanced activation bars and dashed open-arrow replies. The rough draft's entity-to-database routing and transaction statements were not adopted. Only participating domain entities are included; WeeklyRoster is not added merely to match the reference's column count.

## Scenario scope

The main diagram covers the main draft-allocation flow and alternative 8a: UC-13 prevention/cancellation followed by another candidate attempt. In these traces, the selected roster is a draft, an eligible ambulance exists, the Manager fills a role, and the ambulance remains available. The first draft can be created if absent. Other documented alternatives are retained in the companion scenario overview; the main diagram alone is not the complete use-case specification.

The candidate-attempt loop starts at candidate retrieval/display. UC-13 is invoked once per attempt. Only `allowed` reaches the assignment save. Rejection/cancellation performs no assignment write, presents the outcome, and refreshes candidates before the next selection. The boundary execution remains pending only through this related refresh and closes when the candidate view is presented. Its reason/outcome presentation is an asynchronous on-screen event, consistent with the earlier sample.

## Core persistence operations

These are proposed logical persistence-interface operations, not literal SQL statements, newly assumed stored procedures, or transaction guarantees. RosterControl remains responsible for coordinating them. Their replies confirm the documented effects before the saved outcome reaches the Manager.

| Visible operation | Behaviour represented | Original SQL retained |
|---|---|---|
| `retrieveOrCreateWeeklyRoster(week)` | Read the selected roster; create an empty draft if absent; record creation when applicable; retrieve its planning context. | WeeklyRoster SELECT/conditional INSERT in the side note. Creation audit and WeekPlanningView SELECT also appear in source comments. |
| `saveStaffAssignment(proposal, auditData)` | Persist the accepted assignment, any required slot insert/change, and its audit record. Refresh derived staffing status afterwards. | Assignment INSERT in the side note. Conditional RosterSlot INSERT/UPDATE and AuditEntry INSERT in source comments. |
| `saveDraftAndAudit(week, auditData)` | Retain draft status and persist the draft-save audit record. | Original WeeklyRoster UPDATE and AuditEntry INSERT in the side note and source comments. |

The original eligible-ambulance query remains beside `selectSlot` in source comments. Candidate retrieval SQL remains beside `listCandidates`; staff-rule and ambulance-slot context SQL remains before UC-13. These supporting queries are abstracted from the rendered business-flow view, not removed from the behaviour or replaced with a new caching design. Data still has to be retrieved before dependent checks; no snapshot, freshness, locking or concurrency guarantee is asserted.

SQL wording and physical mappings come from the [existing interaction contract](../2026-10-07_22-28-47/interaction-contract.md). The schema was not supplied and SQL remains illustrative PostgreSQL with named query parameters. This sample's presentation and logical persistence operations extend that contract; the older requirement to place SQL on every database arrow does not apply to this requested layout. Audit data, candidate context and derived staffing values retain their existing meanings. Preparing audit/assignment payloads is abstracted in the compact main flow; the companion diagrams retain those domain operations.

## Companion diagrams

| Interaction | Diagram | Source |
|---|---|---|
| Broader scenario overview | [SVG](uc-04-scenario-overview.svg) · [PNG](uc-04-scenario-overview.png) | [PlantUML](uc-04-scenario-overview.puml) |
| Open roster | [SVG](uc-04-open-week.svg) | [PlantUML](uc-04-open-week.puml) |
| Allocate role, including retries | [SVG](uc-04-allocate-role.svg) | [PlantUML](uc-04-allocate-role.puml) |
| Save accepted assignment | [SVG](uc-04-save-assignment.svg) | [PlantUML](uc-04-save-assignment.puml) |
| Remove draft assignment | [SVG](uc-04-remove-assignment.svg) | [PlantUML](uc-04-remove-assignment.puml) |
| Save draft | [SVG](uc-04-save-draft.svg) | [PlantUML](uc-04-save-draft.puml) |
| UC-13 Check Assignment Rule | [Existing SVG](../2026-10-07_21-35-15/uc-13-check-assignment-rule.svg) | [Existing PlantUML](../2026-10-07_21-35-15/uc-13-check-assignment-rule.puml) |

The broader overview preserves the published-roster exit (1a), selection retry when no eligible ambulance exists (4a), removal/replacement (5a), candidate retry (8a) and unavailable-ambulance warning (9a). The 4a sample depicts the Manager electing to retry; it does not introduce an abandon-selection action. The published and warning breaks span all overview lifelines and sit outside its loops, preventing final draft saving on those paths.

In the companions, `saveStaffAssignment(confirmedAssignmentData, slotChanges)` represents assignment/slot persistence, while the overload taking `removalChange` represents deleting the selected assignment. Each is followed by `saveAuditRecord(auditData)`. `saveDraft(week)` plus `saveAuditRecord(auditData)` expands the compact main flow's combined draft/audit operation. The open-roster companion uses `retrieveOrCreateRosterData(week)` for the roster/planning persistence and expands creation auditing explicitly. These are two views of the same required effects, not extra invocations by the main flow. Grouped calls do not promise atomicity.

The internal interaction signatures remain `loadWeek(week): rosterView`, `allocateRole(slotSelection, eligibleAmbulanceRecords, workingContext): outcome`, `persistAccepted(proposal, workingContext): staffingStatus`, `removeExisting(assignmentId, workingContext): staffingStatus`, and `retainDraft(week, workingContext): currentDraft`. Internal helper activation bars show the caller's continuing RosterControl execution. Each reference covers every shared lifeline. UC-13 binds `baseControl` to `roster:RosterControl` and covers Manager, rosterUI, roster, rules, staff and ambulance.

## Sources and limitations

Behaviour is defined by Final Use Case.docx and the domain baseline by Main Class Table.docx. The complete detailed SQL baseline remains in [the previous revision](../2026-10-07_22-28-47/uc-04-build-weekly-roster.puml). All 17 original database-request SQL statements are retained as comments across the companion sources; the compact main flow retains the subset applicable to its scenario. The source documents and prior timestamped diagrams are unchanged.

The [open model issues](../2026-10-07_22-28-47/model-issues.md) remain: pending-assignment policy (I02), date-specific candidate location (I03), and unavailable-ambulance detection/continuation (I07). The companion warning exit retains the existing I07 assumption; the main sample does not depict that scenario. This visual simplification does not resolve those source gaps.

## Verification

See [validation.json](validation.json). All seven sources compile and have regenerated SVG/PNG outputs. Checks confirm three visible database requests in the main flow, original SQL retained in comments, balanced activation totals and fragment structure, and consistent reference coverage. The main layout was visually inspected. Checks do not execute SQL or establish implementation-level guarantees.
