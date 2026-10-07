# UC-04 independent validation

**PASS — at the declared design abstraction.** No diagram defects were found. This result does not settle source gaps I02, I03 or I07 or make the illustrative SQL implementation-ready. Final PNG/SVG exports were checked after the notation cleanup.

Reviewed source: `uc-04-build-weekly-roster.puml`, SHA-256 `a57dae2f851c98a65241b7ea936eae0134508c5451e89fc2ad451b4b2b24771c` (288 lines).

The validator independently opened the original `Final Use Case.docx` with bundled Python/python-docx and read the complete UC-04 and UC-13 tables, rather than relying on the author's map. The original `Main Class Table.docx` was also independently read. Current `AGENTS.md`, `SEQUENCE_DIAGRAM_ORCHESTRATION_PLAN.md`, the batch interaction contract, model issues and related UC-13 source were checked. The author coverage note was read after the independent source/diagram review.

Original source hashes agree with the batch snapshot:

- `Final Use Case.docx`: `eb3ea9945513199512283e80ab0a9f2a548a8cba1a46e63256eaa1e792b6c919`.
- `Main Class Table.docx`: `26b415b0518b62b5b519d3e27bc92776ea221d45ae560c3141c6e38c0880d88e`.

## Seven review gates

| Gate | Result | Independent evidence |
|---|---|---|
| 1. Source coverage | PASS | Main 1–2: lines 37–85; main 3–4: 89–108; main 5–6: 140–160; main 7–8: 163–183; main 9: 184–234 and 253–254; main 10: outer loop 89–262 and final save 264–286. All documented alternatives and postconditions are traced below. |
| 2. Order and branch safety | PASS | Reads precede their dependent behaviour. Published status excludes the entire draft continuation (88–287). Empty eligible results exclude allocation (108–261) and can retry through the outer selection loop. Prevented/cancelled UC-13 results skip all assignment/slot writes, display the outcome, re-read/display candidates and retry step 6 (235–252, loop 163–259). An invalid-ambulance warning excludes further allocation and final save. |
| 3. Participant consistency | PASS | Canonical boundary, Control and entity names/stereotypes at 26–35 match the shared catalogue and domain baseline. Manager requests and outcomes traverse RosterBoundary. Database requests originate at RosterControl and results return there. All shown entities participate directly or through the UC-13 reference. |
| 4. Operation consistency | PASS | Boundary and Control operations, Staff.assignedHours, RosterSlot.staffingStatus, Assignment.remove/prepare and AuditEntry.recordChange match the centrally registered proposed operations. The one UC-13 reference binds the formal caller to roster:RosterControl and includes all six common lifelines (179–183). It contains the invocation, inputs and result, without a second call or duplicated checks. |
| 5. Persistence and state | PASS | Every database request contains visible SQL and a short purpose comment. Queries return records/arrays; every write has a specific persisted-effect acknowledgment. Creation, removal, accepted assignment/slot changes and draft saving are audited. No StaffAvailability write occurs. Final status acknowledgment says `roster retained as draft` (272), consistent with an unchanged saved value. |
| 6. UML and standalone source | PASS | PlantUML `-checkonly` exits 0. Nested alternatives and guarded continuations prevent invalid fallthrough. Synchronous calls, asynchronous outcome presentation, and dashed open replies are distinguished. Activations close with their corresponding execution, including the common assignCandidate reply before the separate refresh request. No automatic/manual numbering, footer, legend or rendered authoring commentary is present. |
| 7. Visual quality | PASS on final renders | Independently rendered PNG and SVG with `PLANTUML_LIMIT_SIZE=30000`; PNG is 2037 × 5445 and SVG viewBox is 2038 × 5446. Inspected all five overlapping PNG tiles at readable scale, including every header, SQL block, the UC-13 reference, nested alternatives, warning and final confirmation. No clipped messages, overlapping text, missing content or ambiguous fragment borders were found. SVG parsed successfully and has the complete diagram extent. Final delivery exports were independently checked; after the two loop-label changes, only PNG rows 1173–1186 and 2751–2764 differ from the fully inspected version. Both final loop headers were visually inspected and are readable. Final SVG text confirms both cleaned guards. |

## Alternative and postcondition trace

| Source requirement | Evidence and outcome |
|---|---|
| 1a1–1a3: published roster and UC-06 handoff | Initial persistent reads retrieve the published roster; lines 80–85 display it and identify UC-06. All later behaviour is guarded by draft status. UC-06 is not incorrectly invoked with a ref at this handoff. |
| 4a1–4a2: no eligible ambulance | Lines 101–108 report the empty result and guard all candidate/removal/allocation work. A new outer iteration starts with shift/slot selection. |
| 5a1–5a2: remove existing assignment | Lines 109–138 show selected removal, domain preparation, explicit DELETE and reply, updated missing roles, persisted audit and actor confirmation. |
| 5a3: replace existing assignment | Lines 140–160 return to the candidate display; approved replacement UPDATE is at 208–212. The existing assignment is not removed on a rejected/cancelled proposal. |
| 8a1–8a3: unaccepted proposal | The allowed guard at 184 excludes all writes for prevented/cancelled results. Lines 235–252 present the reason/outcome before refreshing candidates and their hours; loop 163–259 permits a new selection. |
| 9a1: ambulance made unavailable | The step-9 unavailable operand leads to a warning through Control and boundary (227–257), with no allocation-success claim. The outer loop guard and final-save guard stop continuation. No detection event, recovery process or extra database operation is invented. |
| Current roster stored as draft | Initial absent-draft INSERT and final draft-status UPDATE are explicit. Accepted edits persist incrementally. Published/warning exits do not falsely claim the main-success postcondition. |
| Active, available and unique same-shift ambulance | Eligibility SQL at 95 applies all three constraints; UC-13 rechecks another-slot conflict. Unavailability uses the documented warning path. |
| Valid assignments against slot/roles | Only allowed validation enters INSERT/UPDATE. Statements preserve the slot key, staff key, fulfilled roles and confirmed direct-allocation state. |
| Unfilled roles identified | Staffing status is derived for the initial view and after removal/allocation. Actor responses identify roles requiring manpower. No derived field is presented as an invented persistent column. |
| Staff availability visible and unchanged | WeekPlanningView returns availability and the actor view includes it; candidate reads expose it again. No SQL modifies StaffAvailability. |
| All draft changes audited | Separate audit INSERTs follow creation, removal and accepted assignment/slot changes. The final save event is also audited. |

## Findings and readiness

No FIX items remain. The orchestrator identified that literal `(1,*)` text in the two PlantUML loop labels rendered inside the guard rather than as UML bounds. The author removed that text at lines 89 and 163, retaining the meaningful allocation/retry guards. The validator rechecked the final source hash, standalone compilation, both final PNG headers and SVG guard text. No behaviour changed. The source's pending-assignment policy (I02), the physical source of dated candidate location (I03), and unavailability detection/resumption (I07) remain explicit limitations. The diagram preserves these as abstract policy/data or a safe warning exit. These are not silently resolved and are not diagram defects at this agreed abstraction.

Temporary validation renders and final-header crops under `/private/tmp/uc04-validator` are QA intermediates, not deliverables. The complete pre-cleanup preview was pixel-identical to the corresponding delivery PNG and its SVG was byte-identical; the final preview differs only in the two independently inspected loop headers. A later source change requires an affected-gate recheck and regenerated final PNG/SVG.
