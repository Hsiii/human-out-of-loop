# human-out-of-loop

Codex skills to turn issues into reviewed PRs automatically.

## Prerequisite

These skills expect the `ponytail` skill to be installed for implementation and review threads. If you do not have it, install it from [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail).

## Skills

- `solve-issues [amount]`: processes the requested number of eligible issues per repo, defaulting to `1` with no hard maximum. Compact task titles show each stage, and `$solve-issues resume` or the run heartbeat can recover existing workers without duplicating work.
- `solve-issue [amount]`: convenience alias for `$solve-issues [amount]`, so `$solve-issue 5` works too.
- `pr`: triggered by "pr"; internal mode uses the repo template, Yeet, and browser media upload to create a real draft PR, while external mode writes gitignored `DRAFT.md` and `.codex-pr-media/` without opening a GitHub PR.

## Automation

Ask Codex to create an automation with this prompt:

```text
Call $solve-issues for each Git repo under the current workspace.
```

Pick your own trigger time, trigger interval, and CWD. The CWD should be a folder that contains the repositories you want the automation to process.

## Why

So we can go touch some grass.
