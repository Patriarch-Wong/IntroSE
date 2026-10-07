# Regenerated sequence diagrams

This batch covers UC-02 Manage Availability, UC-04 Build Weekly Roster and UC-13 Check Assignment Rule only. Each standalone diagram covers the main scenario and every documented alternative. Earlier batches are preserved.

| Use case | Zoomable diagram | Image | Editable source |
|---|---|---|---|
| UC-02 Manage Availability | [SVG](uc-02-manage-availability.svg) | [PNG](uc-02-manage-availability.png) | [PlantUML](uc-02-manage-availability.puml) |
| UC-04 Build Weekly Roster | [SVG](uc-04-build-weekly-roster.svg) | [PNG](uc-04-build-weekly-roster.png) | [PlantUML](uc-04-build-weekly-roster.puml) |
| UC-13 Check Assignment Rule | [SVG](uc-13-check-assignment-rule.svg) | [PNG](uc-13-check-assignment-rule.png) | [PlantUML](uc-13-check-assignment-rule.puml) |

Use SVG when zooming through the full UC-04 allocation flow. All labels remain at the authored font size; diagrams are not scaled down to fit a page.

## Sources and scope

Behaviour follows [Final Use Case.docx](../../Final%20Use%20Case.docx); domain responsibilities follow [Main Class Table.docx](../../Main%20Class%20Table.docx). The current [project instructions](../../AGENTS.md) take precedence over historical diagrams and older wording in the [orchestration plan](../../SEQUENCE_DIAGRAM_ORCHESTRATION_PLAN.md). The plan was executed for these three use cases with Astra xhigh authors, independent validators, and central contract reconciliation. The supporting class diagram does not override the class table.

A fresh [source snapshot](evidence/source-snapshot.json) records the authoritative files. No Word source, project instructions or existing diagram batch was changed. Current text excerpts are preserved in `evidence/` for traceability.

## Design and validation

The [shared interaction contract](interaction-contract.md) defines participant labels, operation signatures, result conventions, illustrative SQL mappings and visual conventions. All control classes use `Control` and `«control»`. SQL arrows include a short purpose comment and explicit business-effect write acknowledgments. Replies use dashed open arrows; messages are not numbered.

The [validation report](validation-report.md) records source-step coverage, independent review, corrections and final visual checks. Detailed author and validator notes are in `reviews/`. [Delivery verification](evidence/delivery-verification.json) records the nine diagram-file hashes, dimensions and automated consistency checks.

The [model issues](model-issues.md) distinguish diagram defects from source gaps. Pending-assignment calculations remain policy inputs; the physical source for date-specific candidate location is unspecified; UC-04's unavailable-ambulance alternative ends at its stated warning without an invented continuation. These diagrams are validated at that declared abstraction, not a finalized executable persistence design. SQL, UI/Control operations and identifiers are proposed detailed design.

## Reproduction

The exports use PlantUML 1.2026.8 and Java. Run from this directory:

```sh
PLANTUML_LIMIT_SIZE=30000 plantuml -checkonly *.puml
PLANTUML_LIMIT_SIZE=30000 plantuml -tpng *.puml
PLANTUML_LIMIT_SIZE=30000 plantuml -tsvg *.puml
```

The raised limit prevents long diagrams being clipped by the default 4096-pixel cap. All style and essential definitions are embedded in each source; no external include is required.
