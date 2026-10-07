# UC-04 candidate retry revision

[View SVG](uc-04-build-weekly-roster.svg) · [View PNG](uc-04-build-weekly-roster.png) · [Edit PlantUML](uc-04-build-weekly-roster.puml)

Based on GitHub `main` commit `647cee105db78da394c3275b50229ba096be7f76`, confirmed using `git ls-remote`. The source is the UC-04 PlantUML in the `2026-10-07_21-35-15` batch. This revision does not use the draw.io conversion. Earlier batches remain available.

The complete UC-04 main scenario and alternatives remain in one diagram. Steps 6–9 now share one candidate-attempt loop: retrieve/display candidates, select a candidate, invoke UC-13, then save or present the unsuccessful outcome. Alternative 8a repeats from candidate retrieval at step 6 without a second copy of the query, hours calculation or display. The allowed-and-available persistence conditions share one guard.

The diagram has 82 messages, 17 control-flow frames and a maximum nesting depth of seven, compared with 89, 19 and eight in the baseline. Its PNG is 2012 × 4973, compared with 2037 × 5445. The readable font size is unchanged.

The referenced [UC-13 interaction](../2026-10-07_21-35-15/uc-13-check-assignment-rule.svg) and its participant binding are unchanged. SQL mappings and operation signatures remain in the [interaction contract](interaction-contract.md). The [validation report](validation-report.md) describes scope and checks.

Alternative 9a still uses the baseline's documented warning exit. Its detection, continuation, and applicability to a not-yet-stored slot remain unresolved source/design questions; this retry edit does not settle them. No Word sources were changed.

## Reproduction

Use PlantUML 1.2026.8 with Java, running each format separately:

```sh
PLANTUML_LIMIT_SIZE=30000 plantuml -checkonly uc-04-build-weekly-roster.puml
PLANTUML_LIMIT_SIZE=30000 plantuml -tpng uc-04-build-weekly-roster.puml
PLANTUML_LIMIT_SIZE=30000 plantuml -tsvg uc-04-build-weekly-roster.puml
```

The related source documents, modelling contract, review notes and evidence accompany the diagram artifacts in the repository.
