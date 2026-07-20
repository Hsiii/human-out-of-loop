# human-out-of-loop

Codex skills to turn issues into reviewed PRs automatically.

## Prerequisite

The [Ponytail](https://github.com/DietrichGebert/ponytail) skill to prevent 5.6 Sol from going crazy.

## Skills

- `$solve-issues`
  - No argument picks one eligible issue at random.
  - `$solve-issues 5` targets issue #5 if it is open and has no open or draft PR solving it.
  - `$solve-issues random 5` picks five eligible issues; there is no maximum.
  - Runs sequentially by default; say `parallel` to run all at once when you have enough tokens to burn.
  - Natural follow-ups like “continue” recover the existing run; there is no resume parameter.
- `$pr`
  - For your own repos, publishes a reviewed draft PR with green CI and evidence such as UI screenshots.
  - For external repos, prepares a reviewed local `DRAFT.md` without writing to GitHub.

## How it actually works

- The task you start is the orchestrator: it selects issues, creates workers, tracks progress, and does not edit code itself.
- Each issue gets its own branch and worktree plus a neutral setup task. The developer and reviewer are sibling forks of that neutral task, never forks of each other.
- Those siblings share the same files but keep separate conversation histories, so the reviewer sees the result without inheriting the developer's reasoning or role.
- The orchestrator sends the developer a concrete context packet: repo, issue, mode, branch, worktree, task IDs, artifact paths, and exact completion markers.
- The developer owns edits, commits, checks, and PR updates. The reviewer is read-only and reviews a specific commit SHA; findings loop back to the developer until that exact head passes.
- One bounded wait watches all active workers, while a single heartbeat can reconcile interrupted work. Saying “continue” uses the same recovery behavior.
- Finished tasks stay pinned and unarchived with their worktrees intact, so follow-up changes reuse the same developer/reviewer pair instead of starting over.

## Automation

To make it even more automated, ask Codex to create an automation that calls `$solve-issues` or `$solve-issues random [amount]` for each Git repo under the current workspace, then choose the schedule and workspace folder.

In my case, I maintain symlinks with [Hsiii/fish-alias](https://github.com/Hsiii/fish-alias) to collect ongoing projects into a directory for this.

## Why

So we can go touch some grass.
