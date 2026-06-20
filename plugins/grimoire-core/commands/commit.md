---
description: Stage changes and create a commit with a Conventional Commits message
argument-hint: "[optional extra context or instructions]"
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git diff:*), Bash(git commit:*), Bash(git log:*)
model: haiku
---

# Commit

## Context

- Status: !`git status --short`
- Staged diff: !`git diff --staged`
- Unstaged diff: !`git diff`
- Recent messages (for style): !`git log --oneline -10`

## Task

Write a commit for the changes above following Conventional Commits
(`type(scope): subject`, imperative, subject under 72 chars).

- Infer `type` (feat/fix/refactor/chore/test/docs) and `scope` from the diff.
- If nothing is staged, stage the relevant files first; don't stage unrelated noise.
- Body: explain the _why_, not the _what_, when it isn't obvious.
- Never add a `Co-Authored-By` trailer or any AI co-author attribution
  (Claude, Anthropic, or any other AI) to the commit message.
- Show me the message and wait for confirmation before committing.
- Once I reply `confirm`, `approve`, `accept`, or `yes`, commit immediately
  without asking again, using this exact mechanism so no permission prompt
  appears:
  - Write the final message to a file named `GRIMOIRE_COMMIT_MSG.txt` in your
    scratchpad directory.
  - If anything still needs staging, run `git add` first as its own Bash call.
  - Commit with `git commit -F <scratchpad>/GRIMOIRE_COMMIT_MSG.txt` as a
    separate Bash call; never chain `git add` and `git commit` with `&&`.
- The `GRIMOIRE_COMMIT_MSG.txt` filename is required: a `PreToolUse` hook in my
  settings auto-approves any commit that references it and forces a confirmation
  prompt on every other `git commit`. Don't use this filename anywhere else.

Apply any extra instructions from the user, if provided: $ARGUMENTS
