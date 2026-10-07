# Model issues and reconciliations

Scope: UC-02, UC-04 and UC-13. Behaviour and domain baseline were freshly read from the current Word documents; no source documents are edited. These are source limitations or declared design assumptions, separate from diagram defects.

| ID | Source and affected use cases | Treatment | Status |
|---|---|---|---|
| I01 | UC-02 alternative 5a omits save; postconditions require stored availability and late flags | Save all submitted availability with per-week late flags before audit and late notification; never modify published assignments. | Reconciled with postconditions and AGENTS.md |
| I02 | UC-04 candidate hours; UC-13 conflict/rest/hours | Pending assignments' contribution is not fully specified. Keep PolicyContext abstract; no concrete pending policy or numerical calculation is claimed. | Open policy; design remains conditional on supplied context |
| I03 | UC-04 step 6; Station responsibility | Candidate location for a date has no complete physical source. CandidateOption exposes a supplied datedLocation field, never substitutes station preference. | Open data source; illustrative SQL is not implementation-ready |
| I07 | UC-04 alternative 9a | Warning is required but detection source and continuation are unspecified. Preserve warning and terminate the shown affected path without successful allocation/draft-save claim. Do not invent polling, callback, recovery or new writes. | Open continuation |
| I08 | Main Class Table omits operations and UI/control classes | All boundary/Control classes, operation signatures, DTOs, physical SQL mappings and identifiers are proposed detailed design. | Declared assumption |
| I13 | UC-02 Wednesday cutoff | Week-specific cutoff is checked, but exact time/timezone is a policy parameter; no clock time is invented. | Open policy detail |
| I14 | UC-04 suitable candidate list | Up to three suitable candidates are shown; ranking and tie selection are unspecified. | Retain source abstraction |
| I15 | Notifications and warning delivery | Asynchronous boundary presentation events; no email, scheduler, durable delivery or acknowledgment mechanism assumed. | Declared mechanism assumption |

No UML diagram or physical schema implementation can settle I02, I03 or I07 from these sources alone. Passing diagram review means source coverage and safe control flow at this declared abstraction, not unconditional implementation readiness. Other use cases and unrelated historical issues are outside this batch.
