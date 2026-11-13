# grimoire

A personal Claude Code **plugin marketplace** — a book of reusable "spells" (subagents, skills, and slash commands) that give [Claude Code](https://claude.com/claude-code) focused, opinionated personas for code review, presentations, senior-engineer pair programming, API and architecture docs, and Conventional Commits.

Everything ships as a single plugin, **`grimoire`**, installable straight from GitHub — or from a local clone if you'd rather hack on it.

> [!IMPORTANT]
> **Invocation names are namespaced after install.** Slash commands are prefixed with the plugin name: the `seal` command is invoked as **`/grimoire:seal`**, not `/seal`. Agents and skills are listed with the same prefix (`grimoire:warden`), but asking for one by its bare name still works (e.g. "use the `warden` agent").

## Contents

- [What's inside](#whats-inside) — the catalog of agents, skills, and commands
- [Install](#install) — prerequisites, register, install, verify
- [Update and uninstall](#update-and-uninstall)
- [Optional: skip permission prompts](#optional-skip-permission-prompts) — hooks for `seal` and `recall`
- [Global CLAUDE.md rules](#global-claudemd-rules) — the behavioral half of the commit guard
- [Optional: keep ponytail out of the non-coding agents](#optional-keep-ponytail-out-of-the-non-coding-agents)
- [Repository layout](#repository-layout)
- [Adding your own](#adding-your-own)
- [Appendix: migrating from the old symlink setup](#appendix-migrating-from-the-old-symlink-setup)

## What's inside

The marketplace publishes one plugin, also named `grimoire`, containing three kinds of component. Spells carry arcane names; the tables below say what each one actually does.

### Agents

_Subagents_ — separate personas Claude delegates a task to. Each runs in its own context with a restricted tool set, then returns a result. Good for offloading focused, self-contained work ("review this diff", "build a deck from this analysis").

| Agent         | What it does                                                                                                                                                                                                      | Invoke as                                             |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `warden`      | **Code reviewer.** Read-only review with Go depth — correctness, error handling, concurrency, security, idioms. No shell, so pass it the diff or the changed files. Returns prioritized findings, makes no edits. | delegated by description, or "use the `warden` agent" |
| `illusionist` | **Presenter.** Turns source docs and data into slide decks and visual reports with charts.                                                                                                                        | "use the `illusionist` to build a deck"               |

### Skills

Response modes — they change how Claude itself answers in the main conversation rather than spawning a subagent. Claude activates them by intent; you can also ask for one by name.

| Skill        | What it does                                                                                                                                                                                                               | Invoke as                                              |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| `sage`       | **Senior-engineer mode.** Hyper-concise pair programming: direct answer → code → trade-offs. Zero fluff, exact terminology.                                                                                                | auto by intent, or ask for "sage mode"                 |
| `go-runes`   | **Go conventions.** Reference for error handling, concurrency, generics, testing, tooling, modules, package layout, API design, performance. Preloaded into the `warden` agent.                                            | auto when working on Go, or ask to "apply go-runes"    |
| `codex-rest` | **REST API spec.** Drafts a standardized Markdown REST/HTTP API spec — one endpoint, or several sharing a domain/base URL/auth — with parameter and schema tables, examples, and a shared error model.                     | auto by intent, or ask to "draft a REST API spec"      |
| `codex-grpc` | **gRPC API spec.** Drafts a standardized Markdown gRPC API spec — one RPC, or several sharing a proto package/server/auth — with proto messages, streaming type, `grpcurl` examples, and the gRPC status-code error model. | auto by intent, or ask to "draft a gRPC API spec"      |
| `glyph`      | **Architecture diagrams.** Emits diagrams as code — picks the notation (C4, sequence, class, ER, state, flowchart, deployment, roadmap) and writes renderable Mermaid (default) or PlantUML (fallback).                    | auto by intent, or ask to "draw a C4/sequence diagram" |
| `edict`      | **Decision records.** Drafts an Architecture Decision Record or lightweight RFC — context, drivers, options with honest trade-offs, decision, and consequences — from a MADR-style template.                               | auto by intent, or ask to "write an ADR for X"         |

### Commands

Parameterized prompts invoked with a slash, namespaced by the plugin name.

| Command  | What it does                                                                                                                                                                                                   | Invoke as              |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------- |
| `seal`   | **Commit.** Stages changes and writes a Conventional Commits message from the diff.                                                                                                                            | **`/grimoire:seal`**   |
| `recall` | **Standup log.** Reconstructs a copy-pasteable standup log (Done / In progress / Blockers) from git commit history — current repo or a container of repos, per-repo identity, local-midnight day/week windows. | **`/grimoire:recall`** |

## Install

### Prerequisites

- **Claude Code ≥ 2.1** (`claude --version`). Plugin marketplaces are supported in current releases.

### 1. Register the marketplace

From a terminal:

```sh
claude plugin marketplace add quyennguyenvu/grimoire
```

Or in an interactive Claude Code session:

```text
/plugin marketplace add quyennguyenvu/grimoire
```

This clones the public repo, reads `.claude-plugin/marketplace.json`, and registers a marketplace named **`grimoire`**. You can also pass the full URL (`https://github.com/quyennguyenvu/grimoire`) if you prefer.

> [!TIP]
> **Working on the spells yourself?** Clone the repo and register it from the local path instead — the plugin then loads in place from your clone, so edits take effect at the next session start or `/reload-plugins`:
>
> ```sh
> git clone https://github.com/quyennguyenvu/grimoire.git
> claude plugin marketplace add ./grimoire   # or an absolute path
> ```

### 2. Install the plugin

The ID is `<plugin>@<marketplace>`, and both are named `grimoire`.

```sh
claude plugin install grimoire@grimoire
```

Or in-session:

```text
/plugin install grimoire@grimoire
```

Installed from GitHub, the plugin is **copied into Claude Code's plugin cache** — it does not run from a working tree. Registered from a local clone, it loads in place from that clone.

### 3. Verify

- `/plugin` — opens the plugin manager; `grimoire` should be listed and enabled.
- `/help` — the commands appear as **`/grimoire:seal`** and **`/grimoire:recall`**.
- `/agents` — `warden` and `illusionist` are listed (under the `grimoire` plugin).
- The skills are available — ask for "sage mode" and Claude switches into the terse response style.

## Update and uninstall

### Update to the latest version

A GitHub install is a cached copy, so new changes only take effect after you refresh the marketplace. Pull the latest from GitHub with:

```sh
claude plugin marketplace update grimoire
```

Or in-session, then reload so non-skill components (commands/agents) re-read:

```text
/plugin marketplace update grimoire
/reload-plugins
```

> [!NOTE]
> If you registered from a local clone, there is nothing to refresh: run `git pull` in the clone, then start a new session or run `/reload-plugins`.

### Upgrading from `grimoire-core`

The plugin used to be called `grimoire-core`, and every component had a plain name. A renamed plugin does not update in place, so swap it once:

```sh
claude plugin marketplace update grimoire
claude plugin uninstall grimoire-core@grimoire
claude plugin install grimoire@grimoire
```

Then replace `/grimoire-core:commit` with `/grimoire:seal` in your `~/.claude/CLAUDE.md` commit rule and in the commit hook's messages (both are reproduced below).

| Was                      | Now                |
| ------------------------ | ------------------ |
| `/grimoire-core:commit`  | `/grimoire:seal`   |
| `/grimoire-core:standup` | `/grimoire:recall` |
| `code-reviewer` agent    | `warden`           |
| `presenter` agent        | `illusionist`      |
| `senior-engineer` skill  | `sage`             |
| `go-conventions` skill   | `go-runes`         |
| `api-spec-rest` skill    | `codex-rest`       |
| `api-spec-grpc` skill    | `codex-grpc`       |
| `arch-diagram` skill     | `glyph`            |
| `arch-decision` skill    | `edict`            |

### Uninstall

```sh
claude plugin uninstall grimoire@grimoire
```

Or in-session: `/plugin uninstall grimoire@grimoire`. To drop the marketplace entirely: `claude plugin marketplace remove grimoire`.

## Optional: skip permission prompts

`recall` prints its log the moment you invoke it; `seal` drafts a message and, once you confirm, commits it. In both cases the underlying `git` / `find` / `date` work would normally raise a Claude Code tool-permission prompt. Two `PreToolUse` hooks auto-approve exactly those commands so the flow isn't interrupted — and the commit hook doubles as a guard that **denies** any `git commit` that doesn't carry `seal`'s marker file, plus every history-rewriting `git commit --amend`.

These hooks live in your **user-global** `~/.claude/settings.json` — _not_ in the plugin. A marketplace plugin shouldn't silently alter your permission system, so they are opt-in and per-machine. Without them the commands still work; Claude just prompts before each git command (and before the commit itself).

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
            "command": "cmd=$(jq -r '.tool_input.command // \"\"'); if echo \"$cmd\" | grep -qE '\\bgit[[:space:]]+commit($|[^-a-zA-Z0-9_])'; then if echo \"$cmd\" | grep -qE -- '(^|[^-a-zA-Z0-9_])--amend([^a-zA-Z0-9_]|$)'; then echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"Blocked: git commit --amend rewrites history and is never auto-approved. If you truly need to amend, do it manually in your own terminal.\"}}'; elif echo \"$cmd\" | grep -qF 'GRIMOIRE_COMMIT_MSG'; then echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"allow\",\"permissionDecisionReason\":\"Commit via /grimoire:seal (already confirmed in chat).\"}}'; else echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"Blocked: commits must go through the /grimoire:seal command (it references GRIMOIRE_COMMIT_MSG). Do not commit any other way; if a commit is genuinely needed, tell the user to run /grimoire:seal or commit manually in their own terminal.\"}}'; fi; fi"
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

If you already have a `PreToolUse` block matching `Bash`, **merge** these two entries into its `hooks` array rather than replacing it. After saving, open `/hooks` once (or restart Claude Code) so the config reloads.

### What each hook decides

- **Commit hook** — auto-approves a `git commit` only when it references the `GRIMOIRE_COMMIT_MSG` file that `seal` writes **and** is not a `--amend`. Every other `git commit` — a bare `git commit -m`, an auto-commit from another tool or workflow, or any `git commit --amend` (history rewrite) — is **denied**. (Prefer a prompt over a hard block? Change a branch's `permissionDecision` from `deny` to `ask`.)
- **Standup hook** — auto-approves a Bash command only when its **first line** is the `# GRIMOIRE_STANDUP` marker that `recall` puts on its read-only scans.

### Security: the markers are forgeable

> [!WARNING]
> **Treat these hooks as a convenience you opt into, not a security boundary.** They approve commands by a marker string, and the marker is forgeable — crucially, forgeable **by Claude itself**.
>
> - The commit hook trusts the presence of the `GRIMOIRE_COMMIT_MSG` file as proof that you confirmed, but nothing stops an autonomous workflow (a plan executor, a "commit per task" loop, or a reused leftover file) from writing that file and committing without ever asking you. The hook cannot tell a confirmed `/grimoire:seal` from a forged one.
> - If you rely on reviewing every commit, the real control is the behavioral rule in your `CLAUDE.md` (see [Global CLAUDE.md rules](#global-claudemd-rules)). The hook is only a backstop that blocks stray commits and all `--amend` history rewrites.
> - The standup hook only matches the `# GRIMOIRE_STANDUP` comment the command emits on a read-only scan.
> - Manual commits in your own terminal are unaffected — hooks fire only on Claude's tool calls.

## Global CLAUDE.md rules

The commit hook is only a backstop; the instruction that actually keeps every commit under your review is a behavioral rule in your **user-global** `~/.claude/CLAUDE.md`. Keeping a copy here means it travels with grimoire: on a new machine you paste the hook above **and** these rules, and you're back in sync.

Create `~/.claude/CLAUDE.md` if it doesn't exist, then merge in the sections you want — `## Git` is the half that pairs with the commit hook; `## Code comments` and `## graphify` are personal preferences, safe to drop:

```markdown
# Global instructions

## Git

- Never add a `Co-Authored-By: Claude` trailer (or any Claude/Anthropic
  co-author attribution) to commit messages.
- **Only ever create a commit through the `/grimoire:seal` command, and
  only after I have explicitly confirmed the drafted message in that same
  exchange.** Never run `git commit` (with `-m`, `-F`, or `--amend`) on your own
  initiative, and never write a `GRIMOIRE_COMMIT_MSG.txt` file except as the
  final step of a `/grimoire:seal` I just confirmed — writing that file _is_ the commit
  authorization, so creating it for any other reason forges my approval. If a
  skill or workflow (plan execution, "commit per task", finishing a branch,
  etc.) reaches a commit step, stop and ask me to run `/grimoire:seal`; do not
  auto-commit to keep the workflow moving.
- Never use `git commit --amend` — it rewrites history and is blocked by my
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

## Markdown

- **Any Markdown I ask you to write or edit (`.md`) must pass markdownlint with
  no warnings.** This applies everywhere — docs, READMEs, plans, ADRs, PR
  bodies, scratch notes — not just in repos that happen to have a linter wired
  up. Where a project ships its own config (`.markdownlint*`,
  `.markdownlint-cli2*`), that config wins over these defaults.
- Common rules to get right on the first pass: surround headings, lists, tables,
  and fenced code blocks with blank lines (MD022/MD031/MD032); no blank line
  between adjacent blockquotes — continue one with a `>`-prefixed line (MD028);
  always fence code blocks, never indent them (MD046), and specify a language on
  every fence (MD040); one top-level `#` heading and no skipped levels
  (MD025/MD001); consistent list markers and indentation (MD004/MD005/MD007);
  single trailing newline and no trailing spaces (MD047/MD009).
- **Never hard-wrap prose in any markdown file, anywhere — one line per paragraph or bullet, however long.** MD013/line-length is on by default in the markdownlint CLI (though off in the VS Code extension), so in repos without a markdownlint config, add `.markdownlint.json` with `{"MD013": false}` at the root; for standalone files outside a repo, disregard MD013 findings. If a project's own config deliberately enforces MD013, flag the conflict to me instead of wrapping.
- Verify before saying you're done: run the repo's lint target if it has one
  (e.g. `make lint`), otherwise `markdownlint <file>` or
  `npx markdownlint-cli2 <file>`. If no linter is installed, re-read the file
  against the rules above instead of assuming it's clean.

## Machine resources

- My MacBook has 16 GB RAM and 8 cores. Heavy toolchain commands (full test suites, `golangci-lint`, `go build`/`go vet` of fresh module trees, bundlers, Docker builds) each take 1 GB or more, and running several concurrently has swapped the machine to a freeze. Run them one at a time from the main thread: never inside parallel subagents, and never start one while another is still running in the background.
- For review or analysis fan-out, use read-only agents (Read/Grep/Glob only). At most one agent at a time may run builds, tests, or linters.
- Prefer the project's fast test target for feedback; run full matrices alone with explicit `-parallel` and `-timeout` limits.
- Never delete my caches (`go clean -cache`, the Go module cache, npm/pnpm stores) to get a cold measurement; ask first.

## graphify

- **graphify** (`~/.claude/skills/graphify/SKILL.md`) - any input to knowledge
  graph. Trigger: `/graphify`
  When the user types `/graphify`, use the installed graphify skill or
  instructions before doing anything else.
```

With both halves in the repo, a new machine is fully in sync after just those two pastes.

## Optional: keep ponytail out of the non-coding agents

If you also run the [ponytail](https://github.com/DietrichGebert/ponytail) plugin, its `SubagentStart` hook injects its "lazy senior developer" ruleset into every subagent by default — including `illusionist`, whose final message is meant to be the deliverable itself. Scope it with ponytail's `PONYTAIL_SUBAGENT_MATCHER` (a case-insensitive regex tested against the agent type) in your **user-global** `~/.claude/settings.json`:

```json
{
  "env": {
    "PONYTAIL_SUBAGENT_MATCHER": "^(?!.*illusionist)"
  }
}
```

Every other agent — `warden` and the built-in coding agents — still receives the ruleset. The setting takes effect from the next session.

## Repository layout

```text
grimoire/
├── .claude-plugin/
│   └── marketplace.json            # marketplace manifest → lists plugins
├── plugins/
│   └── grimoire/
│       ├── .claude-plugin/
│       │   └── plugin.json         # plugin manifest (name, version, author)
│       ├── agents/
│       │   ├── illusionist.md      # presenter: decks and visual reports
│       │   └── warden.md           # code reviewer
│       ├── commands/
│       │   ├── recall.md           # standup log from git history
│       │   └── seal.md             # Conventional Commits
│       └── skills/
│           ├── codex-grpc/         # gRPC API spec
│           │   ├── SKILL.md
│           │   ├── template.md
│           │   └── examples/
│           │       ├── single-rpc.md
│           │       └── multiple-rpcs.md
│           ├── codex-rest/         # REST API spec
│           │   ├── SKILL.md
│           │   ├── template.md
│           │   └── examples/
│           │       ├── single-endpoint.md
│           │       ├── multiple-endpoints.md
│           │       └── public-api.md
│           ├── edict/              # ADRs and RFCs
│           │   ├── SKILL.md
│           │   ├── template.md
│           │   └── examples/
│           │       ├── adr-accepted.md
│           │       └── rfc-proposal.md
│           ├── glyph/              # architecture diagrams
│           │   ├── SKILL.md
│           │   ├── reference.md
│           │   └── examples/
│           │       ├── c4-context.md
│           │       ├── sequence.md
│           │       ├── erd.md
│           │       └── class-uml.md
│           ├── go-runes/           # Go conventions
│           │   └── SKILL.md
│           └── sage/               # senior-engineer mode
│               └── SKILL.md
├── .markdownlint.yaml               # lint policy for every .md in the repo
├── CLAUDE.md                        # authoring guidance for Claude Code
└── README.md
```

## Adding your own

1. Drop the component into the plugin's conventional directory: `plugins/grimoire/agents/<name>.md`, `plugins/grimoire/commands/<name>.md`, or `plugins/grimoire/skills/<name>/SKILL.md`.
2. Write a sharp `description` — this is what Claude matches against, so be concrete about _when_ to use it.
3. Keep paths portable: never hardcode `~/.claude` or absolute paths; use plugin-relative paths or `${CLAUDE_PLUGIN_ROOT}` (the plugin is copied into a cache and can't reach outside its own tree).
4. Refresh: `/reload-plugins` for a local clone; `claude plugin marketplace update grimoire` (+ `/reload-plugins`) for a GitHub install.

To add a whole new plugin, create `plugins/<plugin>/` with its own `.claude-plugin/plugin.json` and add an entry to `.claude-plugin/marketplace.json`. See `CLAUDE.md` for the full authoring conventions.

## Appendix: migrating from the old symlink setup

Earlier versions of grimoire were installed by symlinking (or copying) `agents/`, `skills/`, and `commands/` into `~/.claude/`. Those copies will **shadow or duplicate** the plugin's components, so remove them after installing the plugin:

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

After cleanup, the only source of these components is the installed `grimoire` plugin. Confirm with `/agents` and `/help` that each appears exactly once.
