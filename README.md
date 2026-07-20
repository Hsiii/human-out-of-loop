# human-out-of-loop

Codex skills to turn issues into reviewed PRs automatically.

## Prerequisite

These skills expect the `ponytail` skill to be installed for implementation and review threads. If you do not have it, install it from [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail).

## Skills

- `$solve-issues [amount]`
  - Defaults to `1`; no maximum.
  - Internal repos: reviewed draft PR, green CI, Codex-labeled review.
  - External repos: reviewed local `DRAFT.md`; no GitHub writes.
  - Sequential by default; say `parallel` to run every selected issue concurrently.
  - `$solve-issues resume` and the run heartbeat recover existing workers; finished workers stay available for follow-up changes.
- `$pr`
  - Internal repos: publish a real draft PR after checks and review.
  - External repos: prepare a gitignored local `DRAFT.md` only.
  - Never mark ready, request reviewers, merge, or enable auto-merge.

## Automation

Ask Codex to create an automation that calls `$solve-issues` for each Git repo under the current workspace, then choose the schedule and workspace folder.

## Why

So we can go touch some grass.
