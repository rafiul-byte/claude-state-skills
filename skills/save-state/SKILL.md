---
name: save-state
description: Save a complete handoff of the current work session to disk before the laptop is closed, so the next session can continue with no loss of context. Use when the user types /save-state, or says "save state", "save before I close", "I am shutting down", "end of day", or otherwise stops work for the day.
allowed-tools: Bash, Read, Write, Edit
---

# Save state

Write one handoff file that lets a fresh session continue this work with no quality loss.

## Token rules (follow these exactly)

- Build the handoff from what is ALREADY in this conversation. Do not re-read files you have read.
- Do not search the codebase. No Grep, no Glob, no subagents, no web calls.
- Run the git block once. That is the only shell work allowed.
- Keep the file under about 150 lines. Signal only. No summaries of things that do not affect the next step.

## Step 1. Collect ground truth (one command)

```bash
cd "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
mkdir -p .claude/state/history
date '+%Y-%m-%d %H:%M'
echo '--- branch ---'; git branch --show-current 2>/dev/null
echo '--- uncommitted ---'; git status --short 2>/dev/null | head -40
echo '--- commits ---'; git log --oneline -5 2>/dev/null
echo '--- stash ---'; git stash list 2>/dev/null | head -5
```

## Step 2. Write the file

Write `.claude/state/LATEST.md` using the template below. Fill every heading. If a heading has
nothing, write `none`. Never leave a heading out.

Quality guard. These four headings decide whether the next session is as good as this one:
`Next step`, `Verified`, `Not verified`, `Traps`. Be concrete in them. Exact paths, exact
commands, exact error text. Never write "continue the work" or "fix the bug".

## Step 3. Archive and ignore

```bash
cp .claude/state/LATEST.md ".claude/state/history/$(date +%Y%m%d-%H%M).md"
grep -qxF '.claude/state/' .gitignore 2>/dev/null || echo '.claude/state/' >> .gitignore
```

## Step 4. Report

Three lines to the user, no more:
1. Saved, and the path.
2. The one next step that was recorded.
3. `Next session: run /load-state`

## Template

```markdown
# Handoff. <YYYY-MM-DD HH:MM>

## Goal
What we are trying to make work. One or two lines.

## Status
Done, in progress, or blocked. One line.

## Verified
Things proved by real output. Name the proof. Example:
`docker ps shows dbbl-n8n up 4 min`

## Not verified
Things written but never run, or run but never seen working end to end.

## Next step
The single first action of the next session. Exact command, or exact file and line.

## After that
Up to 5 more steps, in order.

## Files touched
`path` - what changed - committed or uncommitted

## Commands that work
Exact, copy ready. Include how to start the stack and how to test.

## Decisions
Decision - why. Only the ones that would be redone wrong without this note.

## Traps
What already broke, and the fix that worked.

## Open question for the user
Only if a real answer is needed before work can continue. Otherwise `none`.

## Git
branch:
uncommitted:
last commit:
stash:
```

## Rules

- No em-dashes in the file.
- Never run `git push`, a deploy, or a paid API call in this skill.
- Do not commit anything unless the user asks in the same message.
