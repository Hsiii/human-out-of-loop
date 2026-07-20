# human-out-of-loop

Codex skills that turn GitHub issues into reviewed draft PRs automaticaly so you can go touch some grass.

## Requirements

- The [Ponytail](https://github.com/DietrichGebert/ponytail) skill to keep changes minimal (so Sol won't go crazy).
- The Codex desktop app for thread management.
- GitHub access to read issues and publish PRs. 

## Install

Ask Codex to install the skills from `Hsiii/human-out-of-loop`.

## Use cases

There're two skills here, `$solve-issues` and `$pr`, you can use them like this:

| What you want | What to say |
| --- | --- |
|  |
| Solve issue #65 | `$solve-issues 65` |
| Solve five random eligible issues sequentially | `$solve-issues random 5` |
| Solve five at once | `$solve-issues random 5 parallel` | |
| Draft a polished PR for the current branch | `$pr` |

## How it works

Running `$solve-issues` turns the current thread into an orchestrator. It gives each issue an isolated branch, worktree, a developer thread and a reviewer thread.

The dev thread receives the issue, implements and commits, runs checks, captures comparison media when useful, and calls `$pr`.

The reviewer thread review the PR and return its findings to the dev thread until the PR passes.

## What Codex modifies

For repositories you own, `$solve-issues` may create worktrees, branches, commits, draft artifacts, media, and a real draft PR. It runs local checks, reviews the full PR diff, waits for configured GitHub checks, and posts review comment beginning with `Codex reviewed this PR and...`.

For external repositories, it may create local worktrees, branches, commits, `DRAFT.md` (PR body draft), `REVIEW.md` (as local PR comments), and comparison media. It never pushes, opens a PR, or posts any GitHub activity.

The skills never mark a PR ready, request reviewers, merge, or enable auto-merge. Finished worker tasks stay available and their worktrees remain intact in case you want follow-up changes.

## Automation

You can ask Codex to create an automation that calls `$solve-issues random [amount]` for each Git repository under a workspace, then choose the schedule and folder for further automation.

I use [Hsiii/fish-alias](https://github.com/Hsiii/fish-alias) to collect suitable ongoing projects into one directory with symlinks.