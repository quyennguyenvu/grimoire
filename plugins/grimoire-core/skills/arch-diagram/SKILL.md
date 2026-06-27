---
name: arch-diagram
description: Produce software-architecture diagrams as code — pick the right notation for the question (C4 context/container, sequence, class, ER, state, flowchart, deployment, roadmap) and emit clean, renderable Mermaid (default) or PlantUML (fallback) source in a Markdown document. Use when the user wants to diagram a system, a request flow, a data model, a state machine, or runtime topology as text that renders in GitHub and Confluence. For prose API specs use api-spec-rest/api-spec-grpc instead.
---

# Architecture Diagram Mode

Produce a clear architecture diagram as **code**, not prose. The job is two
decisions and one artifact: pick the notation that answers the question being
asked, then emit correct, renderable diagram source in a fenced block inside a
short Markdown document. Rendering is delegated to whatever consumes the file —
GitHub, the Confluence Mermaid macro, mermaid.live, or a PlantUML server — so the
deliverable is valid source a reader can paste and render without edits.

Default to **Mermaid**; it renders natively in GitHub and Confluence. Reach for
**PlantUML** only when Mermaid can't express the diagram well (rich UML detail,
detailed C4 component views via C4-PlantUML).

## Method

1. **Identify the question.** A diagram answers one question — "what talks to
   what", "what happens in what order", "what does the data look like", "what are
   the states". Pin that down first; it picks the notation. Ask for the missing
   essentials in one batch — the elements, their relationships, and the boundary
   of what's in scope — rather than guessing the system.
2. **Choose the notation.** Use the table in **Choosing the notation** below to
   map the question to a diagram type, then default to Mermaid for it. Switch to
   PlantUML only for the cases flagged there.
3. **Load the reference.** Read
   `${CLAUDE_PLUGIN_ROOT}/skills/arch-diagram/reference.md` and copy the minimal
   syntax skeleton for the chosen diagram type (Mermaid and PlantUML variants are
   both there). Follow its structure exactly so the source renders first try.
4. **Draw it focused.** Put only the elements that answer the question on the
   diagram. Label every edge with what flows across it. Group related nodes into
   boundaries/subgraphs where it clarifies, not for decoration.
5. **Don't invent.** If a component, protocol, or relationship is unknown, mark
   it `<TODO: …>` rather than inventing one. A small accurate diagram beats a
   speculative busy one.
6. **Deliver.** Write the diagram to the path the user names, or propose a
   sensible default (e.g. `docs/diagrams/<name>.md`) and confirm. Lead the file
   with a one-line title and a sentence of context, then the fenced diagram.
   Report the path.

## Choosing the notation

Map the question to the diagram, then use Mermaid unless the **Fallback** column
says otherwise.

| The question is…                             | Diagram type        | Notation                         | Fallback to PlantUML when…                 |
| -------------------------------------------- | ------------------- | -------------------------------- | ------------------------------------------ |
| What systems/people exist and how they link  | C4 Context          | Mermaid `C4Context`              | you need fine layout control               |
| What containers make up one system           | C4 Container        | Mermaid `C4Container`            | detailed component view (C4-PlantUML)      |
| What happens, in what order, between parties | Sequence            | Mermaid `sequenceDiagram`        | rarely — Mermaid covers this well          |
| What are the types and their relationships   | Class / UML         | Mermaid `classDiagram`           | rich UML (generics, stereotypes, packages) |
| What does the persisted data look like       | Entity-relationship | Mermaid `erDiagram`              | rarely                                     |
| What states does a thing move through        | State machine       | Mermaid `stateDiagram-v2`        | rarely                                     |
| What is the process / decision logic         | Flowchart           | Mermaid `flowchart`              | rarely                                     |
| What runs where at runtime (nodes/zones)     | Deployment          | Mermaid `flowchart` w/ subgraphs | true UML deployment nodes/artifacts        |
| What is the delivery timeline / phasing      | Roadmap             | Mermaid `gantt`                  | n/a                                        |

## Document shape

- **Title + context:** a `#` title and one sentence stating what the diagram
  shows and the boundary of what's in scope.
- **The diagram:** one fenced block tagged `mermaid` or `plantuml`. One diagram
  per file unless the user asks for a set; for a set, give each its own `##`
  heading and context line.
- **Legend (optional):** a short bullet list or note only if the notation isn't
  self-evident (e.g. line styles, colors, or a non-obvious boundary).

## Rules

- **Mermaid-first.** Mermaid renders natively on GitHub and in the Confluence
  Mermaid macro, so it's the default for every diagram type. Only fall back to
  PlantUML for the cases flagged in the table — rich UML or detailed C4 component
  views (C4-PlantUML) — and say once why, so the reader knows it needs a PlantUML
  renderer.
- **One question per diagram.** Each diagram answers a single question. If the
  answer needs two views (e.g. a context *and* a sequence), emit two diagrams,
  each with its own heading — don't cram both into one.
- **Label every edge.** Every arrow/relationship carries a label saying what
  flows or what the relationship is (`places order`, `reads`, `1..*`). An
  unlabeled edge is a question, not an answer.
- **Don't invent.** Mark unknown components, protocols, cardinalities, or
  directions `<TODO: …>` rather than fabricating them. Never guess security or
  trust boundaries.
- **Direction and grouping.** Set an explicit direction (`flowchart LR`,
  `direction TB`) so layout is stable. Use subgraphs/boundaries to show
  ownership or trust zones, and keep nesting shallow.
- **Correct fence language.** Tag the block `mermaid` or `plantuml` — never a
  bare fence. PlantUML source goes between `@startuml` / `@enduml` inside the
  fence.
- **Confluence-portable.** The output may be published to Confluence as well as
  GitHub, so use only portable Markdown around the diagram — headings, lists,
  fenced code. Do **not** use GitHub-only alert callouts (`> [!NOTE]`, …) or raw
  HTML; they render as literal text in Confluence. Note once that Mermaid needs
  the **Mermaid** macro in Confluence and PlantUML needs a PlantUML macro/server.
- **One line per paragraph.** Write each paragraph and list item on a single
  line — do not hard-wrap prose. Confluence renders a mid-paragraph newline as a
  line break.
- **Markdownlint-clean.** Blank lines around headings, lists, tables, and fenced
  blocks; a language on every fence; single trailing newline; no trailing spaces.
- The finished `.md` file is the deliverable — make the source render first try.

## Examples

Bundled reference diagrams built with this skill:

- `examples/c4-context.md` — a C4 Context diagram in Mermaid.
- `examples/sequence.md` — a request flow as a Mermaid sequence diagram.
- `examples/erd.md` — a data model as a Mermaid entity-relationship diagram.
- `examples/class-uml.md` — a class diagram in PlantUML (the fallback case).
