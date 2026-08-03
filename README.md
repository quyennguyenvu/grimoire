# grimoire

A personal Claude Code **plugin marketplace** — a book of reusable "spells"
(subagents, skills, and slash commands) that give [Claude Code](https://claude.com/claude-code)
focused, opinionated personas for code review, research, writing, financial
modeling, presentations, senior-engineer pair programming, and Conventional
Commits.

Everything ships as a single plugin, **`grimoire-core`**, installable straight
from GitHub — or from a local clone if you'd rather hack on it.

> [!IMPORTANT]
> **Invocation names are namespaced after install.** Slash commands are prefixed
> with the plugin name: the `commit` command is invoked as
> **`/grimoire-core:commit`**, not `/commit`. Agents and skills are still
> referred to by their bare name (e.g. "use the `code-reviewer` agent").

## Contents

- [What's inside](#whats-inside) — the catalog of agents, skills, and commands
- [Install](#install) — prerequisites, register, install, verify
- [Update and uninstall](#update-and-uninstall)
- [Optional: skip permission prompts](#optional-skip-permission-prompts) — hooks
  for `commit` and `standup`
- [Global CLAUDE.md rules](#global-claudemd-rules) — the behavioral half of the
  commit guard
- [Repository layout](#repository-layout)
- [Adding your own](#adding-your-own)
- [Appendix: migrating from the old symlink setup](#appendix-migrating-from-the-old-symlink-setup)

## What's inside

The marketplace (`grimoire`) currently publishes one plugin, `grimoire-core`,
containing three kinds of component.

### Agents

_Subagents_ — separate personas Claude delegates a task to. Each runs in its own
context with a restricted tool set, then returns a result. Good for offloading
focused, self-contained work ("review this diff", "size this market").

| Agent                | What it does                                                                                                                                                                            | Invoke as                                                    |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `code-reviewer`      | Read-only code review (Go-fluent) — correctness, error handling, concurrency, security, idioms. Returns prioritized findings, makes no edits.                                           | delegated by description, or "use the `code-reviewer` agent" |
| `researcher`         | Web research, competitor scans, market sizing. Reads many sources, returns a distilled, cited summary.                                                                                  | "have the `researcher` size the market for X"                |
| `writer`             | Drafts and edits clean, persuasive prose — docs, listings, posts, emails, landing copy.                                                                                                 | "use the `writer` to draft …"                                |
| `finance-modeler`    | Cost models, unit economics, break-even, pricing scenarios, P&L. Auditable CSV/markdown with assumptions laid bare.                                                                     | "use the `finance-modeler` for …"                            |
| `presenter`          | Turns source docs and data into slide decks and visual reports with charts.                                                                                                             | "use the `presenter` to build a deck"                        |
| `software-architect` | Designs a system or feature into a small architecture package — C4/sequence/ER diagrams, ADRs, and a linking design brief. Explores the code first, delegates formatting to its skills. | "use the `software-architect` to design X"                   |

### Skills

Response modes — they change how Claude itself answers in the main conversation
rather than spawning a subagent. Claude activates them by intent; you can also
ask for one by name.

| Skill             | What it does                                                                                                                                                                                            | Invoke as                                              |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| `senior-engineer` | Hyper-concise pair-programming mode: direct answer → code → trade-offs. Zero fluff, exact terminology.                                                                                                  | auto by intent, or ask for "senior-engineer mode"      |
| `api-spec-rest`   | Drafts a standardized Markdown REST/HTTP API spec — one endpoint, or several sharing a domain/base URL/auth — with parameter and schema tables, examples, and a shared error model.                     | auto by intent, or ask to "draft a REST API spec"      |
| `api-spec-grpc`   | Drafts a standardized Markdown gRPC API spec — one RPC, or several sharing a proto package/server/auth — with proto messages, streaming type, `grpcurl` examples, and the gRPC status-code error model. | auto by intent, or ask to "draft a gRPC API spec"      |
| `arch-diagram`    | Emits architecture diagrams as code — picks the notation (C4, sequence, class, ER, state, flowchart, deployment, roadmap) and writes renderable Mermaid (default) or PlantUML (fallback).               | auto by intent, or ask to "draw a C4/sequence diagram" |
| `arch-decision`   | Drafts an Architecture Decision Record or lightweight RFC — context, drivers, options with honest trade-offs, decision, and consequences — from a MADR-style template.                                  | auto by intent, or ask to "write an ADR for X"         |

### Commands

Parameterized prompts invoked with a slash, namespaced by the plugin name.

| Command   | What it does                                                                                                                                                                                  | Invoke as                    |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| `commit`  | Stages changes and writes a Conventional Commits message from the diff.                                                                                                                       | **`/grimoire-core:commit`**  |
| `standup` | Reconstructs a copy-pasteable standup log (Done / In progress / Blockers) from git commit history — current repo or a container of repos, per-repo identity, local-midnight day/week windows. | **`/grimoire-core:standup`** |

## Install

### Prerequisites

- **Claude Code ≥ 2.1** (`claude --version`). Plugin marketplaces are supported
  in current releases.

### 1. Register the marketplace

From a terminal:

```sh
claude plugin marketplace add quyennguyenvu/grimoire
```

Or in an interactive Claude Code session:

```text
/plugin marketplace add quyennguyenvu/grimoire
```

This clones the public repo, reads `.claude-plugin/marketplace.json`, and
registers a marketplace named **`grimoire`**. You can also pass the full URL
(`https://github.com/quyennguyenvu/grimoire`) if you prefer.

> [!TIP]
> **Working on the spells yourself?** Clone the repo and register it from the
> local path instead — edits then take effect after a marketplace update (see
> [Update and uninstall](#update-and-uninstall)):
>
> ```sh
> git clone https://github.com/quyennguyenvu/grimoire.git
> claude plugin marketplace add ./grimoire   # or an absolute path
> ```

### 2. Install the plugin

```sh
claude plugin install grimoire-core@grimoire
```

Or in-session:

```text
/plugin install grimoire-core@grimoire
```

The plugin is **copied into Claude Code's plugin cache** at install time — it
does not run from your working tree.

### 3. Verify

- `/plugin` — opens the plugin manager; `grimoire-core` should be listed and
  enabled.
- `/help` — the commands appear as **`/grimoire-core:commit`** and
  **`/grimoire-core:standup`**.
- `/agents` — `code-reviewer`, `researcher`, `writer`, `finance-modeler`,
  `presenter`, and `software-architect` are listed (under the `grimoire-core`
  plugin).
- The skills are available — ask for "senior-engineer mode" and Claude switches
  into the terse response style.

## Update and uninstall

### Update to the latest version

The plugin is copied into the cache at install time, so new changes only take
effect after you refresh the marketplace. Pull the latest from GitHub with:

```sh
claude plugin marketplace update grimoire
```

Or in-session, then reload so non-skill components (commands/agents) re-read:

```text
/plugin marketplace update grimoire
/reload-plugins
```

> [!NOTE]
> If you registered from a local clone, run `git pull` in it first — the
> marketplace update copies from whatever the clone currently contains.

### Uninstall

```sh
claude plugin uninstall grimoire-core@grimoire
```

Or in-session: `/plugin uninstall grimoire-core@grimoire`. To drop the
marketplace entirely: `claude plugin marketplace remove grimoire`.

## Optional: skip permission prompts

`standup` prints its log the moment you invoke it; `commit` drafts a message and,
once you confirm, commits it. In both cases the underlying `git` / `find` /
`date` work would normally raise a Claude Code tool-permission prompt. Two
`PreToolUse` hooks auto-approve exactly those commands so the flow isn't
interrupted — and the commit hook doubles as a guard that **denies** any
`git commit` that doesn't carry `commit`'s marker file, plus every
history-rewriting `git commit --amend`.

These hooks live in your **user-global** `~/.claude/settings.json` — _not_ in the
plugin. A marketplace plugin shouldn't silently alter your permission system, so
they are opt-in and per-machine. Without them the commands still work; Claude
just prompts before each git command (and before the commit itself).

### Add the hooks

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "cmd=$(jq -r '.tool_input.command // \"\"'); if echo \"$cmd\" | grep -qE '\\bgit[[:space:]]+commit($|[^-a-zA-Z0-9_])'; then if echo \"$cmd\" | grep -qE -- '(^|[^-a-zA-Z0-9_])--amend([^a-zA-Z0-9_]|$)'; then echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"Blocked: git commit --amend rewrites history and is never auto-approved. If you truly need to amend, do it manually in your own terminal.\"}}'; elif echo \"$cmd\" | grep -qF 'GRIMOIRE_COMMIT_MSG'; then echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"allow\",\"permissionDecisionReason\":\"Commit via /commit (already confirmed in chat).\"}}'; else echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"Blocked: commits must go through the /commit command (it references GRIMOIRE_COMMIT_MSG). Do not commit any other way; if a commit is genuinely needed, tell the user to run /commit or commit manually in their own terminal.\"}}'; fi; fi"
          },
          {
            "type": "command",
            "command": "cmd=$(jq -r '.tool_input.command // \"\"'); if echo \"$cmd\" | head -n1 | grep -qE '^[[:space:]]*# GRIMOIRE_STANDUP'; then echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"allow\",\"permissionDecisionReason\":\"Read-only standup scan via /standup.\"}}'; fi"
          }
        ]
      }
    ]
  }
}
```

If you already have a `PreToolUse` block matching `Bash`, **merge** these two
entries into its `hooks` array rather than replacing it. After saving, open
`/hooks` once (or restart Claude Code) so the config reloads.

### What each hook decides

- **Commit hook** — auto-approves a `git commit` only when it references the
  `GRIMOIRE_COMMIT_MSG` file that `commit` writes **and** is not a `--amend`.
  Every other `git commit` — a bare `git commit -m`, an auto-commit from another
  tool or workflow, or any `git commit --amend` (history rewrite) — is
  **denied**. (Prefer a prompt over a hard block? Change a branch's
  `permissionDecision` from `deny` to `ask`.)
- **Standup hook** — auto-approves a Bash command only when its **first line** is
  the `# GRIMOIRE_STANDUP` marker that `standup` puts on its read-only scans.

### Security: the markers are forgeable

> [!WARNING]
> **Treat these hooks as a convenience you opt into, not a security boundary.**
> They approve commands by a marker string, and the marker is forgeable —
> crucially, forgeable **by Claude itself**.
>
> - The commit hook trusts the presence of the `GRIMOIRE_COMMIT_MSG` file as
>   proof that you confirmed, but nothing stops an autonomous workflow (a plan
>   executor, a "commit per task" loop, or a reused leftover file) from writing
>   that file and committing without ever asking you. The hook cannot tell a
>   confirmed `/commit` from a forged one.
> - If you rely on reviewing every commit, the real control is the behavioral
>   rule in your `CLAUDE.md` (see
>   [Global CLAUDE.md rules](#global-claudemd-rules)). The hook is only a
>   backstop that blocks stray commits and all `--amend` history rewrites.
> - The standup hook only matches the `# GRIMOIRE_STANDUP` comment the command
>   emits on a read-only scan.
> - Manual commits in your own terminal are unaffected — hooks fire only on
>   Claude's tool calls.

## Global CLAUDE.md rules

The commit hook is only a backstop; the instruction that actually keeps every
commit under your review is a behavioral rule in your **user-global**
`~/.claude/CLAUDE.md`. Keeping a copy here means it travels with grimoire: on a
new machine you paste the hook above **and** these rules, and you're back in
sync.

Create `~/.claude/CLAUDE.md` if it doesn't exist, then merge in the sections you
want — `## Git` is the half that pairs with the commit hook; `## Code comments`
and `## graphify` are personal preferences, safe to drop:

```markdown
# Global instructions

## Git

- Never add a `Co-Authored-By: Claude` trailer (or any Claude/Anthropic co-author attribution) to commit messages.
- **Only ever create a commit through the `/grimoire-core:commit` command, and
  only after I have explicitly confirmed the drafted message in that same
  exchange.** Never run `git commit` (with `-m`, `-F`, or `--amend`) on your own
  initiative, and never write a `GRIMOIRE_COMMIT_MSG.txt` file except as the
  final step of a `/commit` I just confirmed — writing that file _is_ the commit
  authorization, so creating it for any other reason forges my approval. If a
  skill or workflow (plan execution, "commit per task", finishing a branch,
  etc.) reaches a commit step, stop and ask me to run `/commit`; do not
  auto-commit to keep the workflow moving.
- Never use `git commit --amend` — it rewrites history and is blocked by the
  commit hook. If a commit genuinely needs amending, tell me and I'll do it
  manually in my own terminal.

## Code comments

- **Prioritize concise over complete.** Comments explain _why_, not _what_ — the
  code already says what it does. Prefer one short line to a paragraph; prefer no
  comment to a redundant one.
- Don't restate the signature, narrate obvious control flow, or add section
  banners, changelogs, or "added X" notes.
- Match the surrounding file's comment density and style. Doc comments on
  exported/public APIs are fine — keep them to the contract (behavior, params,
  errors), not a tutorial.
- Reserve longer comments for genuinely non-obvious things: tricky invariants,
  workarounds with a reason/link, subtle concurrency or ordering constraints.

## graphify

- **graphify** (`~/.claude/skills/graphify/SKILL.md`) - any input to knowledge graph. Trigger: `/graphify`
  When the user types `/graphify`, use the installed graphify skill or instructions before doing anything else.
```

With both halves in the repo, a new machine is fully in sync after just those
two pastes.

## Repository layout

```text
grimoire/
├── .claude-plugin/
│   └── marketplace.json            # marketplace manifest → lists plugins
├── plugins/
│   └── grimoire-core/
│       ├── .claude-plugin/
│       │   └── plugin.json         # plugin manifest (name, version, author)
│       ├── agents/
│       │   ├── code-reviewer.md
│       │   ├── finance-modeler.md
│       │   ├── presenter.md
│       │   ├── researcher.md
│       │   ├── software-architect.md
│       │   └── writer.md
│       ├── commands/
│       │   ├── commit.md
│       │   └── standup.md
│       └── skills/
│           ├── api-spec-grpc/
│           │   ├── SKILL.md
│           │   ├── template.md
│           │   └── examples/
│           │       ├── single-rpc.md
│           │       └── multiple-rpcs.md
│           ├── api-spec-rest/
│           │   ├── SKILL.md
│           │   ├── template.md
│           │   └── examples/
│           │       ├── single-endpoint.md
│           │       ├── multiple-endpoints.md
│           │       └── public-api.md
│           ├── arch-decision/
│           │   ├── SKILL.md
│           │   ├── template.md
│           │   └── examples/
│           │       ├── adr-accepted.md
│           │       └── rfc-proposal.md
│           ├── arch-diagram/
│           │   ├── SKILL.md
│           │   ├── reference.md
│           │   └── examples/
│           │       ├── c4-context.md
│           │       ├── sequence.md
│           │       ├── erd.md
│           │       └── class-uml.md
│           └── senior-engineer/
│               └── SKILL.md
├── .markdownlint.yaml               # lint policy for every .md in the repo
├── CLAUDE.md                        # authoring guidance for Claude Code
└── README.md
```

## Adding your own

1. Drop the component into the plugin's conventional directory:
   `plugins/grimoire-core/agents/<name>.md`,
   `plugins/grimoire-core/commands/<name>.md`, or
   `plugins/grimoire-core/skills/<name>/SKILL.md`.
2. Write a sharp `description` — this is what Claude matches against, so be
   concrete about _when_ to use it.
3. Keep paths portable: never hardcode `~/.claude` or absolute paths; use
   plugin-relative paths or `${CLAUDE_PLUGIN_ROOT}` (the plugin is copied into a
   cache and can't reach outside its own tree).
4. Refresh: `claude plugin marketplace update grimoire` (+ `/reload-plugins`).

To add a whole new plugin, create `plugins/<plugin>/` with its own
`.claude-plugin/plugin.json` and add an entry to `.claude-plugin/marketplace.json`.
See `CLAUDE.md` for the full authoring conventions.

## Appendix: migrating from the old symlink setup

Earlier versions of grimoire were installed by symlinking (or copying)
`agents/`, `skills/`, and `commands/` into `~/.claude/`. Those copies will
**shadow or duplicate** the plugin's components, so remove them after installing
the plugin:

```sh
# Inspect what points into this repo first
ls -la ~/.claude/agents ~/.claude/skills ~/.claude/commands

# Remove the old symlinks / stale copies (review each before deleting):
rm ~/.claude/skills                       # symlink to grimoire/skills
rm ~/.claude/agents/agents                # stray symlink to grimoire/agents
rm ~/.claude/agents/code-reviewer.md \
   ~/.claude/agents/finance-modeler.md \
   ~/.claude/agents/researcher.md \
   ~/.claude/agents/writer.md             # stale copies
```

After cleanup, the only source of these components is the installed
`grimoire-core` plugin. Confirm with `/agents` and `/help` that each appears
exactly once.
