# human-out-of-loop

Codex skills to turn issues into reviewed PRs automatically.

## Prerequisite

The [Ponytail](https://github.com/DietrichGebert/ponytail) skill to prevent 5.6 Sol from going crazy.

## Skills

- `$solve-issues [amount]`
  - Defaults to solving one issue, can be set to any amount.
  - Sequential by default; say `parallel` to run all at once when you have enough tokens to burn
  - Calls `$pr` and review it to decide whether to continue working on it
- `$pr`
  - For your own repos, publish a draft PR with best practices like include screenshots for UI changes.
  - For external repos, prepare a local `DRAFT.md`

## Automation

To make it even more automized, you can ask Codex to create an automation that calls `$solve-issues[amount]` for each Git repo under the current workspace, then choose the schedule and workspace folder.

In my case, I maintain symlinks with [Hsiii/fish-alias](https://github.com/Hsiii/fish-alias) to collect ongoing projects into a directory for this.

## Why

So we can go touch some grass.
