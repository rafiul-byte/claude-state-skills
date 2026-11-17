---
name: load-state
description: Resume work from the handoff saved by /save-state, then continue from the exact next step. Use when the user types /load-state, or says "load state", "resume", "continue where we left off", "where were we", or opens the laptop to carry on yesterday's work.
allowed-tools: Bash, Read, Write, Edit, Grep, Glob
---

# Load state

Restart the previous session from one small file, then keep working.

## Token rules (follow these exactly)

- Read `.claude/state/LATEST.md` and nothing else from `.claude/state/`.
- Never read `.claude/state/history/`. It is an archive for humans.
- Do not re-explore the codebase. Do not run a search to "get oriented". No subagents.
- Open only the file named in `Next step`. Open more files only when the work needs them.

## Step 1. Read the handoff

```bash
cd "$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
cat .claude/state/LATEST.md
```

If the file does not exist: tell the user there is no saved state, and ask whether to start fresh.
Stop there.

## Step 2. Check reality (one command)

```bash
echo '--- branch ---'; git branch --show-current 2>/dev/null
echo '--- uncommitted ---'; git status --short 2>/dev/null | head -40
echo '--- commits ---'; git log --oneline -3 2>/dev/null
```

Compare with the `Git` section of the handoff. If they differ, for example new commits exist or
changed files are gone, say so in one line before doing anything else. The real repo wins over
the handoff.

## Step 3. Report in this shape

Maximum 8 lines.

```
Last time: <Goal, one line>
Stopped at: <Status, one line>
Working: <Verified, short>
Not proved: <Not verified, short>
Next step: <the exact next step>
```

If the handoff has an `Open question for the user`, ask it now and stop.

## Step 4. Continue the work

Start the `Next step` at once. Do not wait for permission.

Stop and ask first only if the next step is one of these:
- `git push`, a deploy, a production migration, or editing a live workflow
- a live paid API call
- deleting or overwriting files that are not backed up

For those, state the exact action and ask "OK to proceed?".

## Step 5. Keep it fresh

After finishing a meaningful step, update `Next step` and `Status` inside
`.claude/state/LATEST.md` with a small Edit. Do not rewrite the whole file.

## Rules

- No em-dashes in anything written.
- Trust real command output over the handoff text.
- If the handoff is vague, ask one sharp question instead of guessing.
