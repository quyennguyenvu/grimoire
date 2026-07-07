---
description: Reconstruct a copy-pasteable standup-style log from your git commit history. Discovers repos from the current directory (a single repo, or every repo under a container), resolves your author identity per-repo via git config (never hardcoded), and drafts Done / In progress / Blockers. Windows use explicit local-midnight boundaries (default today; --yesterday, --week, --last-week, or a YYYY-MM-DD date).
argument-hint: "[--today (default) | --yesterday | --week | --last-week | YYYY-MM-DD]"
allowed-tools: Bash(git rev-parse:*), Bash(git config:*), Bash(git log:*), Bash(git status:*), Bash(git branch:*), Bash(find:*), Bash(dirname:*), Bash(basename:*), Bash(sort:*), Bash(grep:*), Bash(date:*)
model: haiku
---

# Standup

Reconstruct what I did from my git commit history and hand back a single,
copy-pasteable standup-style Markdown block. Output only — I copy the whole
block and paste it into a journal, Slack, or a doc, so it must be one clean
block with none of your commentary inside it.

## Context

- Scope: current directory (single repo, or every repo one level under a container)
- Identity: resolved per-repo via `git config user.email` (picks up `includeIf` automatically)
- Window: from the argument (`$ARGUMENTS`); empty means today. Boundaries are explicit local midnight — NOT git's `midnight`/`yesterday`/`last monday` keywords, which resolve to the current time of day and silently drop early commits
- Data: commits in the window (no merges, author-filtered) + working state (uncommitted changes and branch) for In progress

## Task

Write my standup from my git commit history.

1. **Resolve the window and gather commits.** Set `ARG` to the argument I passed
   (`$ARGUMENTS`; empty = today), then run this block. It computes explicit
   local-midnight boundaries with `date`, discovers repos, and filters to my
   per-repo identity:

   ```bash
   ARG="$ARGUMENTS"; ROOT="$PWD"; UNTIL=""
   dow=$(date +%u)                          # macOS/BSD date; 1=Mon..7=Sun
   mon="$(date -v-$((dow-1))d +%F)"         # this week's Monday
   lastmon="$(date -v-$((dow-1+7))d +%F)"   # last week's Monday
   case "$ARG" in
     ""|--today|today)          SINCE="$(date +%F) 00:00" ;;
     --yesterday|yesterday)     SINCE="$(date -v-1d +%F) 00:00"; UNTIL="$(date +%F) 00:00" ;;
     --week|--this-week|week)   SINCE="$mon 00:00" ;;
     --last-week|last-week)     SINCE="$lastmon 00:00"; UNTIL="$mon 00:00" ;;
     [0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]) SINCE="$ARG 00:00" ;;
     *)                         SINCE="$ARG" ;;    # any git --since expression
   esac
   { if git -C "$ROOT" rev-parse --show-toplevel >/dev/null 2>&1; then
       git -C "$ROOT" rev-parse --show-toplevel
     else
       find "$ROOT" -maxdepth 2 -name .git -not -path '*/node_modules/*' -exec dirname {} \; | sort -u
     fi
   } | while IFS= read -r R; do
     EMAIL="$(git -C "$R" config user.email)"; [ -z "$EMAIL" ] && continue
     git -C "$R" log --all --no-merges --since="$SINCE" ${UNTIL:+--until="$UNTIL"} \
       --author="$EMAIL" --pretty=tformat:"$(basename "$R")|%h|%ad|%s" --date=short-local
   done; true
   ```

   Window reference: *(none)* or `--today` = since today 00:00; `--yesterday` =
   yesterday 00:00 up to today 00:00; `--week` = since this week's Monday 00:00;
   `--last-week` = last Monday 00:00 up to this Monday 00:00; a bare `YYYY-MM-DD`
   = since that date 00:00.

2. **Identity is per repo.** The block resolves it with `git config user.email`,
   which picks up directory-scoped `includeIf` identities — so different orgs
   resolve to different emails on their own. Never hardcode an email or use
   `--global`. Repos with no identity are skipped.
3. **Synthesize, don't transcribe.** Group by repo and collapse each repo's
   commits into **one** plain-language bullet (map `feat` → added/shipped,
   `fix` → fixed, `refactor` → reworked; drop the `type(scope):` prefix). The
   reader wants the story, not raw subject lines.
4. **Emit exactly one fenced `markdown` block**, nothing else inside it, so I can
   paste it verbatim (the outer fence below is only to display the shape). Title
   the block with the day (`## 2026-07-07 — Standup`) for a single-day window, or
   the range (`## 2026-07-06 → 2026-07-07 — Standup`) for a multi-day one:

   ````text
   ```markdown
   ## 2026-07-07 — Standup

   **Done today**

   - duvel: shipped consumer "comment is gone" messages, fixed the creator react
     count, and published action-log events for post mutations

   **In progress**

   - duvel: on `feature/comm-studio`, uncommitted changes

   **Blockers**

   - none
   ```
   ````

5. **In progress** comes from the working state of the same repos (independent of
   the window). Run this (same discovery, so it also works in container mode
   where you need each repo path, not just its name):

   ```bash
   ROOT="$PWD"
   { if git -C "$ROOT" rev-parse --show-toplevel >/dev/null 2>&1; then
       git -C "$ROOT" rev-parse --show-toplevel
     else
       find "$ROOT" -maxdepth 2 -name .git -not -path '*/node_modules/*' -exec dirname {} \; | sort -u
     fi
   } | while IFS= read -r R; do
     D="$(git -C "$R" status --porcelain 2>/dev/null | grep -c .)"
     B="$(git -C "$R" branch --show-current 2>/dev/null)"
     [ "$D" -gt 0 ] && printf '%s: branch %s, %s uncommitted file(s)\n' "$(basename "$R")" "${B:-?}" "$D"
   done; true
   ```

   Use "—" if every repo is clean. **Blockers** is always `- none`, a
   placeholder for me to fill — never invent blockers.
6. If no commits match the window, don't print an empty log: say so and suggest a
   wider window (`--week`, `--last-week`). Put any footnote (repos scanned,
   window range) **outside** the block so it is never pasted.

Apply any extra instructions from the user, if provided: $ARGUMENTS
