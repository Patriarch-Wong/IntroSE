# UC-04 simplified revision

[View SVG](uc-04-build-weekly-roster.svg) · [View PNG](uc-04-build-weekly-roster.png) · [Edit PlantUML](uc-04-build-weekly-roster.puml)

A shorter redraw of UC-04 Build Weekly Roster. It keeps the database and the persistence convention in `AGENTS.md`, but has **one loop and no nested loops**: 52 messages (previous revision: 82), PNG 1386 × 3012 (previous: 2012 × 4973).

## Scope

Main success scenario and alternatives 1a, 4a, 5a, 8a and 9a, in one diagram. UC-13 is invoked through the same `ref` and participant binding as the previous revision ([UC-13 diagram](../2026-10-07_21-35-15/uc-13-check-assignment-rule.svg)).

## How it is kept short

- **One allocation loop.** Each iteration handles one Manager request, chosen in a flat `alt`: select shift and slot (3–4, 4a), choose a role to fill or replace (5–6, 5a3), remove an assignment (5a1–5a2), or select a candidate (7–9, 8a, 9a). A guard only opens once its earlier step has been shown, so a slot with no eligible ambulance cannot reach candidate allocation.
- **8a retry without a second loop.** A prevented or cancelled UC-13 result saves nothing and leaves the candidates shown; the next loop iteration is the Manager selecting again (return to step 6).
- **Important transactions in full; minor ones skimmed.** Assignment save, removal, ambulance eligibility, candidate retrieval and the UC-13 call are shown with their SQL. Minor steps are omitted: per-slot `staffingStatus` and per-candidate `assignedHours` entity loops (now supplied by the read projections), the `Assignment`/`AuditEntry`/`RosterSlot` entity preparation calls, and audit entries for draft creation and the final draft save.
- **Upserts.** Fill and replace use one `INSERT ... ON CONFLICT` each for the slot and the assignment, instead of `alt` branches with separate INSERT/UPDATE.

## Step mapping

| Source step | Diagram |
|---|---|
| 1–2 | `openAllocation`, WeekPlanningView read, `opt` empty draft INSERT |
| 1a | `alt` published operand; use case ends |
| 3–4, 4a | Loop operand "selects a shift and slot"; inner `alt` on empty result |
| 5–6, 5a3 | Loop operand "chooses an ambulance and role"; `replacedAssignmentId?` marks replacement |
| 5a1–5a2 | Loop operand "removes an assigned staff member"; DELETE + audit |
| 7–8 | Loop operand "selects a candidate"; rule-context read, `ref` UC-13 |
| 9 | `alt` allowed operand: slot upsert, assignment upsert, audit insert |
| 8a | `alt` prevented/cancelled operand; nothing saved |
| 9a | `alt` ambulance-unavailable operand; nothing saved, warning shown |
| 10 | Repeat = the loop; `saveDraft` after it |

## Assumptions and open issues

- SQL is illustrative PostgreSQL. `WeekPlanningView`, `CandidateOption` and `StaffRuleContext` are proposed read projections as in the [previous interaction contract](../2026-10-07_22-28-47/interaction-contract.md); `CandidateOption` now also supplies `assignedHours`, and `StaffRuleContext` also supplies the selected ambulance's current availability.
- **9a (I07):** the source gives no detection mechanism. This revision assumes the unavailability is detected from the availability read at assignment time (step 9) and only shows the warning; no polling, callback or recovery is added.
- Upserts assume `slotID` and `assignmentID` are keys; for a new assignment `:assignmentId` is a new key, for a replacement it is the replaced assignment's key.
- Draft creation and the final draft save are not separately audited here (skimmed); assignment saves and removals are audited, which covers the changes to the draft.
- I02 (pending assignments' effect on hours) and I03 (date-specific location source) remain open, see [model issues](../2026-10-07_22-28-47/model-issues.md).

## Reproduction

PlantUML 1.2025.4 with Java:

```sh
PLANTUML_LIMIT_SIZE=30000 plantuml -tpng uc-04-build-weekly-roster.puml
PLANTUML_LIMIT_SIZE=30000 plantuml -tsvg uc-04-build-weekly-roster.puml
```

## Presentation draft

[View PNG](uc-04-build-weekly-roster-presentation.png) · [Edit PlantUML](uc-04-build-weekly-roster-presentation.puml)

A shorter sample for presentation, modelled on the team's main-scenario reference layout: 38 messages, one loop, no nested loops. It shows the main success scenario (steps 1–10) and alternative 8a only; the full diagram above keeps 1a, 4a, 5a and 9a.

- Steps 3–5 sit in `opt [starting a new allocation]` inside the loop. After an 8a rejection the next iteration skips that `opt` and goes straight back to the candidate list (step 6), so the retry needs no second loop.
- Compared with the reference: no message numbers, SQL on Control→Database arrows instead of entity-to-database routing, replies to the Manager as outcomes rather than `display...()` calls, and an audit entry for each saved assignment. The reference's single `BEGIN ... COMMIT` draft-save transaction is not assumed.
- The "not yet published" condition (1a) is a precondition note in this draft.
