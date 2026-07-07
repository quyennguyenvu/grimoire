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

## What's inside

The marketplace (`grimoire`) currently publishes one plugin, `grimoire-core`:

| Component            | Type    | What it does                                                                                                                                                                                            | Invoke as                                                    |
| -------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `code-reviewer`      | Agent   | Read-only code review (Go-fluent) — correctness, error handling, concurrency, security, idioms. Returns prioritized findings, makes no edits.                                                           | delegated by description, or "use the `code-reviewer` agent" |
| `researcher`         | Agent   | Web research, competitor scans, market sizing. Reads many sources, returns a distilled, cited summary.                                                                                                  | "have the `researcher` size the market for X"                |
| `writer`             | Agent   | Drafts and edits clean, persuasive prose — docs, listings, posts, emails, landing copy.                                                                                                                 | "use the `writer` to draft …"                                |
| `finance-modeler`    | Agent   | Cost models, unit economics, break-even, pricing scenarios, P&L. Auditable CSV/markdown with assumptions laid bare.                                                                                     | "use the `finance-modeler` for …"                            |
| `presenter`          | Agent   | Turns source docs and data into slide decks and visual reports with charts.                                                                                                                             | "use the `presenter` to build a deck"                        |
| `software-architect` | Agent   | Designs a system or feature into a small architecture package — C4/sequence/ER diagrams, ADRs, and a linking design brief. Explores the code first, delegates formatting to its skills.                 | "use the `software-architect` to design X"                   |
| `senior-engineer`    | Skill   | Hyper-concise pair-programming mode: direct answer → code → trade-offs. Zero fluff, exact terminology.                                                                                                  | auto by intent, or ask for "senior-engineer mode"            |
| `api-spec-rest`      | Skill   | Drafts a standardized Markdown REST/HTTP API spec — one endpoint, or several sharing a domain/base URL/auth — with parameter and schema tables, examples, and a shared error model.                     | auto by intent, or ask to "draft a REST API spec"            |
| `api-spec-grpc`      | Skill   | Drafts a standardized Markdown gRPC API spec — one RPC, or several sharing a proto package/server/auth — with proto messages, streaming type, `grpcurl` examples, and the gRPC status-code error model. | auto by intent, or ask to "draft a gRPC API spec"            |
| `arch-diagram`       | Skill   | Emits architecture diagrams as code — picks the notation (C4, sequence, class, ER, state, flowchart, deployment, roadmap) and writes renderable Mermaid (default) or PlantUML (fallback).               | auto by intent, or ask to "draw a C4/sequence diagram"       |
| `arch-decision`      | Skill   | Drafts an Architecture Decision Record or lightweight RFC — context, drivers, options with honest trade-offs, decision, and consequences — from a MADR-style template.                                  | auto by intent, or ask to "write an ADR for X"               |
| `commit`             | Command | Stages changes and writes a Conventional Commits message from the diff.                                                                                                                                 | **`/grimoire-core:commit`**                                  |
| `standup`            | Command | Reconstructs a copy-pasteable standup log (Done / In progress / Blockers) from git commit history — current repo or a container of repos, per-repo identity, local-midnight day/week windows.           | **`/grimoire-core:standup`**                                 |

## Setup

### 1. Prerequisites

- **Claude Code ≥ 2.1** (`claude --version`). Plugin marketplaces are supported
  in current releases.

### 2. Register the marketplace

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
> step 5):
>
> ```sh
> git clone https://github.com/quyennguyenvu/grimoire.git
> claude plugin marketplace add ./grimoire   # or an absolute path
> ```

### 3. Install the plugin

```sh
claude plugin install grimoire-core@grimoire
```

Or in-session:

```text
/plugin install grimoire-core@grimoire
```

The plugin is **copied into Claude Code's plugin cache** at install time — it
does not run from your working tree.

### 4. Verify

- `/plugin` — opens the plugin manager; `grimoire-core` should be listed and
  enabled.
- `/help` — the `commit` command appears as **`/grimoire-core:commit`**.
- `/agents` — `code-reviewer`, `researcher`, `writer`, `finance-modeler`, and
  `presenter` are listed (under the `grimoire-core` plugin).
- The `senior-engineer` skill is available — ask for "senior-engineer mode" and
  Claude switches into the terse response style.

### 5. Update to the latest version

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

### 6. Uninstall

```sh
claude plugin uninstall grimoire-core@grimoire
```

Or in-session: `/plugin uninstall grimoire-core@grimoire`. To drop the
marketplace entirely: `claude plugin marketplace remove grimoire`.

### 7. Migrating from the old symlink setup

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

## Optional: run `commit` and `standup` without permission prompts

`standup` prints its log the moment you invoke it; `commit` drafts a message and,
once you confirm, commits it. In both cases the underlying `git` / `find` /
`date` work would normally raise a Claude Code tool-permission prompt. Two
`PreToolUse` hooks auto-approve exactly those commands so the flow isn't
interrupted — and the commit hook doubles as a guard that **blocks** any commit
that didn't come from `commit` (e.g. an auto-commit from another tool), so
nothing lands behind your back.

These hooks live in your **user-global** `~/.claude/settings.json` — _not_ in the
plugin. A marketplace plugin shouldn't silently alter your permission system, so
they are opt-in and per-machine. Without them the commands still work; Claude
just prompts before each git command (and before the commit itself). To enable
the frictionless flow, add both hooks:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "cmd=$(jq -r '.tool_input.command // \"\"'); if echo \"$cmd\" | grep -qE '\\bgit[[:space:]]+commit($|[^-a-zA-Z0-9_])'; then if echo \"$cmd\" | grep -qF 'GRIMOIRE_COMMIT_MSG'; then echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"allow\",\"permissionDecisionReason\":\"Commit via /commit (already confirmed in chat).\"}}'; else echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"Blocked: commits must go through the /commit command (it references GRIMOIRE_COMMIT_MSG). Do not commit any other way; if a commit is genuinely needed, tell the user to run /commit or commit manually in their own terminal.\"}}'; fi; fi"
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

- **Commit hook** — auto-approves a `git commit` only when it references the
  `GRIMOIRE_COMMIT_MSG` file that `commit` writes; every other `git commit` —
  including auto-commits from other tools — is **denied**. (Prefer a prompt over
  a hard block? Change that branch's `permissionDecision` from `deny` to `ask`.)
- **Standup hook** — auto-approves a Bash command only when its **first line** is
  the `# GRIMOIRE_STANDUP` marker that `standup` puts on its read-only scans.

If you already have a `PreToolUse` block matching `Bash`, **merge** these two
entries into its `hooks` array rather than replacing it. After saving, open
`/hooks` once (or restart Claude Code) so the config reloads.

> [!WARNING]
> These hooks approve commands by a marker string, which is forgeable — treat
> them as a convenience you opt into, not a security boundary. The standup hook
> only matches the `# GRIMOIRE_STANDUP` comment the command emits on a read-only
> scan; the commit hook **denies** any commit that didn't come from `commit`
> (manual commits in your own terminal are unaffected — hooks fire only on
> Claude's tool calls).

## Agents vs. skills vs. commands

- **Agents** (`agents/`) are _subagents_ — separate personas Claude delegates a
  task to. Each runs in its own context with a restricted tool set, then returns
  a result. Good for offloading focused, self-contained work ("review this
  diff", "size this market").
- **Skills** (`skills/`) modify how Claude itself responds in the main
  conversation. `senior-engineer` switches Claude into a terse, technically
  dense style.
- **Commands** (`commands/`) are parameterized prompts invoked with a slash.
  `commit` gathers the git diff and drafts a Conventional Commits message.

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
