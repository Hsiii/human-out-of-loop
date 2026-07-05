# human-out-of-loop

Codex skills to turn issues into reviewed PRs automaticaly.

## Prerequisite

These skills expect the `ponytail` skill to be installed for implementation and review threads. If you do not have it, install it from [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail).

## Skills

- `solve-issues`: triggered by "solve issues" or "building loops"; fetches open issues, determines internal/external mode, creates a neutral worktree seed plus two visible ponytail Codex app worker threads per issue, and calls `$solve-issue` with a task packet. The implementation and review workers are same-directory siblings forked from the neutral seed, never from each other, so role context does not leak between them.
- `solve-issue`: runs one issue only; it receives the task packet, coordinates implementation/review heartbeat, and stops on the explicit `PASS` marker.
- `pr`: triggered by "pr"; internal mode uses the repo template, Yeet, and browser media upload to create a real draft PR, while external mode writes gitignored `DRAFT.md` and `.codex-pr-media/` without opening a GitHub PR.

## Automation

Ask Codex to create an automation with this prompt:

```text
Call $solve-issues for each Git repo under the current workspace.
```

Pick your own trigger time, trigger interval, and CWD. The CWD should be a folder that contains the repositories you want the automation to process.

## Why

So we can go touch some grass.
