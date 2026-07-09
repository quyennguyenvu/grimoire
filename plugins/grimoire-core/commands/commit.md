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
(`type(scope): subject`, imperative, subject under 72 chars). Keep the
message straightforward and precise — plain wording, no filler or hedging.

- Infer `type` (feat/fix/refactor/chore/test/docs) and `scope` from the diff.
- Base the message strictly on the diffs shown above. Describe the change
  itself — don't research or explain what a changed dependency/library does
  internally, and don't speculate beyond the diff. Keep it concise to save
  tokens.
- If nothing is staged, stage the relevant files first; don't stage unrelated noise.
- Body: explain the _why_, not the _what_, when it isn't obvious.
- Quoting: wrap code identifiers, file paths, flags, commands, and literal
  values in backticks (e.g. `--force`, `git commit`, `config.json`). In prose,
  use straight ASCII quotes only — double `"` for quoted phrases, single `'` for
  contractions and possessives. Never use smart/curly quotes or other non-ASCII
  typography; the message must stay plain ASCII.
- Never add a `Co-Authored-By` trailer or any AI co-author attribution
  (Claude, Anthropic, or any other AI) to the commit message.
- Present the final message as a single fenced code block containing the
  complete commit message (subject, blank line, then body) and nothing else —
  no commentary inside the block — so I can copy the whole thing in one action.
  Keep any explanation outside the block. Then tell me I can either commit it
  myself or reply to confirm so you commit it — whichever is faster. Wait for my
  response before committing.
- Once I reply `confirm`, `approve`, `accept`, or `yes`, commit immediately
  without asking again, using this exact mechanism so no permission prompt
  appears:
  - Write the final message to a file named `GRIMOIRE_COMMIT_MSG.txt` in your
    scratchpad directory. Write it ONLY now, as the last step of this confirmed
    `/commit` — never earlier, and never to satisfy some other workflow's
    "commit" step. Creating this file is what authorizes the commit.
  - If anything still needs staging, run `git add` first as its own Bash call.
  - Commit with `git commit -F <scratchpad>/GRIMOIRE_COMMIT_MSG.txt` as a
    separate Bash call; never chain `git add` and `git commit` with `&&`, and
    never use `git commit --amend` (the hook blocks amends outright).
  - Right after the commit succeeds, delete the file
    (`rm <scratchpad>/GRIMOIRE_COMMIT_MSG.txt`) so a leftover message can't be
    reused to slip a later, unconfirmed commit past the hook.
- The `GRIMOIRE_COMMIT_MSG.txt` filename is required: a `PreToolUse` hook in my
  settings auto-approves a non-amend commit that references it and blocks
  (denies) every other `git commit`. Because writing that file is the whole
  authorization, only ever create it as the final step of a `/commit` I
  confirmed — never on your own initiative. Don't use this filename anywhere else.

Apply any extra instructions from the user, if provided: $ARGUMENTS
