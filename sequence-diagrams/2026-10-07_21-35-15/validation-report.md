# Validation report

**PASS for UC-02, UC-04 and UC-13 at the documented design abstraction.** All main and documented alternative scenarios have been regenerated, independently reviewed, corrected where necessary and visually inspected in both PNG and SVG. The nine required diagram files are present. Open source-policy and data-source questions remain disclosed in [model-issues.md](model-issues.md); this result does not imply implementation-ready SQL or settled pending-assignment policy.

## Review arrangement and source baseline

Each of the three use cases had an Astra (`gpt-6-astra`, `xhigh`) author and a separate independent validator. Validators read the original `Final Use Case.docx` and `Main Class Table.docx`, not only the author coverage maps. The orchestrator established and reconciled the shared contract, reviewed cross-diagram semantics and completed final file and visual checks. UC-04 and UC-13 were authored as a linked pilot; UC-02 reused the frozen participant and persistence conventions.

Source hashes are in [source-snapshot.json](evidence/source-snapshot.json). The current Word sources, AGENTS.md, plan and supporting structural files still match that snapshot. No source document or historical output was edited. Only these three use cases are part of this batch.

## Individual results

| Use case | Independent review | Source and branch safety | Participant and operation consistency | Persistence and state | UML and syntax | Final PNG and SVG |
|---|---|---|---|---|---|---|
| UC-02 Manage Availability | [Review](reviews/uc-02-validation.md) | PASS | PASS | PASS | PASS | PASS |
| UC-04 Build Weekly Roster | [Review](reviews/uc-04-validation.md) | PASS | PASS | PASS | PASS | PASS |
| UC-13 Check Assignment Rule | [Review](reviews/uc-13-validation.md) | PASS | PASS | PASS — supplied context, no extra database query | PASS | PASS |

## Findings resolved before delivery

| Finding | Correction and recheck |
|---|---|
| UC-02 opening precondition note crossed the left canvas edge | Author reanchored it over the boundary and Control. Both exports were regenerated. Parent and independent validator confirmed the whole note is now visible; its SVG geometry is inside the canvas. |
| UC-04 loop interval text appeared inside the rendered condition | Author removed `(1,*)` from both labels, preserving the actual selection/retry guards. Independent recheck passed. Image differences are confined to the two changed headers. |
| UC-04 draft execution closure needed clearer separation from candidate refresh | Before final validation, the author introduced one common assignment-outcome reply and closed that Control execution. The subsequent refresh has a separate call/execution within the related actor request. Independent activation review passed. |

## Cross-diagram consistency

- Canonical labels, `Control` names and explicit `«control»`, `«boundary»` and `«entity»` stereotypes agree with the shared catalogue. Only participating lifelines appear. The same RosterBoundary, Staff and Ambulance labels are used in the included interaction and its caller.
- UC-04 invokes UC-13 once at validation through an interaction reference covering all six common lifelines. The caller binding is `baseControl = roster:RosterControl`; proposal/context inputs and the result are explicit. UC-04 supplies retrieved context and does not duplicate rule checks or the hours decision.
- UC-13 preserves qualification, staff conflict, ambulance conflict, rest and weekly-hours order. Each of four blocking paths skips all later checks. At or below 40 hours returns allowed; above 40 returns allowed after confirmation or cancelled after cancellation. Manager participates through the boundary only for the hours decision.
- UC-04 writes assignments only when the rule result is allowed and the ambulance has not been made unavailable. Prevention/cancellation returns to candidate retrieval/display/selection. Empty eligible-ambulance results skip allocation and permit new shift/slot selection. Published status excludes draft continuation. The unavailable warning excludes further allocations and final draft success.
- UC-02 evaluates each submitted week, persists on-time and late availability with the relevant flags, records the audit, notifies Manager only for late weeks and gives one shared saved confirmation. No SQL modifies published assignments. Its late save explicitly reconciles the source alternative with the required postconditions.
- Domain responsibilities match the class table. All added operation signatures, boundaries, Control classes, transient types and physical keys are declared proposed design. No domain class table or supporting class diagram was edited.
- Every database-directed message originates at the requesting Control, contains a short purpose line immediately above parameterized SQL and receives a dashed open-arrow reply. Queries return records/arrays. Write acknowledgments name the confirmed persisted effect. The final draft acknowledgment correctly states that the roster is retained as draft.
- SQL relation/column names and placeholders agree across diagrams. `AuditEntry.recordChange` has one shared signature and SQL mapping. Candidate date-location and pending-assignment rules remain abstract documented inputs, not invented schema or policy.
- Messages have no numeric prefixes. Calls, asynchronous presentations and replies are distinguished. No captions, footer legends, authoring commentary or source-mapping labels appear in the rendered diagrams.

## Rendering and delivery verification

PlantUML 1.2026.8 compiled every standalone source successfully. PNG and SVG exports use `PLANTUML_LIMIT_SIZE=30000`, avoiding the default 4096-pixel truncation risk. Fonts remain Arial 14 with wrapped labels. All final PNGs were inspected; SVGs were independently rasterized on white backgrounds with `rsvg-convert` and inspected. UC-04 was checked in overlapping full-width sections through its final confirmation. After the final loop-label edit, image comparison confirmed only the expected guard labels changed; those regions were reinspected in both formats.

The final PNG metadata embeds the exact corresponding `.puml` source. SVG dimensions agree with PNG dimensions within the renderer's one-pixel rounding. Automated checks cover source snapshot stability, exact nine-file inventory, canonical message direction, SQL purpose/reply pairing, absence of generic write acknowledgments, balanced source activations/fragments, unsupported legends/includes/numbering, SVG bounds and artifact hashes. These checks supplement independent semantic and visual review; they do not execute the illustrative SQL or resolve source policies.

| Use case | PNG pixels | SVG canvas | Final source SHA-256 |
|---|---|---|---|
| UC-02 | 1653 × 1419 | 1654px × 1420px | `0fa0d6e317a0941d7926925ac9f45d22c388b192a90c60a696bc2601ae30f21e` |
| UC-04 | 2037 × 5445 | 2038px × 5446px | `a57dae2f851c98a65241b7ea936eae0134508c5451e89fc2ad451b4b2b24771c` |
| UC-13 | 1403 × 1311 | 1404px × 1312px | `4d4c48202b09562703d441682571309c895407f6319dce71e4e84971ee71dfce` |

[Machine-readable delivery verification](evidence/delivery-verification.json) contains hashes, SQL message counts and guard paths.

## UC-02 Manage Availability coverage
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

## UC-04 Build Weekly Roster coverage
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

## UC-13 Check Assignment Rule coverage
| Source element | Diagram representation and continuation |
|---|---|
| Description, relationships, precondition and trigger | `baseControl:BaseControl` invokes `checkAssignment(proposal, context)`. A short precondition note records the preceding staff selection in the base workflow; there is no direct actor invocation and no repeat of the base use case. The formal caller is bound to `roster:RosterControl` by the UC-04 reference. UC-06 and UC-08 are future callers outside this batch. |
| Main 1 | `Staff.isQualified(proposal.roles)` reads the supplied Staff state and returns `qualified`. |
| 1a and 1a1 | The unqualified operand returns `prevented(not qualified for role)`; every later check is contained in the opposite operand. |
| Main 2 | `Staff.hasShiftConflict(proposal, context)` returns `staffConflict`. |
| 2a and 2a1 | A same-shift assignment to another slot returns `prevented(staff already assigned in shift)`; ambulance, rest and hours checks are outside that operand. |
| Main 3 | `Ambulance.hasShiftConflict(proposal, context)` returns `ambulanceConflict`. |
| 3a and 3a1 | Another slot using the ambulance in the same shift returns `prevented(ambulance already assigned in shift)`; rest and hours checks are outside that operand. The selected slot is not treated as another slot. |
| Main 4 | `Staff.hasMinimumRest(proposal, context)` returns `hasMinimumRest`, evaluated against the 12-hour threshold. |
| 4a and 4a1 | Rest below 12 hours returns `prevented(less than 12 hours rest)`; no hours calculation or approval follows on that operand. |
| Main 5 | `Staff.projectedWeeklyHours(proposal, context)` returns `projectedHours`. |
| 5a and 5a1 | Above 40 hours, `AssignmentRuleControl` requests a decision through `RosterBoundary`; the boundary asynchronously presents the on-screen warning and confirm/cancel choice to Manager. This condition does not itself prevent assignment. |
| 5a2 confirm | Manager submits `submitHoursDecision(decision)` through the boundary; after acknowledgment and return of the decision, confirm returns `allowed` to the base Control. |
| 5a2 cancel | The same boundary decision exchange returns cancel, after which `cancelled` returns to the base Control. No assignment is created by this use case. |
| Main 6 | At or below 40 hours, or after an above-40 confirmation, `allowed` returns to the base Control. |
| Postconditions | Every completed branch returns exactly one `allowed`, reason-bearing `prevented(...)`, or `cancelled` result. The distinct cancellation result preserves alternative 5a2 even though the postcondition summarizes only allowed/prevented. The calling workflow displays the eventual actor-facing assignment outcome and persists only an allowed proposal. |

## Remaining source limitations

I02 (pending commitments in conflict/rest/hours), I03 (date-specific candidate location) and I07 (unavailable-ambulance detection/continuation) remain open. Exact Wednesday cutoff timing and candidate ranking are also unspecified. The diagrams preserve the required behaviour at an abstract policy/data level and make no speculative recovery or database-concurrency guarantees. Refer to [model-issues.md](model-issues.md) and the [interaction contract](interaction-contract.md) for the exact treatment.
