# UC-13 independent validation

**Decision: PASS for source, semantics, contract consistency and independently rendered PNG review.** No diagram defect requiring correction was found. The orchestrator owns the final delivered PNG/SVG render and visual gate. This is a pass at the documented design abstraction, conditional on the unresolved pending-assignment policy; it is not a claim of implementation readiness.

The validator independently read the UC-13 table in the original `Final Use Case.docx`, its UC-04/06/08 callers, and all twelve class rows in the original `Main Class Table.docx` using bundled Python and `python-docx`. The current `AGENTS.md`, seven orchestration gates, batch interaction contract and model issues were checked. Author coverage was read only after the independent source criteria were established. No author, contract or original source file was edited.

## Reviewed versions

| File | SHA-256 |
|---|---|
| `uc-13-check-assignment-rule.puml` | `4d4c48202b09562703d441682571309c895407f6319dce71e4e84971ee71dfce` |
| `uc-04-build-weekly-roster.puml` — reference integration only | `a45ead6e014ee88147dc9ebd878f9fbc7884b899df5cf61d7991e129b48d0f80` |
| `interaction-contract.md` | `64bf4f1842fefe77a5b6efda0c1d0f62d70967eea5694b8d473410f1461f724b` |
| `Final Use Case.docx` | `eb3ea9945513199512283e80ab0a9f2a548a8cba1a46e63256eaa1e792b6c919` |
| `Main Class Table.docx` | `26b415b0518b62b5b519d3e27bc92776ea221d45ae560c3141c6e38c0880d88e` |
| `AGENTS.md` | `01fb06ebb175e456ed2f589b8a12a4c723e797eb7eb61a3aeff32f958153a583` |

## Seven gates

| Gate | Result | Independent evidence |
|---|---|---|
| 1. Source coverage | PASS | Lines 39–40 invoke the included use case from the formal base Control. Main steps 1–5 are represented by calls at lines 43, 51, 59, 67 and 75. Main step 6 returns allowed at lines 90 and 96. Alternatives 1a–4a return reason-bearing prevention at lines 48, 56, 64 and 72. Alternative 5a warns and obtains confirm/cancel at lines 79–93. The preceding staff selection is a precondition note, not a repeated base interaction. |
| 2. Ordering and branch safety | PASS | Qualification → staff conflict → ambulance conflict → minimum rest → weekly hours is preserved. Each later check is nested in every preceding valid operand. Four prevention paths, ordinary allowed, confirmed allowed and cancelled are seven mutually exclusive terminal paths; each returns one result. No blocked or cancelled operand reaches a later check or allowed result. Exactly 12 hours passes; exactly 40 hours does not prompt. |
| 3. Participant consistency | PASS | Manager, `rosterUI:RosterBoundary`, `rules:AssignmentRuleControl`, `staff:Staff` and `ambulance:Ambulance` match the catalogue. `baseControl:BaseControl` is the declared formal caller. All participants interact. The Manager participates only at the hours decision and only through the boundary. Domain behavior matches Staff and Ambulance responsibilities. |
| 4. Operation consistency | PASS | `checkAssignment`, `isQualified`, both `hasShiftConflict` operations, `hasMinimumRest`, `projectedWeeklyHours`, `requestHoursDecision` and `submitHoursDecision` match the registered owners, inputs and results. Result vocabulary is `allowed`, `prevented(reason)` or `cancelled`; none is asserted as a persisted state. |
| 5. Persistence and state | PASS | UC-13 has no database secondary actor and the base supplies retrieved context. No gratuitous database query, hidden entity persistence, audit or save success appears. Cancellation returns to the caller without creating an assignment. Rule-result reporting and subsequent persistence remain in the base use case. |
| 6. UML and source quality | PASS | Standalone `plantuml -checkonly` independently passed, exit 0. Correct `alt` structure, concrete guards, explicit stereotypes, synchronous filled arrows (`->`), asynchronous warning (`->>`) and dashed open replies (`-->>`) are used. No message numbering or authoring notes render. Staff/Ambulance activations pair with their calls/replies. The rules execution closes once after its mutually exclusive terminal responses. The prompt's boundary execution remains pending for its related decision; the nested submission execution closes before the prompt returns. No unrelated user action is bridged by an activation. |
| 7. Visual quality | PASS for independent PNG preview; final delivered pair pending orchestrator | Rendered the reviewed source independently to `/private/tmp/uc13-independent-review/uc-13-check-assignment-rule.png` with `PLANTUML_LIMIT_SIZE=30000` and visually inspected the whole image. Title, all seven paths, open/filled arrow distinctions, nested fragments and activation ends are present, legible and unclipped. No overlap was found. The orchestrator must verify final PNG/SVG correspondence and inspect the final pair after any revision. |

## UC-04 reference integration

The inspected UC-04 draft retrieves candidate rule data and same-shift ambulance slots before the reference (lines 168–175). Its `ref` covers every common lifeline: Manager, RosterBoundary, RosterControl, AssignmentRuleControl, Staff and Ambulance (lines 179–183). The exact UC-13 title, `result = checkAssignment(proposal, context)`, and `baseControl = roster:RosterControl` binding are present. There is no duplicate standalone invocation or repeated hours decision in the caller. The caller branches on prevention/cancellation versus allowed, refreshes candidate retrieval/display after rejected proposals, and contains persistence inside the allowed operand. This integration check does not replace the complete independent UC-04 review.

## Diagram defects

None found in the reviewed UC-13 source. No author correction requested.

## Source gaps and declared assumptions

- **I02 remains open:** how pending assignments contribute to conflict, rest and weekly hours is unspecified. The diagram accepts context and the contract retains an abstract PolicyContext; it does not choose a concrete policy.
- **I08 remains a design assumption:** operations, boundary/Control classes, value types and the formal caller role are proposed detailed design because the class table defines responsibilities and attributes without operations.
- **I15 remains a delivery assumption:** the on-screen warning is an asynchronous presentation through the boundary. No email, delivery guarantee or acknowledgment mechanism is invented. The explicit decision-received reply acknowledges the Manager's submission, not a saved assignment.
- The distinct `cancelled` result is a justified representation of source alternative 5a2, whose outcome is more specific than the postcondition's allowed/prevented summary.

Any change to the reviewed source, relevant contract signatures, UC-04 reference binding or authoritative documents requires targeted revalidation. The final batch report should combine this source pass with the orchestrator's final render gate and continue to disclose I02.
