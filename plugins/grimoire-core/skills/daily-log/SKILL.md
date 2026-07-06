---
name: daily-log
description: Summarize what you did across your git repos into a daily standup-style log. Use when the user wants a daily log, work journal, standup notes, or "what did I do today" from commit history. Discovers repos from the current directory (a single repo, or every repo under a container), resolves the author identity per-repo via git config (never hardcoded), filters commits to a time window (today by default), and drafts Done / In progress / Blockers. Ask to widen the window (yesterday, this week) or change the format.
---

# Daily Log Mode

Reconstruct what the user did from their git commit history and hand back a
single, copy-pasteable standup-style daily log. Output goes to the terminal
only — the user copies the whole block and pastes it into a journal, Slack, or a
doc, so the log must be one clean Markdown block with none of your commentary
inside it.

## Method

1. **Resolve scope.** The scan root is the path or glob the user named,
   otherwise the current directory (`$PWD`). If the root is inside a git repo,
   scan that one repo; otherwise treat it as a container and discover every repo
   beneath it. Never assume a fixed location like `~/workspace` — derive it.
2. **Discover repos and read commits.** Run this portable block. It is written
   to work in both `bash` and `zsh`, so use the `while read` loop as shown — do
   **not** rewrite it as `for R in $REPOS`, which does not word-split in zsh and
   collapses every path into one.

   ```bash
   ROOT="${TARGET:-$PWD}"          # the path/glob the user named, else cwd
   WINDOW="${WINDOW:-midnight}"    # today by default; see step 4 for others
   {
     if git -C "$ROOT" rev-parse --show-toplevel >/dev/null 2>&1; then
       git -C "$ROOT" rev-parse --show-toplevel                    # single repo
     else
       find "$ROOT" -maxdepth 2 -name .git -not -path '*/node_modules/*' \
         -exec dirname {} \; | sort -u                             # container
     fi
   } | while IFS= read -r R; do
     EMAIL="$(git -C "$R" config user.email)"                      # per-repo identity
     [ -z "$EMAIL" ] && { printf 'skip: %s (no git identity)\n' "$R"; continue; }
     OUT="$(git -C "$R" log --all --no-merges --since="$WINDOW" \
             --author="$EMAIL" --pretty=format:'%h|%ad|%s' --date=short 2>/dev/null)"
     [ -n "$OUT" ] && printf '== %s  <%s> ==\n%s\n\n' "$(basename "$R")" "$EMAIL" "$OUT"
   done
   ```

3. **Trust per-repo identity.** The block resolves the author with
   `git -C "$R" config user.email`, which picks up directory-scoped
   `includeIf` identities — so different orgs resolve to different emails
   automatically. Never use `--global` and never hardcode an email. A repo with
   no configured identity is skipped with a note (don't fall back to dumping
   teammates' commits). Honor an explicit author override, or an "all authors"
   request, only when the user asks for one.
4. **Pick the window.** Default `midnight` (today). If the user names a period,
   translate it to a `--since` value: `yesterday`, `"last monday"`,
   `"3 days ago"`, `"1 week ago"`, or a date like `2026-07-01`.
5. **Handle an empty result.** If no commits match the window, say so plainly
   and suggest widening it (for example `yesterday` or `this week`) instead of
   printing an empty log — early in the day "today" is legitimately empty.
6. **Synthesize, don't transcribe.** Group the commits by repo and collapse each
   repo's commits into **one** plain-language bullet describing the day's work
   there — not one line per commit. Translate Conventional-Commit types into
   plain verbs (`feat` → added/shipped, `fix` → fixed, `refactor` → reworked,
   `chore`/`docs`/`test` → keep light) and drop the `type(scope):` prefix.
7. **Note work in progress.** For each scanned repo, check
   `git -C "$R" status --porcelain` (uncommitted changes) and
   `git -C "$R" branch --show-current` (a feature branch off `main`/`master`/
   `develop`). List those under "In progress"; use "—" when everything is clean.
8. **Emit one block.** Deliver the log as a single fenced `markdown` block
   containing nothing but the standup, so the user can select and paste it
   verbatim. Put any coverage footnote (window scanned, repo count) or
   empty-result note **outside** the block, after it, so it is never pasted.

## Output shape

Emit exactly one fenced `markdown` block, shaped like this (the outer fence
below is only to display it):

````text
```markdown
## 2026-07-06 — Daily log

**Done today**

- duvel: shipped consumer "comment is gone" messages, fixed the creator react
  count, and published action-log events for post mutations

**In progress**

- hoegaarden: on `feature/comm-studio`, uncommitted changes

**Blockers**

- none
```
````

Then, on a separate line outside the block, a short footnote such as
`Scanned 6 repos under ~/workspace/gh_leo for commits since midnight.`

## Rules

- **Identity is per-repo.** Always resolve the author with
  `git -C "$R" config user.email`. Never `--global`, never a literal email in
  the command.
- **Always author-filter.** Team repos have many committers; an unfiltered log
  is wrong. Only widen to all authors on explicit request.
- **Synthesize, don't dump.** One bullet per repo in plain language. The reader
  wants the story of the day, not raw subject lines.
- **Never invent blockers.** git can't know them — leave `- none` as a
  placeholder for the user to fill.
- **One copyable block.** Emit exactly one fenced `markdown` block with no
  prose inside it; footnotes and suggestions live outside it.
- **Plain ASCII, Markdownlint-clean.** No smart quotes; blank lines around
  headings, lists, and fenced blocks; a language on every fence; single trailing
  newline.
