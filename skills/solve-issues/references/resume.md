# Continue a Run

Use for “continue,” unfinished status, an unattended heartbeat, or interrupted-wait recovery. This is behavior, not a `$solve-issues` parameter.

## Recover

- Continue in the owning orchestrator. Otherwise find the matching developer and reviewer tasks with `list_threads` and adopt orchestration here. If a developer is missing but its worktree survives, create a replacement there; create a missing reviewer only for a ready head. If several groups match, ask.
- Restore selector/count and every per-issue record: task IDs, worktree, branch, base, ready and reviewed heads, checks, media, PR URL, mode, lifecycle state, and both cursors. Verify with tasks, Git, artifacts, and GitHub; match by repo + issue + branch, never title.
- Reuse existing tasks, branches, worktrees, review requests, pushes, and PRs; never duplicate them. A pass belongs to one head SHA; older passes are stale. Keep review communication inside Codex tasks and never post GitHub review activity.

## Continue from the Furthest Stage

| Stage | Action |
| --- | --- |
| `build`, `check`, `fix` | Continue or wait for the existing developer. |
| `review` | Wait for the existing reviewer. Send only a ready head it has not seen; reuse its prior head for a delta pass. |
| `pr`, `ci` | Finalize only after the current-head pass. Reuse an existing draft PR, then inspect checks once. |
| `done` | Verify the reviewed head matches the internal draft PR or external `DRAFT.md`; for internal mode, verify CI was inspected. Stop heartbeat and preserve both workers, branch, and worktree. |
| `block` | Preserve everything; report the exact decision or authority needed. |

## Keep It Alive

- Ordinary foreground waits use `wait_threads` and no heartbeat. Pending CI is terminal unless the user asked to babysit it. When other unattended work requires wakeups, keep exactly one heartbeat on the orchestrator; each wake reconciles once, takes one bounded snapshot of all active workers, then yields. Delete it when all issues are terminal.
- Preserve sequential/parallel mode and selector; never increase a picked count. Restore a missing static title when useful, but never encode lifecycle stage in it. Never archive tasks or remove worktrees automatically.
