# grimoire

A personal Claude Code **plugin marketplace** — a book of reusable "spells"
(subagents, skills, and slash commands) that give [Claude Code](https://claude.com/claude-code)
focused, opinionated personas for code review, research, writing, financial
modeling, presentations, senior-engineer pair programming, and Conventional
Commits.

Everything ships as a single plugin, **`grimoire-core`**, installed from a
**local path** — no remote, no GitHub auth, nothing leaves your machine.

> [!IMPORTANT]
> **Invocation names are namespaced after install.** Slash commands are prefixed
> with the plugin name: the `commit` command is invoked as
> **`/grimoire-core:commit`**, not `/commit`. Agents and skills are still
> referred to by their bare name (e.g. "use the `code-reviewer` agent").

## What's inside

The marketplace (`grimoire`) currently publishes one plugin, `grimoire-core`:

| Component | Type | What it does | Invoke as |
| --- | --- | --- | --- |
| `code-reviewer` | Agent | Read-only code review (Go-fluent) — correctness, error handling, concurrency, security, idioms. Returns prioritized findings, makes no edits. | delegated by description, or "use the `code-reviewer` agent" |
| `researcher` | Agent | Web research, competitor scans, market sizing. Reads many sources, returns a distilled, cited summary. | "have the `researcher` size the market for X" |
| `writer` | Agent | Drafts and edits clean, persuasive prose — docs, listings, posts, emails, landing copy. | "use the `writer` to draft …" |
| `finance-modeler` | Agent | Cost models, unit economics, break-even, pricing scenarios, P&L. Auditable CSV/markdown with assumptions laid bare. | "use the `finance-modeler` for …" |
| `presenter` | Agent | Turns source docs and data into slide decks and visual reports with charts. | "use the `presenter` to build a deck" |
| `senior-engineer` | Skill | Hyper-concise pair-programming mode: direct answer → code → trade-offs. Zero fluff, exact terminology. | auto by intent, or ask for "senior-engineer mode" |
| `commit` | Command | Stages changes and writes a Conventional Commits message from the diff. | **`/grimoire-core:commit`** |

## Setup

### 1. Prerequisites

- **Claude Code ≥ 2.1** (`claude --version`). Plugin marketplaces are supported
  in current releases.
- The repo cloned somewhere on disk. It can live anywhere — the examples below
  assume:

  ```sh
  # wherever you keep it; substitute your own path throughout
  GRIMOIRE=~/workspace/gh_leo/grimoire
  ```

  All commands work equally with an absolute path or, when run from inside the
  repo, with `.`.

### 2. Register the marketplace (local path)

From a terminal:

```sh
claude plugin marketplace add "$GRIMOIRE"
# …or, from inside the repo:
#   cd "$GRIMOIRE" && claude plugin marketplace add .
```

Or in an interactive Claude Code session:

```text
/plugin marketplace add ~/workspace/gh_leo/grimoire
```

This reads `.claude-plugin/marketplace.json` and registers a marketplace named
**`grimoire`**. Because the source is a local path, nothing is fetched remotely.

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

### 5. Update after editing the repo

Because the plugin is copied at install time, edits to the repo only take effect
after you refresh the marketplace:

```sh
cd "$GRIMOIRE" && git pull          # if you track changes remotely
claude plugin marketplace update grimoire
```

Or in-session, then reload so non-skill components (commands/agents) re-read:

```text
/plugin marketplace update grimoire
/reload-plugins
```

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
│       │   └── writer.md
│       ├── commands/
│       │   └── commit.md
│       └── skills/
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
