# UC-10 and UC-11, scoped down

Revised drafts of UC-10 Review Staff Workload and UC-11 View Ambulance Utilisation, following the prof's review on 2026-10-08. `Final Use Case.docx` is not changed yet; once these drafts are agreed, the two tables are copied over.

What changed, in short: each main scenario now covers only what the Essential requirements need, calculation rules are referenced in one line instead of being spelled out in the steps, and only real alternatives are kept.

## UC-10: Review Staff Workload

| Field | Details |
|---|---|
| **ID** | UC-10 |
| **Name** | Review Staff Workload |
| **Description** | The manager checks how many hours each staff member is assigned in a roster week so jobs can be spread fairly. |
| **Primary Actor** | Manager |
| **Secondary Actor(s)** | Database |
| **Related Requirements** | FR-19, FR-26, NFR-03 |
| **Relationships** | None |
| **Preconditions** | The manager is logged in. |
| **Trigger** | The manager opens their landing page to review staff workload. |
| **Main Success Scenario** | **1.** The manager opens their landing page and selects a roster week.<br>**2.** The system retrieves the assignments for that week and displays each staff member's assigned hours.<br>**3.** The system lists the three staff members with the lowest assigned hours (all staff tied at the third value are included, per FR-19) and highlights every staff member above 40 hours. |
| **Alternative Scenarios** | **1a.** The manager selects the monthly view.<br>&nbsp;**1a1.** The system displays each staff member's total assigned hours for the calendar month.<br>**2a.** No assignments exist for the selected week.<br>&nbsp;**2a1.** The system displays that no hours are assigned for that week. |
| **Postconditions** | The manager has seen the weekly (or monthly) assigned hours of each staff member. No data is changed. |
| **Priority** | High |

### What was cut from UC-10 and why

| Before | After | Why |
|---|---|---|
| Main steps 3 and 4 listed the lowest three and the over-40 highlight separately | One step 3 | Both are the same display of the same data (FR-19). |
| Alternative 3a spelled out tie handling | One clause in step 3, pointing to FR-19 | Prof asked for rules as one-liners, not separate branches. The rule itself stays in FR-19. |
| Main step 5 made the monthly view part of every run | Alternative 1a | FR-26 is Desirable, not Essential, and the manager does not always need it. This matches how UC-09 handles its own monthly view (3a). |
| No empty case | Alternative 2a added (one line) | A week with no assignments is a real outcome the page must show. |
| Trigger "wants to review staff workload and identify workload imbalances" | "opens their landing page to review staff workload" | Prof asked that every trigger name who starts the use case and what they do. |

## UC-11: View Ambulance Utilisation

| Field | Details |
|---|---|
| **ID** | UC-11 |
| **Name** | View Ambulance Utilisation |
| **Description** | The manager views how ambulances and the fleet are utilised and which roster slots still need staff. |
| **Primary Actor** | Manager |
| **Secondary Actor(s)** | Database |
| **Related Requirements** | FR-20, FR-21, FR-22 |
| **Relationships** | None |
| **Preconditions** | The manager is logged in. |
| **Trigger** | The manager opens the utilisation page to review ambulance utilisation and manpower shortfalls. |
| **Main Success Scenario** | **1.** The manager opens the utilisation page and selects a day or a week.<br>**2.** The system retrieves the roster and ambulance records for that period, then calculates and displays the utilisation of each ambulance and of the fleet for each shift (calculation rules in FR-20 and FR-21).<br>**3.** The system lists the roster slots requiring manpower, with the missing role and number of staff required. |
| **Alternative Scenarios** | **3a.** No slot requires manpower.<br>&nbsp;**3a1.** The system displays that all slots are fully staffed. |
| **Postconditions** | The manager has seen ambulance and fleet utilisation and the slots requiring manpower. No data is changed. |
| **Priority** | High |

### What was cut from UC-11 and why

| Before | After | Why |
|---|---|---|
| Steps 2 and 3 wrote out both formulas (in-service hours ÷ shift hours, at most 11 h per 12-hour shift; rostered ambulances ÷ active fleet) | One step 2 that names FR-20 and FR-21 for the calculation | Same reason as UC-10: the use case says *what* the manager sees, the requirement keeps the formula. The sequence and class diagrams can still use the formulas from FR-20/FR-21. |
| Four main steps | Three | Ambulance and fleet utilisation come from the same retrieval and are shown on the same page. |
| Alternative 4a ("fully staffed\*") | 3a, stray asterisk removed | Renumbered after the merge. |

## Prof's points checked against both use cases

| Point | UC-10 | UC-11 |
|---|---|---|
| Who triggers it | Manager, from the landing page | Manager, from the utilisation page |
| Manager as secondary actor only for notifications | Not used | Not used |
| Email/SMS server needed | No notification is sent | No notification is sent |
| Include/extend | None; both are read-only views with no shared validation | None |
| Validation depth | No validation (read-only) | No validation (read-only) |
| Every actor in the description is on the use case diagram | Manager, Database | Manager, Database |

## Consistency with UC-02 and UC-04 (draft PR #1)

- Both use the same actors as UC-02 and UC-04: **Manager** as primary, **Database** as secondary. If the team decides to drop Database as an actor after the prof's comments, it should be dropped from every use case at once, not just these two.
- The 40-hour rule appears in three places with different roles, which the diagrams should keep distinct: UC-04 warns at the point of assignment (FR-11), UC-10 only highlights it on the landing page (FR-19), and the pre-publication summary lists it as advisory.
- "Roles requiring manpower" is shown in UC-04 step 2 for the draft being built, and in UC-11 step 3 across any day or week. Same data, same wording, so the sequence diagrams can share one query.
- Neither use case writes data, so neither has an audit entry, cut-off or late-flag logic.

## Open question for the team

- Should the manager's monthly workload view (FR-26, Desirable) stay in UC-10 at all? It is kept as alternative 1a here. Dropping it would leave UC-10 as three steps with a single empty-week alternative.
