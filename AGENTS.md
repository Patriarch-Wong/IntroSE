# UML modelling rules

## Sources and scope

- Use UML 2.5 notation and semantics for all UML models and diagrams.
- `Final Use Case.docx` defines behaviour and `Main Class Table.docx` defines the domain baseline. Treat supplied diagrams as design references to evaluate, not as authority over these sources or rules.
- State the scenario scope in accompanying documentation. A complete sequence model for a use case must cover the main and documented alternative scenarios; a limited sample must be identified as such. Coverage may span several diagrams.

## Deliverables

- For every UML diagram created or revised, deliver both a PlantUML `.puml` source and an editable draw.io `.drawio` copy in the same directory with the same base filename. Include referenced subinteractions and supporting class diagrams.
- Use editable native shapes, connectors and text in draw.io; an embedded image or PlantUML block alone is insufficient.
- Keep both formats consistent in content and UML semantics. Check their rendered layouts and matching content before delivery, and update both whenever a diagram changes.

## Participants and messages

- Use `Control` rather than `Controller`, with the `«control»` stereotype consistently in class and sequence diagrams.
- Use consistent `instance:Class` labels and canonical class names across diagrams. Identify boundary, Control and entity roles, and include only participating lifelines.
- Include boundary and Control participants in detailed actor-facing sequence diagrams. Route human requests and responses through a boundary; internal referenced interactions need only their participating lifelines.
- Keep message labels concise: operations with relevant arguments for calls, meaningful values or outcomes for replies, and outcomes rather than human-owned methods for information presented to actors.
- Use solid lines with filled arrowheads for synchronous calls, solid lines with open arrowheads for asynchronous messages, and dashed lines with open arrowheads for replies. Choose notification semantics from the interaction; a notification is not automatically a reply.
- Show activation bars for the executions being detailed and check their closure on every branch.

## Behaviour and control flow

- Preserve the use case's processing and validation order, alternatives, warnings and outcomes within the stated scope.
- Use `alt` for choices, `opt` for conditional additions and `loop` for repetition. Guards must express actual conditions.
- Rejection and cancellation must not reach success-only checks, writes or confirmations. Shared continuation is valid only for branches that can reach it; an error response or an `alt` branch does not itself terminate the interaction.
- Use `break` only for genuinely terminating alternatives. It skips the remainder of its enclosing interaction fragment, must cover that fragment's lifelines, and is not a programming-language `continue` or an escape from multiple enclosing loops.
- Show retries through actual loop or interaction structure; a note saying “return to step” is insufficient.
- Define every `ref` interaction and supply its inputs, results and participant bindings. Cover all lifelines common to the caller and referenced interaction. Use a reference only where the interaction executes, without duplicating its invocation or checks in the caller.
- Trace success, rejection, cancellation and retry paths before delivery, including continuation across references.

## Persistence

- Show the database and core persistence requests and replies when persistent data is read or written. Keep entity lifelines separate from the database.
- Follow the project routing convention: boundary calls Control; Control coordinates domain entities and sends persistence requests to the database; database replies return to the requesting Control.
- Retrieve data before checks that depend on it. Show required writes and their confirmed outcomes before reporting persisted success through the boundary to the actor.
- Use concise logical persistence-operation labels. Query replies identify returned data; write replies identify the confirmed persisted effect, rather than generic success or a bare row count.
- Keep SQL in non-rendered source comments. Persistence acknowledgments must be supported by the SQL or documented persistence contract. Do not invent transaction, caching or recovery guarantees; record necessary design assumptions.

Notation reference: [OMG UML 2.5](https://www.omg.org/spec/UML/2.5/PDF), clauses 17.4, 17.6 and 17.7. Naming, stereotypes, persistence routing and open-arrow replies are project conventions.
