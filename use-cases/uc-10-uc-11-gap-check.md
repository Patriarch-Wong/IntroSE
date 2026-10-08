# UC-10 and UC-11 gap check

The original UC-10 Review Staff Workload and UC-11 View Ambulance Utilisation in `Final Use Case.docx` stay as the baseline. This note lists only the gaps found when checking them against three things: the prof's 2026-10-08 review, the final FR list in the requirements register, and the neighbouring use cases. Each gap comes with the smallest fix that closes it.

## Already fine

- **Trigger and actors:** the Manager starts both as the primary actor. Neither sends a notification, so the points about Manager as a secondary actor and an Email/SMS server do not apply.
- **Include/extend:** neither uses one, which is right for two read-only views.
- **Validation:** none is needed because nothing is written.
- **Requirements:** UC-10 covers FR-19, FR-26 and NFR-03, and UC-11 covers FR-20, FR-21 and FR-22. Both match the final FR sheet.

## Gaps

| # | Use case | Gap | Minimal fix |
|---|---|---|---|
| 1 | UC-10 | Step 2 shows hours "for the roster week", but no step says which week. In UC-09 the staff member selects one. | Change step 1 to "The manager opens their landing page **and selects a roster week**." Alternatively, state the default, for example "the week currently being planned". |
| 2 | UC-10 | The use case has no outcome for a week that has no assignments yet, which is likely early in planning. UC-09 covers the same case as 2a. | Add **2a.** No assignments exist for the selected week. **2a1.** The system displays zero assigned hours for every staff member. |
| 3 | UC-11 | Alternative 4a1 ends in "fully staffed\*", but the asterisk has no footnote. | Remove the asterisk, or add the footnote it was meant to point to. |
| 4 | UC-11 | Step 2 says "the roster" without saying whether draft weeks count or only published ones. This changes the numbers, and the sequence diagram has to pick one. | Change step 2 to "retrieves the **published and draft** roster" (or "the published roster") for that period. |
| 5 | UC-11 | The use case has no outcome for a period with no roster at all, such as a week that is not yet planned. Steps 2–4 would then have nothing to calculate. | Add **2a.** No roster exists for the selected period. **2a1.** The system displays that no roster is available for that period, and the use case ends. |
| 6 | Both | The prof said every actor in a description must appear on the use case diagram. Both use cases list **Database** as a secondary actor. | Check that Database is drawn on the use case diagram and linked to UC-10 and UC-11, the same way as for UC-02 and UC-04. |

## Consistency notes for other diagrams

These are not gaps in UC-10 or UC-11, but they affect how the other diagrams line up with them.

- UC-10 highlights staff above 40 hours on the landing page (FR-19). The warning at the point of assignment (FR-11) belongs to UC-04, but UC-04's description does not mention it yet. When UC-04 is revised, both should use the same 40-hour wording.
- UC-11 step 4 and UC-04 step 2 both show "roles requiring manpower". Keeping the same wording in both lets the sequence diagrams share one query.
