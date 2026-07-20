# human-out-of-loop

Codex skills to turn issues into reviewed PRs automatically.

## Prerequisite

The [Ponytail](https://github.com/DietrichGebert/ponytail) skill to prevent 5.6 Sol from going crazy.

## Skills

- `$solve-issues`
  - `$solve-issues 5` solves issue #5 if it is open and has no open or draft PR solving it.
  - `$solve-issues random 5` solves five eligible issues sequentially by default; say `parallel` to run all at once when you have enough tokens to burn.
  - Calls `$pr` and review it.
- `$pr`
  - For your own repos, open a draft PR with green CI and evidence such as UI screenshots.
  - For external repos, prepares a local `DRAFT.md`.

## How it works

### The Orchestration layer
Running `$solve-issues` turns the thread into an orchestration layer. It will start dispatching issues and lets them get their own branch, worktree, a dev thread, and a reviewer thread, then track their progress and orchestrate them.

### The dev thread
Like how you'd use (I guess) codex to solve issues normally, this thread will receive context from the orchestrator and be asked to solve the issue, then calls `$pr` when it's done.

### The reviewer thread
The reviewer thread will review the PR drafted, then ask the developer to update the PR if needed in a loop.

## Automation

To make it even more automated, ask Codex to create an automation that calls `$solve-issues random [amount]` for each Git repo under the current workspace, then choose the schedule and workspace folder.

In my case, I maintain symlinks with [Hsiii/fish-alias](https://github.com/Hsiii/fish-alias) to collect suitable ongoing projects into a directory for this.

## Why

So we can go touch some grass.
