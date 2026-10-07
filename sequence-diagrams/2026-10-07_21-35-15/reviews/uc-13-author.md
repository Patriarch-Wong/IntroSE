# UC-13 author coverage and self-review

Draft for independent validation. The standalone source covers the complete main scenario and every documented alternative of **UC-13 Check Assignment Rule**. The current `Final Use Case.docx` UC-13 table and all twelve rows of `Main Class Table.docx` were read directly with bundled Python `python-docx`; their extracted content matches this batch's evidence. No Word document was changed. `AGENTS.md`, the orchestration plan, the shared interaction contract and `model-issues.md` were read before drafting.

## Source-step coverage

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

## Proposed operations and assumptions

The source class table defines responsibilities and attributes without operations. Every operation below is proposed detailed design already registered in the shared contract.

| Owner | Signature used | Source responsibility |
|---|---|---|
| AssignmentRuleControl | `checkAssignment(proposal, context): RuleResult` | Coordinate this included use case and its ordered checks. |
| Staff | `isQualified(roles): Boolean` | Maintain recorded qualifications. |
| Staff | `hasShiftConflict(proposal, context): Boolean` | Check staff assignment conflicts. |
| Staff | `hasMinimumRest(proposal, context): Boolean` | Check staff rest. |
| Staff | `projectedWeeklyHours(proposal, context): Hours` | Derive weekly assigned hours. |
| Ambulance | `hasShiftConflict(proposal, context): Boolean` | Check same-shift slot conflicts. |
| RosterBoundary | `requestHoursDecision(projectedHours)` | Present the relevant base workflow's on-screen warning and obtain the decision. The synchronous call returns a `confirm` or `cancel` decision. |
| RosterBoundary | `submitHoursDecision(decision)` | Receive the Manager's explicit confirm/cancel action. Its actor-facing acknowledgment is `decision received`; it does not claim assignment persistence. |

`BaseControl` is a formal caller role, not a new domain class. The boundary and both Control roles follow the canonical instance names and stereotypes. No participant or operation addition is proposed beyond the current contract.

The base Control supplies retrieved `RuleContext`: qualifications, relevant commitments, shift timing including adjacent weeks, same-shift ambulance slots and workload policy. Replacement context excludes the replaced assignment where appropriate while retaining other commitments. Staff hours represent duty time, so a staff member fulfilling multiple roles in one duty is not counted more than once. `PolicyContext` leaves pending-assignment contributions unresolved under I02; the diagram makes no concrete policy or numerical calculation claim.

There is no database participant, SQL, write, audit or extra query: UC-13 has no database secondary actor, and every check uses context already retrieved by its caller. The design does not hide required persistence in entity calls. The base use case owns the permitted subsequent writes and candidate retry after prevention/cancellation.

The warning uses the declared asynchronous on-screen presentation mechanism (I15), with no email or delivery guarantee. `requestHoursDecision` remains pending while the related human decision is obtained. The Manager's separate request creates a short nested boundary execution and receives a dashed open-arrow acknowledgment; the boundary then returns the decision to the pending rules execution. No callback operation is invented, and no activation spans an unrelated user action.

## Self-review

- All five checks occur once and in source order: qualification, staff conflict, ambulance conflict, rest, weekly hours.
- Each blocking result is in an operand that contains no later checks or success-only continuation. All future checks are nested inside the corresponding valid operand. No `break` is needed.
- There are seven terminal paths: four prevention reasons, ordinary allowed, above-40 confirmed allowed and above-40 cancelled. Each returns exactly once to the caller.
- Manager participates only in the hours-decision branch. All human interactions traverse `RosterBoundary`; actor labels describe outcomes rather than `display...()` operations.
- Entity executions begin on calls and end on their replies. The rules execution has one common closure after the alternatives, reached after whichever exclusive terminal result occurs. The boundary's decision-submission execution closes before the pending prompt execution returns.
- All synchronous calls use `->`, asynchronous warning presentation uses `->>`, and replies use `-->>`. There is no automatic or manual message numbering.
- Rendered content has the exact use-case title, short messages and a behavioural precondition only. Source mappings, formal binding explanations and policy assumptions are kept here or in non-rendered comments.
- `PLANTUML_LIMIT_SIZE=30000 plantuml -checkonly` passed with exit status 0. A local PNG preview rendered successfully and was visually inspected: complete content, readable 14-point text, explicit stereotypes, clear nested fragments, no overlap or clipping. The final source uses the centrally revised `maxMessageSize 350` pixel setting. The orchestrator owns final PNG/SVG exports and independent validation.
