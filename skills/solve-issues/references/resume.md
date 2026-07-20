# Continuation and Recovery

Use this procedure when the user naturally asks an existing run to continue, when a status request reveals unfinished work, on a run-level heartbeat wake, or after an interrupted wait. Continuation is behavior of the already-active skill, not a `$solve-issues` parameter.

1. Continue in the orchestration task that already owns the run. If the current task is a worker or another task, use `list_threads` to find the matching active `ISS <selector> <status>` task in the same project and send the continuation request there. If no parent survives, adopt the run in the current task; if multiple runs match, ask the user which one.
2. Recover the original selector, requested amount when applicable, selected issue numbers, worker IDs, worktrees, branches, internal or external mode, and last `wait_threads` cursors from the parent history. Cross-check with `list_threads`, Git worktrees and branches, `DRAFT.md`/`REVIEW.md`, and GitHub issues and PRs. Match by repo, issue, and branch—not title alone.
3. Never create a replacement task, branch, worktree, review request, push, or PR when an equivalent already exists. Treat the current branch head SHA as the identity of a review result, while keeping the review scope as the whole base-to-head PR diff; an older pass marker is stale. Before posting to a user-owned published PR, check whether a `Codex reviewed this PR at <shortHeadSha>.` review already exists for that head and do not duplicate it. Never post review activity to an external repo.
4. Reconcile each issue by its furthest durable stage:
   - `setup`: finish the seed, then create only missing sibling workers.
   - `build`, `check`, or `fix`: continue or wait for the existing implementation task.
   - `draft` or `review`: restore the existing review wait; resend only when no request exists for the current head SHA.
   - `pr` or `ci`: inspect the existing draft PR and checks; never open another PR.
   - `done`: verify the final head, review pass, and internal-mode CI result. Stop the heartbeat but leave tasks pinned and unarchived and preserve the worktree and branch. On follow-up changes, wake the existing workers, resume at `fix`, and repeat checks, draft update, review, and the mode's PR/CI behavior for the new head. Never create a replacement issue group.
   - `block`: preserve every artifact and report the exact decision or authority needed.
5. Restore one run-level heartbeat on the orchestration task if unattended work remains, updating the existing automation instead of creating a duplicate. Each wake performs one reconciliation and one bounded `wait_threads` snapshot over active workers, then returns control. Delete the heartbeat when all requested issues are terminal, without archiving tasks or removing worktrees.
6. Restore compact task titles from observed state rather than trusting stale title text. Continue sequential or parallel scheduling from the original request; never change the exact target or increase a random batch's requested amount during continuation.
