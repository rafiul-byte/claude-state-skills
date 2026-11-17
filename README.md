# Claude Code session state skills

Two Claude Code skills that let you stop work and start again with no loss of context.

- `save-state` writes one small handoff file before you close the laptop.
- `load-state` reads that file in the next session and continues from the exact next step.

Both skills are built to use very few tokens. They read the current conversation and one git
block. They do not search the codebase.

## Install

Copy both folders into your user skills folder:

```bash
git clone https://github.com/rafiul-byte/claude-state-skills.git
cp -r claude-state-skills/skills/save-state ~/.claude/skills/save-state
cp -r claude-state-skills/skills/load-state ~/.claude/skills/load-state
```

For one project only, use `.claude/skills/` inside that project instead.

## Use

At the end of a session:

```
/save-state
```

It writes `.claude/state/LATEST.md` in the repo, copies it to `.claude/state/history/`, and adds
`.claude/state/` to `.gitignore`.

At the start of the next session:

```
/load-state
```

It reads the handoff, checks the real git state, prints a short summary, and starts the next step.

## What is in the handoff file

| Heading | Why it matters |
| --- | --- |
| Goal | What you are trying to make work |
| Status | Done, in progress, or blocked |
| Verified | Proved by real command output |
| Not verified | Written but never run |
| Next step | The single first action of the next session |
| After that | Up to five more steps, in order |
| Files touched | Path, what changed, committed or not |
| Commands that work | Copy ready |
| Decisions | Decision and the reason |
| Traps | What broke, and the fix that worked |
| Open question | Only if an answer is needed to continue |
| Git | Branch, uncommitted files, last commit, stash |

The four headings that decide quality are `Next step`, `Verified`, `Not verified` and `Traps`.
They must hold exact paths, exact commands and exact error text.

## Safety

- Neither skill runs `git push`, a deploy, or a paid API call.
- `load-state` stops and asks before a push, a deploy, a live workflow edit, a paid API call, or
  deleting files that are not backed up.
- Real command output always wins over the text in the handoff.

## Licence

MIT.
