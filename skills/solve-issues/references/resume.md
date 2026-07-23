# Continue a Run

Use for “continue,” unfinished status, an unattended heartbeat, or interrupted-wait recovery. This is behavior, not a `$solve-issues` parameter.

## Recover

- Continue in the owning orchestrator. Otherwise find the matching developer and reviewer/publisher tasks with `list_threads` and adopt orchestration here. If a developer is missing but its worktree survives, create a replacement there; create a missing reviewer/publisher only for a ready head. If several groups match, ask.
- Restore selector/count and each per-issue record: task IDs, worktree, accepted head, PR URL or draft path, lifecycle state, and both cursors. Derive branch and base from Git, checks and media from DEV history, and publication mode and metadata from `$pr` artifacts or GitHub. Match runs by repo + issue + branch, never title.
- Reuse existing tasks, branches, worktrees, review requests, pushes, and PRs; never duplicate them. A pass belongs to one head SHA; older passes are stale and exact replayed results are ignored. Keep review communication inside Codex tasks and never post GitHub review activity.

## Continue from the Furthest Stage

| Stage | Action |
| --- | --- |
| `build`, `check`, `fix` | Continue or wait for DEV; its replacement result goes only to `REV/PR`. |
| `review` | Continue or wait for `REV/PR`, which owns the DEV feedback loop. |
| `pr`, `ci` | Continue or wait for the existing reviewer/publisher; never finalize in the orchestrator. |
| `done` | Verify the reviewed head matches the internal draft PR head, or verify the external `DRAFT.md`, `.codex-pr-media/title`, and matching `.codex-pr-media/reviewed-head` sidecar all exist; for internal mode, verify CI was inspected. Stop heartbeat and preserve both workers, branch, and worktree. |
| `block` | Preserve everything; report the exact decision or authority needed. |

## Keep It Alive

- When unattended work requires wakeups, keep exactly one heartbeat on the orchestrator; each wake reconciles once, takes one bounded snapshot of all active workers, then yields. Delete it when all issues are terminal.
- Preserve sequential/parallel mode and selector; never increase a picked count. Keep the static naming convention and never encode lifecycle stage in titles.
