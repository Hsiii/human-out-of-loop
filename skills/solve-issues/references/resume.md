# Continue a Run

Use for “continue,” unfinished status, an unattended heartbeat, or interrupted-wait recovery. This is behavior, not a `$solve-issues` parameter.

## Recover

- Continue in the owning orchestrator. Otherwise find the matching developer and reviewer/publisher tasks with `list_threads` and adopt orchestration here. If a developer is missing but its worktree survives, create a replacement there; create a missing reviewer/publisher only for a ready head. If several groups match, ask.
- Restore selector/count and every per-issue record: task IDs, worktree, branch, base, ready and reviewed heads, checks, media, PR title, PR URL or draft path, mode, lifecycle state, buffered reviewer results, and both cursors. Rebuild missing buffers from task history and artifacts, keyed by task ID + result kind + head SHA, before advancing any cursor. Verify with tasks, Git, artifacts, and GitHub; match by repo + issue + branch, never title. When a cursor is unrecoverable, resume that task from an empty cursor and reconcile replayed results by the same key.
- Reuse existing tasks, branches, worktrees, review requests, pushes, and PRs; never duplicate them. A pass belongs to one head SHA; older passes are stale and exact replayed results are ignored. Keep review communication inside Codex tasks and never post GitHub review activity.

## Continue from the Furthest Stage

| Stage | Action |
| --- | --- |
| `build`, `check`, `fix` | Continue or wait for the existing developer. |
| `review` | Wait for the existing reviewer/publisher. Send only a ready head it has not seen; reuse its prior head for a delta pass. |
| `pr`, `ci` | Continue or wait for the existing reviewer/publisher; never finalize in the orchestrator. |
| `done` | Verify the reviewed head matches the internal draft PR head, or verify the external `DRAFT.md`, `.codex-pr-media/title`, and matching `.codex-pr-media/reviewed-head` sidecar all exist; for internal mode, verify CI was inspected. Stop heartbeat and preserve both workers, branch, and worktree. |
| `block` | Preserve everything; report the exact decision or authority needed. |

## Keep It Alive

- When unattended work requires wakeups, keep exactly one heartbeat on the orchestrator; each wake reconciles once, takes one bounded snapshot of all active workers, then yields. Delete it when all issues are terminal.
- Preserve sequential/parallel mode and selector; never increase a picked count. Restore a missing static title when useful, but never encode lifecycle stage in it.
