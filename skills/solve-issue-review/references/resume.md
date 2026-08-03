Do not parse continuation as a `$solve-issue-review` parameter.

## Recover

- Continue in the owning orchestrator. Otherwise find the matching `DEV` and `REV` tasks with `list_threads` and adopt orchestration here. If `DEV` is missing but its worktree survives, create a replacement there; create a missing `REV` only for a ready head. If several groups match, ask.
- Restore selector/count and each per-issue record: task IDs, worktree, accepted head, PR URL, lifecycle state, and both cursors. Derive branch and base from Git, checks and media from DEV history, and publication metadata from GitHub. Match runs by repo + issue + branch, never title.
- Reuse existing tasks, branches, worktrees, review requests, pushes, and PRs; never duplicate them. A pass belongs to one head SHA; older passes are stale and exact replayed results are ignored. Keep review communication inside Codex tasks and never post GitHub review activity.
- Recover lifecycle state from structured worker results and artifacts, not intermediate technical commentary. Never restate or independently interpret worker diagnosis, design, implementation, or validation.

## Stage

| Stage | Action |
| --- | --- |
| `build`, `check`, `fix` | Wait for DEV. Do not probe it or report its intermediate technical progress; its replacement result goes only to `REV`. |
| `review` | Wait for `REV`, which owns the DEV feedback loop. Report only lifecycle transitions. |
| `pr` | Wait for `REV`; never finalize in `ORCH` or mirror publication work. |
| `done` | Verify the reviewed head matches the draft PR head. Stop heartbeat and preserve both workers, branch, and worktree. |
| `block` | Preserve everything; report the exact decision or authority needed. |

## Heartbeat

- When unattended work requires wakeups, keep exactly one heartbeat on the orchestrator; each wake reconciles once, takes one bounded snapshot of all active workers, then yields. Delete it when all issues are terminal.
- A timeout, commentary update, or tool marker is not a lifecycle transition. Preserve each cursor and wait again unless a worker explicitly requests help, completes with a malformed result, fails a readiness invariant, or receives a user-requested scope change.
- Preserve sequential/parallel mode and selector; never increase a picked count. Keep the static naming convention and never encode lifecycle stage in titles.
