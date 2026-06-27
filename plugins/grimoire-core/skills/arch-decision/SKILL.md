---
name: arch-decision
description: Draft an Architecture Decision Record (ADR) or lightweight design RFC as a Markdown document — context and problem, decision drivers, considered options with honest trade-offs, the chosen outcome, and consequences — using a MADR-style template. Use when the user wants to capture or propose a technical decision (choose X over Y, adopt a pattern, change an architecture) in a durable, reviewable format. For diagrams use arch-diagram; for prose API specs use api-spec-rest/api-spec-grpc.
---

# Architecture Decision Mode

Produce one decision record: a short, durable Markdown document that captures
*why* a technical choice was made — the context, the options weighed, the
decision, and what it costs. The same template serves two modes that differ only
in **Status**: an **ADR** records a decision already made (`Accepted`); an
**RFC** proposes one for review (`Proposed`). One record = one decision.

A good record is honest about trade-offs. The value is the reasoning a future
reader (often the author) needs to understand why the path was taken and when to
revisit it — not a sales pitch for the winning option.

## Method

1. **Gather the decision.** Pin down: the decision being made (one sentence), the
   context and problem forcing it, the decision drivers (the criteria that
   matter), the options considered, the chosen outcome, and the consequences.
   Ask for the missing essentials in one batch — especially the *rejected*
   options and *why* they lost, since that is the part most often missing.
2. **Pick the mode.** ADR (decision already made → `Accepted`) or RFC (proposing
   for review → `Proposed`). It only changes the Status line and tone, not the
   structure.
3. **Load the template.** Read
   `${CLAUDE_PLUGIN_ROOT}/skills/arch-decision/template.md` and follow its
   structure exactly.
4. **Fill concretely.** Give each considered option real, specific pros and cons
   tied to the decision drivers — no strawmen. State the chosen option plainly
   and justify it against the drivers. List consequences as good / bad / neutral,
   including the follow-up work the decision creates.
5. **Number and file it.** Decisions are sequentially numbered:
   `NNNN-kebab-case-title.md` (e.g. `0007-use-postgres-over-dynamodb.md`) under
   `docs/adr/` (or `docs/decisions/`). Use the next free number.
6. **Deliver.** Write the record to that path, or confirm a proposed default.
   Report the path and the assigned number.

## Document shape

- **Title:** `# NNNN. <decision in a short noun phrase>` — what was decided, not
  the problem (e.g. "Use PostgreSQL for the order store").
- **`**Status:**` line:** one paste-ready status (see the legend in the
  template), plus a date and, when superseding, a link to the prior record.
- **`## Context and problem`:** the forces at play and the problem this decision
  resolves — enough that a newcomer understands why a decision was needed.
- **`## Decision drivers`:** the criteria the options are judged against
  (a short bullet list) — the rubric the decision is made on.
- **`## Considered options`:** each option as a `###` subsection with honest
  **Pro:** / **Con:** bullets tied to the drivers. Include the chosen option too.
- **`## Decision`:** the chosen option stated plainly, and *why* it best satisfies
  the drivers. For an RFC, this is the recommendation under review.
- **`## Consequences`:** what becomes true as a result — **Good**, **Bad**, and
  **Neutral** bullets, including new follow-up work or risks accepted.
- **`## Links` (optional):** related records, tickets, or docs; the superseding
  or superseded record.

## Rules

- **One decision per record.** If the task bundles several decisions, split them
  into separate numbered records that link each other.
- **Honest options, no strawmen.** Every considered option gets a fair hearing
  with concrete pros and cons. A reader should believe a different choice was
  genuinely possible. Don't pad the field with options nobody considered.
- **Tie reasoning to the drivers.** Pros, cons, and the decision all reference
  the decision drivers — that's what makes the record auditable rather than an
  opinion.
- **Status legend.** Define status with one paste-ready line under the title.
  Lifecycle: `Proposed` → `Accepted` → later `Deprecated` or `Superseded by
  NNNN`. The legend lives in the template; copy the matching line.
- **Don't invent.** If a benchmark, cost, constraint, or requirement is unknown,
  mark it `<TODO: …>` rather than fabricating numbers. Never invent measured
  results to justify the decision.
- **Immutable once accepted.** An accepted record isn't edited to change the
  decision — it's superseded by a new record that links back. Note this when the
  user asks to "change" an existing ADR.
- **Sequential numbering.** `NNNN-kebab-case-title.md` with a zero-padded number;
  use the next free number under the decisions directory.
- **Confluence-portable.** The record may be published to Confluence as well as
  GitHub, so use only portable Markdown — headings, tables, fenced code, lists,
  blockquotes, emoji. Do **not** use GitHub-only alert callouts (`> [!NOTE]`, …)
  or raw HTML; they render as literal text in Confluence.
- **One line per paragraph.** Write each paragraph and list item on a single
  line — do not hard-wrap prose. Confluence renders a mid-paragraph newline as a
  line break.
- **Markdownlint-clean.** Blank lines around headings, lists, tables, and fenced
  blocks; a language on every fence; single trailing newline; no trailing spaces.
- The finished `.md` file is the deliverable — make it clear and ready to publish.

## Examples

Bundled reference records built from this template:

- `examples/adr-accepted.md` — a decided ADR (`Accepted`).
- `examples/rfc-proposal.md` — the same shape in RFC mode (`Proposed`, under
  review).
