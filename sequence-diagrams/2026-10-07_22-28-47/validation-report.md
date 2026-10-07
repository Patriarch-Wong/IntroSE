# UC-04 retry revision validation

The focused retry change passes compilation, path-trace and visual checks. This is a review of the current edit, not a new independent-agent validation or resolution of the baseline's open policies.

## Baseline and scope

GitHub `main` was checked against commit `647cee105db78da394c3275b50229ba096be7f76`; it matches the local baseline. Current UC-04 wording was read from `Final Use Case.docx`: alternative 8a requires no assignment save, presentation of the reason/outcome, and return to step 6. The domain baseline and UC-13 reference remain unchanged. Source hashes are in `source-snapshot.json`.

Only the candidate-attempt section changed. The initial roster read/create/display, published handoff, ambulance selection and no-eligible retry, removal path, outer allocation loop and final draft-save section match the baseline apart from comments/indentation.

## Behaviour and execution review

- Main 5 and replacement 5a3 enter one loop beginning at candidate retrieval/display (step 6).
- Each candidate attempt retrieves fresh candidate data and hours, accepts the Manager's selection, retrieves rule context, and invokes the existing UC-13 ref exactly once.
- Prevented/cancelled results perform no assignment, slot or assignment-change audit writes. The boundary presents the outcome before the next iteration's candidate retrieval.
- Only `allowed` with an ambulance that has not been made unavailable reaches the existing persistence block. New-slot creation, ambulance replacement, assignment insertion/replacement, staffing calculation and audit SQL retain their original statements and write acknowledgments.
- The candidate loop repeats after prevention/cancellation and ends after allowed results. The warning path still suppresses later allocation and final draft saving under the baseline's I07 assumption.
- A boundary execution is pending at each loop entry: initially the role-selection request, subsequently the rejected/cancelled candidate-selection request. Candidate presentation replies to and closes that request before the next human selection. The final allowed/warning reply closes at loop exit. Control executions close before subsequent refresh calls.
- The failure presentation remains asynchronous as in the baseline. Candidate-list responses, database responses and final saved/warning responses remain dashed open-arrow replies.

The source was expanded into 24 candidate-path traces: immediate allowed, prevented then allowed, cancelled then allowed, and repeated prevented/cancelled then allowed; each for new/existing/replacement slots, ending either in a save or the existing unavailable warning. Checks verify one candidate query and one UC-13 invocation per attempt, rejection-before-refresh order, no writes on rejected/cancelled/warning attempts, expected successful write counts, and balanced executions with no boundary execution spanning the next unrelated human selection. These checks do not execute the illustrative SQL or UC-13 internals. Results are in `retry-verification.json`.

## Artifact checks

PlantUML 1.2026.8 `-checkonly` passed. Both PNG and SVG were regenerated. PNG metadata contains the exact final PlantUML source; SVG viewBox 2013 × 4974 agrees with PNG 2012 × 4973 within renderer rounding. All five overlapping full-width PNG sections were inspected at readable scale: no clipping, overlapping labels or ambiguous frame boundaries were found. The SVG was independently rasterized with rsvg-convert and its retry and outcome sections were also inspected. Every distinct database statement is preserved, with only the duplicate drawn candidate query removed.

## Remaining limitations

The diagram still contains detailed SQL, removal and audit flows; this focused revision makes it shorter without claiming the whole interaction is now compact. Baseline I02/I03 policy/data gaps remain open. I07 remains unresolved: the use case specifies an unavailable-ambulance warning but does not establish its detection or subsequent workflow; the warning wording also requires clarification when a proposed slot/assignment has not yet been stored. No new resolution of those issues is asserted.
