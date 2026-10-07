# UC-02 Manage Availability — independent validation

Verdict: **PASS — semantic, source, syntax and corrected render checks complete.**

## Review baseline

Independently read the original UC-02 table from `Final Use Case.docx` and all classes in `Main Class Table.docx` with the bundled Python `docx` library. Source documents were not edited or re-exported. Applied the documents skill's read-only inspection workflow. Read current `AGENTS.md`, the orchestration plan, this batch's shared interaction contract and model issues, then reviewed the authored source without treating its coverage notes as authority.

Reviewed SHA-256 values:

- `Final Use Case.docx`: `eb3ea9945513199512283e80ab0a9f2a548a8cba1a46e63256eaa1e792b6c919`
- `Main Class Table.docx`: `26b415b0518b62b5b519d3e27bc92776ea221d45ae560c3141c6e38c0880d88e`
- Final reviewed `uc-02-manage-availability.puml`: `0fa0d6e317a0941d7926925ac9f45d22c388b192a90c60a696bc2601ae30f21e`
- Initial reviewed source before V02-01 correction: `9aef42cf9593ea5be49003111f8e82bb980365b55021e7d57557b224ca983c0d`

## Resolved finding history

**V02-01 — RESOLVED, visual layout, source lines 35–38.** Initially the precondition note anchored over `staffActor, planning` extended beyond the left PNG canvas edge and its left border was clipped. The parent reported the defect and this validator independently confirmed it. The author changed the anchor to `planningUI, planning`; the parent re-rendered PNG and SVG. The validator rechecked the final source hash, repeated standalone syntax checking (exit 0), and visually inspected the corrected full PNG: the complete note frame is now inside the canvas with a clear left margin. The SVG parses successfully with viewBox `0 0 1654 1420`; its corrected note text begins at x=243.806 with both lines inside the viewport. The change is solely note placement; the approved behaviour remains intact.

## Semantic and source coverage

| Gate | Result and evidence |
|---|---|
| Exact identity, trigger and precondition | PASS. Title is `UC-02: Manage Availability`; logged-in Staff opens their own availability through StaffPlanningBoundary. The five-week, 12-hour day/night constraint is stated. |
| Main 1–4 | PASS. Open → Control read → database records → boundary view/actor outcome precedes selection and submission. Each distinct actor form action has its own completed execution. The SELECT scopes the staff member and planning window. |
| Main 5 and alternative 5a1 | PASS. `loop each submitted roster week` calls `checkCutoff(week, submittedAt, cutoffPolicy)` before preparing each week's values. The result drives `lateFlag = isLate`; comments and the contract define late as after that week's Wednesday cutoff and restrict selected shifts to the current submitted week. Mixed on-time/late submissions are supported. |
| Main 6 and late storage postcondition | PASS. Every prepared shift is explicitly UPSERTed with availability and lateFlag before audit or notification. Both on-time and late values use the common persistence path. Only StaffAvailability and AuditEntry are written. |
| Main 7 and alternative 5a2 | PASS. AuditEntry prepares the payload, Control INSERTs it, and the database confirms the stored availability-change audit entry. The saved-state note accurately expresses manager visibility from shared persistence; no separate manager read is required by this scenario. |
| Alternative 5a3 | PASS. Only `opt one or more submitted weeks are late` sends a notification; it travels through NotificationBoundary to Manager and names the affected staff member and weeks for consideration. No notification is modelled as a reply or as unconditional on-time behaviour. |
| Main 8 and alternative 5a4 | PASS. Both paths reach the single saved confirmation after all required persistence. Control replies through the boundary to Staff. There are no rejection/cancellation branches or invalid success fall-throughs in this source use case. |
| Postconditions | PASS. Availability/late flags are stored inside the displayed planning window; audit is stored; manager visibility and late notification are represented; published assignments remain unchanged. UC-06 is correctly treated as a possible future amendment rather than an interaction executed here. |
| Domain roles and signatures | PASS. StaffAvailability prepares individual-shift records/late flags and AuditEntry prepares audit data, consistent with the original class table. UI/Control/notification classes, operations, payloads and identifiers are clearly declared proposed detailed design and match the central contract. Unused entities are omitted. |
| Persistence convention | PASS. All three database-directed arrows originate from Control and show SQL plus a short purpose comment immediately above it. SELECT returns `shiftAvailabilityRecords[]`; write replies describe the saved business effect without unsupported returned records or bare row counts. Named placeholders, unique key and PostgreSQL UPSERT assumptions are documented. |
| UML and presentation source | PASS. Rectangular stereotyped participants, human actors and database cylinder are used; Control naming is consistent. Synchronous calls are `->`, asynchronous notification messages are `->>`, and every reply is `-->>`. Activations balance, including the nested cutoff self-call and optional notification execution. Loops and the conditional `opt` match the behaviour. No numbering, legend, caption, scale directive or rendered author/review commentary appears. |
| Standalone syntax | PASS. Independent `PLANTUML_LIMIT_SIZE=30000 plantuml -checkonly` completed with exit 0. |
| Render inspection | PASS. V02-01 is resolved in the final exports. The corrected full PNG presents the complete path, SQL, loops, final replies and manager notification readably, with the complete precondition note frame visible. The corrected SVG's XML and note bounds were independently checked. |

## Source-policy gaps and assumptions, not diagram defects

- **I01:** The late alternative omits an explicit save despite requiring stored availability and late flags in its postconditions. The diagram reconciles this exactly as required by current AGENTS.md, saving before audit/notification; the reconciliation is documented outside the rendered diagram.
- **I13:** The exact Wednesday cutoff time/timezone is absent from the source. It remains a supplied policy input; the diagram makes no concrete clock-time claim.
- **I08:** The class table supplies responsibilities and attributes, not the proposed operations, UI/Control classes or physical schema. The central contract declares those design details and the diagram respects them.
- **I15:** Asynchronous presentation through NotificationBoundary is a declared delivery assumption. No email, durable delivery, acknowledgment, scheduler, transaction, rollback or atomic-save guarantee is invented.

PASS means source-faithful interaction design at these declared abstractions. No unresolved diagram defects remain; the documented source-policy inputs and proposed design assumptions remain explicit.
