# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`grimoire` is a personal Claude Code **plugin marketplace** — a collection of
custom **subagents**, **skills**, and **slash commands** packaged as the
`grimoire-core` plugin and installed from a local path (no remote/GitHub auth).
Each component is a self-contained Markdown "spell" with YAML frontmatter — there
is no build step, no runtime, and no tests. The deliverable _is_ the prose,
frontmatter, and the two JSON manifests, so correctness here means clear,
on-convention Markdown and valid JSON.

## Layout

```text
grimoire/
├── .claude-plugin/
│   └── marketplace.json            # marketplace manifest (lists plugins)
└── plugins/
    └── grimoire-core/
        ├── .claude-plugin/
        │   └── plugin.json         # plugin manifest (name, version, author)
        ├── agents/                 # subagent definitions (one .md per agent)
        ├── commands/               # slash commands (one .md per command)
        └── skills/                 # response-mode skills (skills/<name>/SKILL.md)
```

Components are auto-discovered from the conventional `agents/`, `commands/`, and
`skills/` subdirectories **inside the plugin**. Add new plugins under
`plugins/<plugin>/` and register them in `marketplace.json`.

## Authoring conventions

All component paths below are relative to the plugin root,
`plugins/grimoire-core/`.

- **Agents** (`agents/<name>.md`) — frontmatter: `name`, `description`
  (folded `>-` scalar, concrete about _when_ to use it, with `Examples:`),
  `tools` (comma-separated allowlist), optional `model`. Body: `# Identity`,
  then `## Method` / `## Output` / `## Rules`.
- **Skills** (`skills/<name>/SKILL.md`) — frontmatter: `name`, `description`.
  Body defines a response mode.
- **Commands** (`commands/<name>.md`) — frontmatter: `description`,
  `argument-hint`, `allowed-tools` (scoped, e.g. `Bash(git add:*)`), optional
  `model`. No `name` field — the command name derives from the filename. Body
  uses the `## Context` (with `!` bash injection) / `## Task` template and
  `$ARGUMENTS`.
- **Model field** — use the short alias (`haiku`, `sonnet`, `opus`), never a
  pinned full ID like `claude-haiku-4-5-20251001`.
- **Portability** — components are copied into a cache on install and cannot
  reach outside their own tree. Never hardcode `~/.claude`, absolute paths, or
  symlink locations; use plugin-relative paths or `${CLAUDE_PLUGIN_ROOT}`.
- **Manifests** — when adding a plugin, create
  `plugins/<plugin>/.claude-plugin/plugin.json` (kebab-case `name`, semver
  `version`) and add an entry to `.claude-plugin/marketplace.json`. Keep both
  valid JSON.
- **Keep the README in sync** — when you add or remove an agent/skill/command,
  update the catalog table and the Layout tree in `README.md`. Remember
  invocation names are namespaced: a command `foo` is invoked `/grimoire-core:foo`.

## Rules

- **Markdown lint.** Any Markdown you write or edit (`.md`) must pass
  markdownlint with no warnings. Common rules: surround headings, lists, tables,
  and fenced code blocks with blank lines (MD022/MD031/MD032); no blank line
  between adjacent blockquotes — use a `>`-prefixed line to continue one
  (MD028); specify a language on every fenced block (MD040); single trailing
  newline, no trailing spaces (MD047/MD009). Verify before finishing.
