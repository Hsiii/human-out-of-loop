# human-out-of-loop

Codex skills that turn GitHub issues and committed changes into reviewed draft PRs, so you can go touch some grass.

## Install

Requires the Codex desktop app, GitHub access, and [Ponytail](https://github.com/DietrichGebert/ponytail).

Ask Codex:

> Install the skills from `Hsiii/human-out-of-loop`.

## Use

| Goal | Prompt |
| --- | --- |
| Create a draft PR | `$pr` |
| Solve issue #65 | `$solve-issues 65` |
| Solve five issues in parallel | `$solve-issues pick 5` |
| Solve five issues sequentially | `$solve-issues pick 5, one by one` |

## What to expect

`$pr` reviews committed changes and prepares a draft PR. For external repositories, it creates a local `DRAFT.md` instead.

`$solve-issues` implements and reviews issues in isolated branches and worktrees, then runs `$pr` for each solution.

The skills may create branches, worktrees, commits, and comparison media. They never merge or mark a PR ready.

## Automation

Ask Codex to run `$solve-issues pick [amount]` on a schedule across the Git repositories in a workspace.
