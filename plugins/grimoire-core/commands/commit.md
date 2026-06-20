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
- Show me the message and wait for confirmation before running `git commit`.
- Once I reply `confirm`, `yes`, or `accept`, run `git commit` immediately —
  do not ask again or request further permission.
- Run `git add` and `git commit` as separate Bash calls; never chain them with
  `&&`. Each call then matches its own allowlist rule and avoids a permission
  prompt.

Apply any extra instructions from the user, if provided: $ARGUMENTS
