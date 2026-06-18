# grimoire

A personal collection of custom **subagents** and **skills** for [Claude Code](https://claude.com/claude-code) — a book of reusable "spells" that give Claude focused, opinionated personas for code review, research, writing, financial modeling, presentations, and senior-engineer pair programming.

## What's inside

| Type  | Name                                               | What it does                                                                                                                                  |
| ----- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Agent | [code-reviewer](agents/code-reviewer.md)           | Read-only code review (Go-fluent) — correctness, error handling, concurrency, security, idioms. Returns prioritized findings, makes no edits. |
| Agent | [researcher](agents/researcher.md)                 | Web research, competitor scans, market sizing. Reads many sources and returns a distilled, cited summary.                                     |
| Agent | [writer](agents/writer.md)                         | Drafts and edits clean, persuasive prose — docs, listings, posts, emails, landing copy.                                                       |
| Agent | [finance-modeler](agents/finance-modeler.md)       | Cost models, unit economics, break-even, pricing scenarios, P&L. Produces auditable CSV/markdown with assumptions laid bare.                  |
| Agent | [presenter](agents/presenter.md)                   | Turns source docs and data into slide decks and visual reports with charts.                                                                   |
| Skill | [senior-engineer](skills/senior-engineer/SKILL.md) | Hyper-concise pair-programming mode: direct answer → code → trade-offs. Zero fluff, exact terminology.                                        |

## Agents vs. skills

- **Agents** (`agents/`) are _subagents_ — separate personas Claude can delegate a task to. Each runs in its own context with a restricted tool set, then returns a result. Useful for offloading focused, self-contained work (e.g. "review this diff", "size this market").
- **Skills** (`skills/`) modify how Claude itself responds in the main conversation. The `senior-engineer` skill switches Claude into a terse, technically dense communication style.

## Installation

Claude Code discovers agents and skills from `.claude/` directories. You can install these at the **user level** (available in every project) or per **project**.

### User-level (recommended)

Make them available everywhere by symlinking (or copying) into `~/.claude/`:

```sh
git clone git@github.com:quyennguyenvu/grimoire.git
cd grimoire

# Symlink so updates here flow through automatically
mkdir -p ~/.claude/agents ~/.claude/skills
ln -s "$PWD"/agents/*.md            ~/.claude/agents/
ln -s "$PWD"/skills/senior-engineer ~/.claude/skills/senior-engineer
```

### Project-level

Copy just the agents/skills you want into a project's `.claude/` directory:

```sh
mkdir -p your-project/.claude/agents
cp agents/code-reviewer.md your-project/.claude/agents/
```

## Usage

Once installed, Claude Code picks these up automatically.

- **Agents** are invoked by Claude when a task matches their description, or explicitly:
  > "Use the code-reviewer agent on this diff"
  > "Have the researcher size the market for X"
- **Skills** can be triggered by name as a slash command or by intent:
  > `/senior-engineer`
  > "Answer in senior-engineer mode"

## File format

Each agent/skill is a markdown file with YAML frontmatter:

```markdown
---
name: code-reviewer
description: >-
  When to use this agent... (Claude reads this to decide delegation)
tools: Read, Grep, Glob # restrict the agent's tool access
model: sonnet # optional model override
---

# Identity

...the system prompt that defines the persona...
```

| Field         | Purpose                                                                       |
| ------------- | ----------------------------------------------------------------------------- |
| `name`        | Identifier used to invoke the agent/skill.                                    |
| `description` | Tells Claude _when_ to use it — the most important field for auto-delegation. |
| `tools`       | (Agents) Comma-separated allowlist of tools the agent may use.                |
| `model`       | (Optional) Pin a specific model, e.g. `sonnet`.                               |

## Adding your own

1. Create `agents/<name>.md` (or `skills/<name>/SKILL.md`) with the frontmatter above.
2. Write a sharp `description` — this is what Claude matches against, so be concrete about _when_ to use it.
3. Keep the body focused: identity, method, output format, and hard rules.
4. Reinstall (or rely on the symlink) and Claude Code will discover it.

## Layout

```sh
grimoire/
├── agents/                 # subagent definitions
│   ├── code-reviewer.md
│   ├── finance-modeler.md
│   ├── presenter.md
│   ├── researcher.md
│   └── writer.md
└── skills/                 # response-mode skills
    └── senior-engineer/
        └── SKILL.md
```
