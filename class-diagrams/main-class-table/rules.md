# Ambulance rostering rules

These rules accompany [the domain class diagram](domain-class-diagram.puml). The 12 domain classes and original key attributes come from [Main Class Table.docx](../../Main%20Class%20Table.docx). Behaviour follows [Final Use Case.docx](../../Final%20Use%20Case.docx). The crew composition and 11-hour service limit come from page 1 of [the project description](../../TP2_INF2001%20Team%20Project%20Description%20-%20Ambulance%20Services.pdf).

The simplified diagram shows the reference's 12 classes, 33 attributes in class compartments, two association-end properties and four enumerations. Operations, additional audit fields and the explicit roster-to-shift association have been omitted. Business rules remain here so the diagram can focus on domain structure.

## Accounts and retained records

- `staffID` is unique across all accounts. `{id}` identifies it as an identifier in the diagram; this rule supplies the cross-account uniqueness constraint.
- Each `Staff` profile belongs to exactly one `UserAccount`. An account may have no staff profile, for example an administrative account.
- A deactivated account cannot log in or be an assignment candidate. Candidate eligibility also requires a staff profile; qualifications, availability, conflicts and rest are checked separately for the proposed assignment.
- Deactivation retains the account, staff profile and past records. Existing draft or published assignments are brought to the manager's attention for amendment.
- Composition expresses exclusive ownership. Retained owners and their historical parts must not be deleted as part of deactivation. Removing a current assignment must preserve its audit history.

## Availability and preferences

- Each staff availability record concerns exactly one staff member and one shift. Each ambulance availability record concerns exactly one ambulance and one shift.
- There is at most one current availability record for each owner/shift pair.
- Staff can submit availability within the five-week planning period. The Wednesday cut-off is evaluated for each submitted roster week.
- Late submissions or changes set `lateFlag` for the affected availability; managers are notified. Published assignments remain unchanged until a manager amends the roster.
- There is at most one `JobPreference` per staff member and roster week. A preference may contain no shift type and zero or more stations. Blank input means no preference.
- Preferred stations express preferences, not the station actually assigned to a duty.

## Rosters, shifts and slots

- There is one weekly roster per roster week once that roster is created.
- Each weekly roster concerns all 14 shifts: one day shift and one night shift for each of its seven days. Publication checks include shifts with no slots. This is a business constraint, not an additional association in the simplified diagram.
- A shift can exist before its weekly roster, for availability planning. The date and shift type identify a single shift occurrence.
- Each `RosterSlot` belongs to exactly one weekly roster and represents exactly one ambulance in exactly one shift. Its shift's start date must fall within the roster week.
- An ambulance occupies at most one slot in a shift. A staff member holds at most one assignment in a shift. Several roles performed by the same person in the same slot belong to one assignment.
- Each assignment belongs to exactly one slot and exactly one staff member.
- A slot may have zero or one station. The station on the slot supplies the location for assignments in that slot; it does not establish a permanent staff or ambulance base.
- An empty draft can have no slots. A slot exists only once its ambulance has been selected; a temporary unallocated UI row is not a domain `RosterSlot` in this model.

## Crew and assignment rules

- A complete crew covers one Driver position, one Care Assistant position and two Paramedic positions, with two distinct people filling the Paramedic positions.
- A Driver qualified as a Care Assistant may cover both positions in one assignment. Otherwise, the positions require separate people. A complete crew therefore requires three or four people.
- Each value in `Assignment.fulfilledRoles` must be present in the assigned staff member's `qualifiedRoles`. The only shared crew positions allowed by this model are Driver and Care Assistant.
- `missingRoles` lists unfilled role positions. `{nonunique}` permits repeated values, such as two missing Paramedic positions.
- `staffRequired` means the additional people needed to fill the missing positions, allowing qualified Driver/Care Assistant dual coverage. The dual-role case requires an eligible person with both qualifications.
- `isFullyStaffed` is true when every required crew position is covered by confirmed assignments. A pending assignment does not fill a vacancy.
- An assignment fills a role only when it is confirmed and includes that role.
- A pending standby assignment becomes confirmed when the selected staff member confirms. Ordinary manager allocations and direct reassignments count as confirmed; they do not use the standby confirmation workflow.
- Declining or withdrawing a pending request removes that pending assignment. Rejecting or removing an existing duty releases its roles. Rejection requires a reason; roster amendments record the stated reason.
- Staff need at least 12 hours of rest between duties. Less than 12 hours blocks a proposed assignment.
- Exceeding 40 assigned hours in the roster week produces a warning. The manager may confirm or cancel; exceeding 40 hours alone does not block the assignment.
- Ambulance allocation requires an active ambulance that is available and has no other slot in the same shift. Later unavailability flags the affected slot for amendment.

## Publication and amendment

- Publication requires at least six fully staffed, available ambulances in every shift of the week. Allocation also requires those ambulances to be active.
- A failed publication check leaves the roster as draft and identifies the affected shifts and missing roles.
- A published roster stays published during amendment. A resulting shortfall is flagged; it does not revert the roster to draft.

## Dates and derived values

- `Shift.date` is its start date. The shift ends 12 hours after its start, on the following date for an overnight shift. `startTime`, `endTime` and `/durationHours = 12` must agree.
- `/weeklyAssignedHours` and `/monthlyAssignedHours` are summaries for the selected roster week and calendar month. The report context supplies the period; operation signatures are omitted from this domain view.
- Workload is derived from assignments and shift duration. A person covering Driver and Care Assistant in one duty is counted once, not once per role.
- `/scheduledUtilisation` is a summary for the selected day or week, supplied by the report context.
- Ambulance utilisation is scheduled in-service hours divided by total shift hours for the selected period. A rostered 12-hour shift contributes at most 11 in-service hours.
- Fleet utilisation for a shift is the number of rostered ambulances divided by the active fleet size. This ratio is distinct from an individual ambulance's service-hour utilisation.

## Audit entries

- Every audit entry has exactly one acting user and a timestamp. `actingUser` is a private association end at `UserAccount`.
- The simplified diagram keeps the original class-table attributes. How an audit entry identifies the event and affected record is left for detailed design.
- Previous and new values are optional and should identify the changed fields when present. Reasons are recorded where required by the use case.
- Audit records are retained independently of removed assignments. The acting user is the person performing the action, not necessarily the person whose account or assignment changed.

## Modeling choices and details still to specify

- Association multiplicities, attribute types and ownership are modeling choices based on the class responsibilities. `preferredStations` and `actingUser` are private association ends. Navigability is left unspecified. Operations are omitted from this view.
- The optional staff profile and unique current availability/preference records follow the reference. Week membership is expressed as a constraint on shift dates rather than another link.
- The sources do not fully specify whether pending assignments reserve staff time or contribute to workload. The one-assignment-per-shift constraint in this diagram includes pending assignments; their treatment in rest and workload calculations remains to be agreed.
- Allocation of an overnight duty's hours across a calendar-month boundary, the precise scheduled in-service-hour calculation, and the result of fleet utilisation when the active fleet is empty remain to be specified.
- The source table mentions candidate location for a date but does not define where it comes from for staff without a slot on that date. This diagram does not invent a permanent staff-station relationship.
