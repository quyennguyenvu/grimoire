---
name: api-spec-grpc
description: Draft a clean, standardized gRPC API specification as a Markdown document. Use when the user wants to document a gRPC service and its RPCs — one method or several sharing a proto package, server, and authentication — in one consistent template covering the proto package, auth via metadata, request/response messages, streaming type, examples, and the gRPC status-code error model. For REST/HTTP endpoints use api-spec-rest instead.
---

# gRPC API Spec Mode

Produce a clear, consistent Markdown specification for one or more gRPC services.
Shared context (proto package, servers, authentication, conventions, status-code
error model) is written **once** in a preamble; each RPC is its own section
underneath. The result is a single self-contained `.md` file a reader can
implement a client or server against without guessing.

## Method

1. **Gather the inputs.** For the document as a whole: title, proto
   package/file/`go_package`, server addresses + TLS, authentication scheme
   (usually a metadata token or mTLS), and cross-cutting conventions (pagination,
   timestamps, deadlines, error model). For each RPC: name, signature, streaming
   type, required auth/scope, request and response messages, method-specific
   status codes, and a concrete example. Ask for the missing essentials in one
   batch — don't interrogate field by field.
2. **Load the template.** Read `${CLAUDE_PLUGIN_ROOT}/skills/api-spec-grpc/template.md`
   and follow its structure exactly. It defines the preamble plus one repeatable
   RPC section.
3. **Assemble.** For a multi-RPC spec, lead with a two-level `## Contents` right
   after the intro (title, owner, overview); omit it for a single-RPC spec. Then
   `## How to read this doc`, `## Basics` (the shared context as `###`
   subsections, including the `### Package` `service` block listing all RPCs),
   and `## RPCs` with one `###` section per RPC (its detail blocks are `####`).
4. **Fill concretely.** Use real `.proto` message definitions and realistic
   values in every `grpcurl` example so the format is unambiguous. Mirror field
   names and numbers between the proto block and the field table.
5. **Mark the streaming type.** State whether each RPC is unary, server
   streaming, client streaming, or bidirectional — it changes the client code.
6. **Drop what doesn't apply.** Omit any subsection an RPC doesn't use. A short,
   accurate spec beats a padded one with empty tables.
7. **Deliver.** Write the spec to the path the user names, or propose a sensible
   default (e.g. `docs/api/<service>.md`) and confirm. Report the path.

## Document shape

- **`## Contents`** (multi-RPC specs only): at the top, right after the intro —
  a two-level list. Top level is the `##` sections, second level is the `###`
  items under Basics and RPCs.
- **`## How to read this doc`:** one-paragraph orientation + the status legend
  (a table of paste-ready `**Status:**` lines).
- **`## Basics`** (shared, written once): `### Package` (info + `service` block),
  `### Servers`, `### Authentication` (via metadata), `### Conventions`,
  optionally `### Enums` (proto enums reused across RPCs), then `### Errors`
  (canonical status-code table + `google.rpc.Status` detail).
- **`## RPCs`:** one `### <name>` per RPC, each with its own `**Status:**` line,
  `rpc` signature, streaming type, auth note, and `####` detail blocks — request
  message (proto + field table), response message (proto + field table), a
  `#### Status codes` table, a `grpcurl` example, notes. Every field table ends
  with an **Example** column; field tables use dotted paths for nested fields.

## Rules

- **Don't invent.** If an auth scheme, status code, field, or proto type is
  unknown, mark it `<TODO: …>` rather than fabricating it. Never guess security
  behavior.
- **Use canonical status codes.** Map failures to the standard gRPC codes
  (`INVALID_ARGUMENT`, `NOT_FOUND`, `PERMISSION_DENIED`, `UNAUTHENTICATED`, …) —
  there is no per-RPC HTTP status here.
- **Concise and scannable.** Prefer tables over prose for message fields. One
  idea per row. Keep descriptions to a single line where possible.
- **Example column.** End every field table with an **Example** column holding a
  realistic sample value for each row.
- **Dotted-path fields.** Name nested message fields by path — `customer.id`,
  `data[].id` — so one flat table conveys structure. Reference a shared enum in
  the Type column as `Name (enum)`.
- **Per-RPC status codes.** Give each RPC a `#### Status codes` table
  (`Code | When`) listing the codes it returns and the exact trigger — more
  specific than the shared Errors table in Basics.
- **Shared enums (optional).** If proto enums recur across RPCs, list them once
  in `### Enums`; omit the section when there are none. (gRPC has no response
  envelope, so there's no "Response format" section — the `google.rpc.Status`
  error model in Errors covers the shared wrapper.)
- **Group shared context once.** Anything common to all RPCs (package, servers,
  auth, conventions, error model) lives under `## Basics`, never repeated per
  section. Keep the top-level headings to three — How to read this doc, Basics,
  RPCs.
- **Public services.** If the service needs no authentication, say so once: set
  `### Authentication` to "None — this service is public", omit the per-RPC
  **Auth** notes, and drop `PERMISSION_DENIED`/`UNAUTHENTICATED` from the shared
  status-code table.
- **Per-RPC status.** Status is set per RPC, not per document. Define the legend
  once in a **How to read this doc** section (a table of paste-ready
  `**Status:** 🟡 Draft` / `🟢 Stable` / `🔴 Deprecated` lines and their
  meanings), then put a single `**Status:**` line under each RPC heading.
- **Confluence-portable.** The output is published to Confluence as well as
  GitHub, so use only portable Markdown — headings, tables, fenced code, lists,
  blockquotes, and emoji. Do **not** use GitHub-only alert callouts
  (`> [!NOTE]`, `> [!WARNING]`, …) or raw HTML; they render as literal text in
  Confluence. Status color comes from the emoji. For a native colored status
  lozenge or an auto-built TOC in Confluence, note that the author can swap in
  the **Status** and **Table of Contents** macros.
- **Plain URLs.** Write absolute `http(s)://` URLs as plain text, not inside code
  spans — Confluence auto-links a URL in a code span and renders a literal `< >`
  around it. (gRPC `host:port` addresses have no scheme, so they stay as code.)
- **One line per paragraph.** Write each paragraph and list item on a single
  line — do not hard-wrap prose. Confluence renders a mid-paragraph newline as a
  line break, so wrapped source produces chopped-up, unnatural paragraphs.
- **Two-level Contents.** When present, Contents sits at the top — right after
  the intro, before How to read this doc — as a two-level list: top level = the
  `##` sections (How to read this doc, Basics, RPCs), second level = the `###`
  items under Basics and RPCs. Don't list the document title, the Contents
  section itself, or `####` RPC internals. Every link must point to a real
  heading slug (lowercase, spaces → hyphens, punctuation dropped), so name
  sections plainly (`### CreateOrder`).
- **Markdownlint-clean.** Blank lines around headings, lists, tables, and fenced
  blocks; a language on every fence (`protobuf`, `json`, `bash`, `text`); single
  trailing newline; no trailing spaces.
- The finished `.md` file is the deliverable — make it correct and ready to
  publish.

## Examples

Bundled reference specs built from this template:

- `examples/single-rpc.md` — one RPC, no Contents; also a public, no-auth
  service (Authentication is "None").
- `examples/multiple-rpcs.md` — several RPCs sharing a proto package and
  authentication, with a two-level Contents.
