# human-out-of-loop

Codex skills that turn GitHub issues into reviewed draft PRs while you do something else.

## Requirements

- The Codex desktop app with task, worktree, and task-coordination features.
- GitHub access to read issues and PRs. Publishing to your repositories also requires authenticated write access.
- The [Ponytail](https://github.com/DietrichGebert/ponytail) skill, which keeps implementation and review workers focused on the smallest correct solution.

GitHub Yeet is not required; these skills use the Git and GitHub tools already available to Codex.

## Install

Paste this repository URL into Codex and ask:

> Install the `solve-issues` and `pr` skills from https://github.com/Hsiii/human-out-of-loop.

Do the same for Ponytail if it is not installed:

> Install the `ponytail` skill from https://github.com/DietrichGebert/ponytail.

The skills become available on your next turn.

## Use cases

| What you want | What to say |
| --- | --- |
| Pick and solve one eligible issue | `$solve-issues` |
| Solve issue #65 | `$solve-issues 65` |
| Solve five random eligible issues sequentially | `$solve-issues random 5` |
| Solve five at once | `Run $solve-issues random 5 in parallel` |
| Continue an interrupted run | `Continue` in the same task |
| Draft a polished PR for the current branch | `$pr` |

An eligible issue is open and is not already linked to an open or draft PR that solves it. There is no batch maximum, but parallel runs create more tasks and use more tokens.

## What Codex will change

For repositories you own, `$solve-issues` may create worktrees, branches, commits, draft artifacts, media, and a real draft PR. It runs local checks, reviews the full PR diff, waits for configured GitHub checks, and posts at most one transparent review comment per head beginning `Codex reviewed this PR at …`. Repositories without CI checks skip that wait.

For external repositories, it may create local worktrees, branches, commits, `DRAFT.md`, `REVIEW.md`, and comparison media. It never pushes, opens a PR, or posts any GitHub activity.

The skills never mark a PR ready, request reviewers, merge, enable auto-merge, or impersonate you. Finished worker tasks stay available and their worktrees remain intact for follow-up changes.

## How it works

### Orchestrator

Running `$solve-issues` turns the current task into an orchestrator. It selects issues, gives each one an isolated branch and worktree, and tracks dedicated developer and reviewer tasks. The orchestrator does not edit code.

### Developer

The developer receives the issue and workspace contract, activates Ponytail, implements and commits the smallest correct fix, runs checks, captures comparison media when useful, and calls `$pr`.

### Reviewer

The reviewer shares the worktree but has separate conversation context. It reviews the entire base-to-head PR diff without editing code. Findings return to the developer until the current head passes.

## Automation

Ask Codex to create an automation that calls `$solve-issues random [amount]` for each Git repository under a workspace, then choose the schedule and folder.

I use [Hsiii/fish-alias](https://github.com/Hsiii/fish-alias) to collect suitable ongoing projects into one directory with symlinks.

## License

[MIT](LICENSE)

## Why

So we can go touch some grass.
