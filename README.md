# human-out-of-loop

Codex skills that turn GitHub issues into reviewed draft PRs automatically so you can go touch some grass.

## Requirements

- The [Ponytail](https://github.com/DietrichGebert/ponytail) skill to keep changes minimal (so Sol won't go crazy).
- The Codex desktop app for thread management.
- GitHub access to read issues and publish PRs.

## Install

Ask Codex to install the skills from `Hsiii/human-out-of-loop`.

## Skills

- `$pr`
  - For your own repos, opens a polished draft PR.
  - For external repos, drafts it in a local `DRAFT.md`.
  - Fetches the current remote base and refuses branches already used by a closed or merged PR.
- `$solve-issues`
  - Picks eligible open issues without an existing solving PR, implements minimal solutions, reviews them independently, and creates draft PRs.

## Examples

| What you want | What to say |
| --- | --- |
| Draft a polished PR | `$pr` |
| Solve issue #65 | `$solve-issues 65` |
| Solve five issues in parallel | `$solve-issues pick 5` |
| Solve five one by one so token limits do not interrupt the whole batch | `$solve-issues pick 5, one by one` |

## How it works

Running `$solve-issues` turns the current thread into an orchestrator. It gives each issue an isolated branch and worktree, a Ponytail developer thread, and an independent reviewer/publisher thread created when code is ready.

The dev thread receives the issue, implements and commits, runs checks, and captures before and after media for every UI change—videos whenever interaction, motion, or multiple steps are involved, and images otherwise.

The first dev result creates the reviewer/publisher thread. After that, the reviewer owns the direct feedback loop with dev, runs `$pr` for each head, and reports lifecycle state to the orchestrator. It publishes an internal draft PR or writes an external `DRAFT.md` after review passes.

## What Codex modifies

For repositories you own, `$solve-issues` may create worktrees, branches, commits, media, and a real draft PR. It runs local checks, reviews the PR, and inspects configured GitHub checks once. It does not create `DRAFT.md`.

For external repositories, it may create local worktrees, branches, commits, `DRAFT.md` (PR body draft), and comparison media. It never pushes, opens a PR, or posts any GitHub activity.

The skills never post GitHub review activity, mark a PR ready, request reviewers, merge, or enable auto-merge. Review feedback stays inside the worker tasks. Finished workers and worktrees remain available for follow-up changes.

## Automation

You can ask Codex to create an automation that calls `$solve-issues pick [amount]` for each Git repository under a workspace, then choose the schedule and folder for further automation.

I use [Hsiii/fish-alias](https://github.com/Hsiii/fish-alias) to collect suitable ongoing projects into one directory with symlinks.
