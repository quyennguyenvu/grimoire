---
name: api-spec-rest
description: Draft a clean, standardized REST/HTTP API specification as a Markdown document. Use when the user wants to document an HTTP/REST endpoint — or several endpoints that share a domain, base URL, and authentication — in one consistent template covering auth, parameters, request/response schemas, examples, and a shared error model. For gRPC services use api-spec-grpc instead.
---

# REST API Spec Mode

Produce a clear, consistent Markdown specification for one or more HTTP APIs.
Shared context (base URL, authentication, conventions, error model) is written
**once** in a preamble; each endpoint is its own section underneath. The result
is a single self-contained `.md` file a reader can implement against without
guessing.

## Method

1. **Gather the inputs.** For the document as a whole: title, environments/base
   URLs, authentication scheme, and any cross-cutting conventions (pagination,
   timestamps, error format, rate limits). For each endpoint: method + path,
   purpose, required auth/scope, path & query parameters, request body, response
   status + body, and a concrete example. Ask for the missing essentials in one
   batch — don't interrogate field by field.
2. **Load the template.** Read `${CLAUDE_PLUGIN_ROOT}/skills/api-spec-rest/template.md`
   and follow its structure exactly. It defines the preamble plus one repeatable
   endpoint section.
3. **Assemble.** For a multi-endpoint spec, lead with a two-level `## Contents`
   right after the intro (title, owner, overview); omit it for a single-endpoint
   spec. Then `## How to read this doc`, `## Basics` (the shared context as
   `###` subsections), and `## Endpoints` with one `###` section per endpoint
   (its detail blocks are `####`).
4. **Fill concretely.** Use realistic values in every JSON and `curl` example so
   the format is unambiguous. Mirror field names between the schema table and
   the example body.
5. **Drop what doesn't apply.** Omit any subsection an endpoint doesn't use (no
   query params → no query-params table). A short, accurate spec beats a padded
   one with empty tables.
6. **Deliver.** Write the spec to the path the user names, or propose a sensible
   default (e.g. `docs/api/<resource>.md`) and confirm. Report the path.

## Document shape

- **`## Contents`** (multi-endpoint specs only): at the top, right after the
  intro — a two-level list. Top level is the `##` sections, second level is the
  `###` items under Basics and Endpoints.
- **`## How to read this doc`:** one-paragraph orientation + the status legend
  (a table of paste-ready `**Status:**` lines).
- **`## Basics`** (shared, written once): `### Base URLs`, `### Authentication`,
  `### Conventions`, optionally `### Response format` (the shared envelope) and
  `### Enums` (values reused across endpoints), then `### Errors` (envelope +
  status/code table).
- **`## Endpoints`:** one `### <name>` per endpoint, each with its own
  `**Status:**` line, `METHOD /path`, optional description, auth note, and
  `####` detail blocks — path/query parameters, request body, response, a `curl`
  example, and a `#### Errors` table. Every parameter/field table ends with an
  **Example** column; response field tables use dotted paths for nested fields.

## Rules

- **Don't invent.** If an auth scheme, status code, rate limit, or field is
  unknown, mark it `<TODO: …>` rather than fabricating it. Never guess security
  behavior.
- **Concise and scannable.** Prefer tables over prose for parameters and fields.
  One idea per row. Keep descriptions to a single line where possible.
- **Example column.** End every parameter and field table with an **Example**
  column holding a realistic sample value for each row.
- **Dotted-path fields.** Name nested response fields by path — `customer.id`,
  `data[].id`, `pagination.total` — so one flat table conveys structure.
  Reference a shared enum in the Type column as `enum (name)`.
- **Per-endpoint errors.** Give each endpoint a `#### Errors` table
  (`Status | Code | When`) listing the codes it returns and the exact trigger —
  more specific than the shared Errors table in Basics.
- **Optional shared sections.** If every response is wrapped, document the
  envelope once in `### Response format` and have endpoint responses describe
  only the varying payload. If values recur across endpoints, list them once in
  `### Enums`. Omit either section when it doesn't apply.
- **Group shared context once.** Anything common to all endpoints (base URLs,
  auth, conventions, error model) lives under `## Basics`, never repeated per
  section. Keep the top-level headings to three — How to read this doc, Basics,
  Endpoints.
- **Public APIs.** If the API needs no authentication, say so once: set
  `### Authentication` to "None — this API is public", omit the per-endpoint
  **Auth** notes, and drop `401`/`403` from the shared error table.
- **Per-endpoint status.** Status is set per endpoint, not per document. Define
  the legend once in a **How to read this doc** section (a table of paste-ready
  `**Status:** 🟡 Draft` / `🟢 Stable` / `🔴 Deprecated` lines and their
  meanings), then put a single `**Status:**` line under each endpoint heading.
- **Confluence-portable.** The output is published to Confluence as well as
  GitHub, so use only portable Markdown — headings, tables, fenced code, lists,
  blockquotes, and emoji. Do **not** use GitHub-only alert callouts
  (`> [!NOTE]`, `> [!WARNING]`, …) or raw HTML; they render as literal text in
  Confluence. Status color comes from the emoji. For a native colored status
  lozenge or an auto-built TOC in Confluence, note that the author can swap in
  the **Status** and **Table of Contents** macros.
- **Plain URLs.** Write absolute `http(s)://` URLs (e.g. base URLs) as plain
  text, not inside code spans — Confluence auto-links a URL in a code span and
  renders a literal `< >` around it.
- **One line per paragraph.** Write each paragraph and list item on a single
  line — do not hard-wrap prose. Confluence renders a mid-paragraph newline as a
  line break, so wrapped source produces chopped-up, unnatural paragraphs.
- **Two-level Contents.** When present, Contents sits at the top — right after
  the intro, before How to read this doc — as a two-level list: top level = the
  `##` sections (How to read this doc, Basics, Endpoints), second level = the
  `###` items under Basics and Endpoints. Don't list the document title, the
  Contents section itself, or `####` endpoint internals. Every link must point to
  a real heading slug (lowercase, spaces → hyphens, punctuation dropped), so name
  sections plainly (`### Create order`).
- **Markdownlint-clean.** Blank lines around headings, lists, tables, and fenced
  blocks; a language on every fence (`http`, `json`, `bash`); single trailing
  newline; no trailing spaces.
- The finished `.md` file is the deliverable — make it correct and ready to
  publish.

## Examples

Bundled reference specs built from this template:

- `examples/single-endpoint.md` — one endpoint, no Contents section.
- `examples/multiple-endpoints.md` — several endpoints sharing a domain and
  authentication, with a two-level Contents.
- `examples/public-api.md` — a public, no-auth API (Authentication is "None").
