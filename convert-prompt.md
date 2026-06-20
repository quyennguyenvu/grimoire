# Convert prompt

You're working in my `grimoire` repo — a personal, PRIVATE Claude Code
marketplace. It currently has top-level `skills/`, `commands/`, and `agents/`
directories that I symlink into `~/.claude/`. Convert grimoire into a proper
plugin marketplace installed via a LOCAL-PATH source (private repo — avoid any
remote/GitHub auth flow), then update the README with exact step-by-step setup
instructions. Work in phases and check in with me before any file moves or
deletions.

PHASE 0 — Inventory & plan (NO changes yet)

- List every file under skills/, commands/, agents/. Confirm each skill is a
  directory containing SKILL.md.
- Read each command/agent/skill. Flag any that reference absolute paths,
  `~/.claude`, or assume a symlink location — these need fixing in Phase 2.
- Detect whether components cluster into clear domains (git, research, finance, etc.).
- Propose a plugin layout. DEFAULT: a single plugin `grimoire-core` under
  `plugins/grimoire-core/`, unless clear domain clusters justify splitting into
  multiple plugins. Show me the proposed directory tree + plugin list and WAIT
  for my approval before moving anything.

PHASE 1 — Restructure (after approval; use `git mv` to preserve history)

- Create `plugins/<plugin>/` with `commands/`, `agents/`, `skills/` subdirs as needed.
- `git mv` existing components in; keep each skill's dir + SKILL.md intact.
- Create `plugins/<plugin>/.claude-plugin/plugin.json` per plugin (schema below).
- Create `.claude-plugin/marketplace.json` at repo root (schema below), one entry
  per plugin with `source: "./plugins/<plugin>"`.

PHASE 2 — Fix portability

- Installed plugins are COPIED into a cache dir and cannot reference files outside
  their own tree. Replace any hardcoded `~/.claude/...` or absolute paths in
  commands/agents/hooks/.mcp.json with `${CLAUDE_PLUGIN_ROOT}` or plugin-relative
  paths. Flag anything you can't safely rewrite.

PHASE 3 — README

- Rewrite README.md to cover: what grimoire is now (a local plugin marketplace);
  a catalog table (plugin → components → invocation namespace); and a
  copy-pasteable setup guide:
  1. Prerequisites (clone location, Claude Code version).
  2. Register locally: `claude plugin marketplace add ~/grimoire`
     (and in-session `/plugin marketplace add ~/grimoire`).
  3. Install: `claude plugin install <plugin>@grimoire`.
  4. Verify: `/plugin`, `/help` (commands appear as `/<plugin>:<command>`), `/agents`.
  5. Update: `git pull` then `/plugin marketplace update grimoire`
     (+ `/reload-plugins` for non-skill changes).
  6. Uninstall: `/plugin uninstall <plugin>@grimoire`.
  7. Migration from the old symlink setup (Phase 4 steps).
- Call out the namespacing change (`/foo` → `/<plugin>:foo`) prominently, since
  invocation names change.

PHASE 4 — Deprecate symlinks (CONFIRM before deleting)

- Show me the symlinks under `~/.claude/{skills,commands,agents}` that point into
  this repo. Record their targets, then remove them ONLY after I confirm, so the
  installed plugin doesn't conflict with the old symlinked copies.

PHASE 5 — Verify & commit

- Actually run `claude plugin marketplace add .` then install each plugin, and
  confirm components load (`/help`, `/agents`, skill namespace). Report results
  and fix any issues.
- Commit in logical steps with Conventional Commits (e.g.
  `refactor(plugins): move components into grimoire-core`,
  `docs(readme): add plugin install guide`). Do NOT push.

CONSTRAINTS

- Preserve git history (`git mv`, never delete+recreate).
- Valid JSON; plugin names kebab-case; semver versions.
- Never delete anything without showing me first. Don't touch files outside the
  repo except the symlinks I approve in Phase 4.

SCHEMAS
`plugins/<plugin>/.claude-plugin/plugin.json`:

```json
{
  "name": "grimoire-core",
  "version": "0.1.0",
  "description": "<desc>",
  "author": { "name": "<you>" }
}
```

`.claude-plugin/marketplace.json` (repo root):

```json
{
  "name": "grimoire",
  "owner": { "name": "<you>" },
  "plugins": [{ "name": "grimoire-core", "source": "./plugins/grimoire-core" }]
}
```
