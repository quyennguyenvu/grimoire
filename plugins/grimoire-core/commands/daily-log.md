---
description: Reconstruct a copy-pasteable standup-style daily log from your git commit history. Discovers repos from the current directory (a single repo, or every repo under a container), resolves your author identity per-repo via git config (never hardcoded), and drafts Done / In progress / Blockers. Pass a time window like today (default), yesterday, or "this week".
argument-hint: "[time window: today | yesterday | this week | since 2026-07-01]"
allowed-tools: Bash(git rev-parse:*), Bash(git config:*), Bash(git log:*), Bash(git status:*), Bash(git branch:*), Bash(find:*), Bash(dirname:*), Bash(basename:*), Bash(sort:*), Bash(grep:*)
model: haiku
---

# Daily Log

Reconstruct what I did from my git commit history and hand back a single,
copy-pasteable standup-style Markdown block. Output only — I copy the whole
block and paste it into a journal, Slack, or a doc, so it must be one clean
block with none of your commentary inside it.

## Context

- Today's commits, discovered from the current directory and filtered to my identity per repo (all branches, no merges): !`ROOT="$PWD"; { if git -C "$ROOT" rev-parse --show-toplevel >/dev/null 2>&1; then git -C "$ROOT" rev-parse --show-toplevel; else find "$ROOT" -maxdepth 2 -name .git -not -path '*/node_modules/*' -exec dirname {} \; | sort -u; fi; } | while IFS= read -r R; do EMAIL="$(git -C "$R" config user.email)"; [ -z "$EMAIL" ] && { printf 'skip: %s (no git identity)\n' "$R"; continue; }; OUT="$(git -C "$R" log --all --no-merges --since=midnight --author="$EMAIL" --pretty=format:'%h|%ad|%s' --date=short 2>/dev/null)"; [ -n "$OUT" ] && printf '== %s <%s> ==\n%s\n\n' "$(basename "$R")" "$EMAIL" "$OUT"; done; true`
- Working state for the In progress section (repos with uncommitted changes): !`ROOT="$PWD"; { if git -C "$ROOT" rev-parse --show-toplevel >/dev/null 2>&1; then git -C "$ROOT" rev-parse --show-toplevel; else find "$ROOT" -maxdepth 2 -name .git -not -path '*/node_modules/*' -exec dirname {} \; | sort -u; fi; } | while IFS= read -r R; do D="$(git -C "$R" status --porcelain 2>/dev/null | grep -c .)"; B="$(git -C "$R" branch --show-current 2>/dev/null)"; [ "$D" -gt 0 ] && printf '%s: branch %s, %s uncommitted file(s)\n' "$(basename "$R")" "${B:-?}" "$D"; done; true`

## Task

Write my daily log from the commits above.

1. **Window.** The commits above already cover today. If `$ARGUMENTS` names a
   different window (`yesterday`, `this week`, `since 2026-07-01`, ...), ignore
   the injected commits and re-run this block via Bash with `WINDOW` set to the
   matching `git --since` value, then use its output instead:

   ```bash
   ROOT="$PWD"; WINDOW="yesterday"   # set from $ARGUMENTS: "1 week ago", "last monday", a date, ...
   { if git -C "$ROOT" rev-parse --show-toplevel >/dev/null 2>&1; then
       git -C "$ROOT" rev-parse --show-toplevel
     else
       find "$ROOT" -maxdepth 2 -name .git -not -path '*/node_modules/*' -exec dirname {} \; | sort -u
     fi
   } | while IFS= read -r R; do
     EMAIL="$(git -C "$R" config user.email)"; [ -z "$EMAIL" ] && continue
     git -C "$R" log --all --no-merges --since="$WINDOW" --author="$EMAIL" \
       --pretty=format:"$(basename "$R")|%h|%ad|%s" --date=short
   done; true
   ```

2. **Identity is per repo.** The block resolves it with `git config user.email`,
   which picks up directory-scoped `includeIf` identities — so different orgs
   resolve to different emails on their own. Never hardcode an email or use
   `--global`. Repos with no identity are skipped.
3. **Synthesize, don't transcribe.** Group by repo and collapse each repo's
   commits into **one** plain-language bullet (map `feat` → added/shipped,
   `fix` → fixed, `refactor` → reworked; drop the `type(scope):` prefix). The
   reader wants the story of the day, not raw subject lines.
4. **Emit exactly one fenced `markdown` block**, nothing else inside it, so I can
   paste it verbatim (the outer fence below is only to display the shape):

   ````text
   ```markdown
   ## 2026-07-06 — Daily log

   **Done today**

   - duvel: shipped consumer "comment is gone" messages, fixed the creator react
     count, and published action-log events for post mutations

   **In progress**

   - duvel: on `feature/comm-studio`, uncommitted changes

   **Blockers**

   - none
   ```
   ````

5. **In progress** comes from the working-state list above (use "—" if it is
   empty). **Blockers** is always `- none`, a placeholder for me to fill — never
   invent blockers.
6. If no commits match the window, don't print an empty log: say so and suggest a
   wider window (`yesterday`, `this week`). Put any footnote (repos scanned,
   window) **outside** the block so it is never pasted.

Apply any extra instructions from the user, if provided: $ARGUMENTS
