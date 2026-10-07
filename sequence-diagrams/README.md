# Sequence diagrams

## Latest versions

| Use case | Diagram | Editable source | Design and review notes |
|---|---|---|---|
| UC-02 Manage Availability | [SVG](2026-10-07_21-35-15/uc-02-manage-availability.svg) · [PNG](2026-10-07_21-35-15/uc-02-manage-availability.png) | [PlantUML](2026-10-07_21-35-15/uc-02-manage-availability.puml) | [Batch notes](2026-10-07_21-35-15/README.md) |
| UC-04 Build Weekly Roster | [SVG](2026-10-07_22-28-47/uc-04-build-weekly-roster.svg) · [PNG](2026-10-07_22-28-47/uc-04-build-weekly-roster.png) | [PlantUML](2026-10-07_22-28-47/uc-04-build-weekly-roster.puml) | [Retry revision](2026-10-07_22-28-47/README.md) · [Validation](2026-10-07_22-28-47/validation-report.md) |
| UC-13 Check Assignment Rule | [SVG](2026-10-07_21-35-15/uc-13-check-assignment-rule.svg) · [PNG](2026-10-07_21-35-15/uc-13-check-assignment-rule.png) | [PlantUML](2026-10-07_21-35-15/uc-13-check-assignment-rule.puml) | [Batch notes](2026-10-07_21-35-15/README.md) |

UC-04's latest revision moves candidate retrieval/display into one retry loop matching alternative 8a's return to step 6. UC-13 is still invoked through the existing interaction reference. Earlier timestamped batches are historical versions.

## Source and reference documents

- [Final Use Case.docx](../Final%20Use%20Case.docx): behavioural authority.
- [Main Class Table.docx](../Main%20Class%20Table.docx): domain baseline.
- [AGENTS.md](../AGENTS.md): current UML and project conventions.
- [Project description](../TP2_INF2001%20Team%20Project%20Description%20-%20Ambulance%20Services.pdf): project requirements/reference.
- [Supporting class diagram](../class-diagrams/main-class-table/domain-class-diagram.svg) and [modelling rules](../class-diagrams/main-class-table/rules.md).
- [Orchestration plan](../SEQUENCE_DIAGRAM_ORCHESTRATION_PLAN.md): historical execution plan; current project instructions take precedence.
- [Current interaction contract](2026-10-07_22-28-47/interaction-contract.md) and [open model issues](2026-10-07_22-28-47/model-issues.md).

Historical validation records describe their own source snapshots. They do not validate later edits. UC-04 alternative 9a's unavailability detection/continuation remains unresolved; see the latest revision's validation report.
