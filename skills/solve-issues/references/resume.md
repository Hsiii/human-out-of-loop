# Continue a Run

Use for “continue,” unfinished status, heartbeat, or interrupted-wait recovery. This is behavior, not a `$solve-issues` parameter.

## Recover

- Continue in the owning orchestrator. Else find its matching active `ISS <selector> <status>` task in this project with `list_threads` and message it. If none survives, adopt here; if several match, ask.
- Restore selector/count, issues, task IDs, worktrees, branches, mode, and wait cursors from history. Verify with tasks, Git, artifacts, and GitHub; match by repo + issue + branch, never title.
- Reuse existing tasks, branches, worktrees, review requests, pushes, and PRs; never duplicate them. A pass belongs to one head SHA but reviews the full base-to-head diff; older passes are stale. Before an internal PR comment, check for `Codex reviewed this PR at <shortHeadSha>.` Never write to an external repo.

## Continue from the Furthest Stage

| Stage | Action |
| --- | --- |
| `setup` | Finish setup; add only missing sibling workers. |
| `build`, `check`, `fix` | Continue or wait for the existing developer. |
| `draft`, `review` | Restore review wait; resend only when the current head has no request. |
| `pr`, `ci` | Inspect the existing draft PR and checks; never open another PR. |
| `done` | Verify final-head review and internal CI; stop heartbeat. Keep tasks pinned/unarchived and preserve branch/worktree. For follow-ups, wake the same workers at `fix` and repeat checks, draft, full review, and PR/CI. |
| `block` | Preserve everything; report the exact decision or authority needed. |

## Keep It Alive

- Keep exactly one heartbeat on the orchestrator while unattended work remains; update it, never add another. Each wake reconciles once, takes one bounded `wait_threads` snapshot of active workers, then yields. Delete it when all issues are terminal.
- Restore titles from actual state. Preserve sequential/parallel mode and selector; never increase a random count. Never archive tasks or remove worktrees automatically.
