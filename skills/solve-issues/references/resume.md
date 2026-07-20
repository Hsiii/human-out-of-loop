# Resume

Use this procedure for `$solve-issues resume`, a run-level heartbeat wake, or recovery after an interrupted wait.

1. If the current task is not the orchestration task, use `list_threads` to find the matching active `ISS <amount> <status>` task in the same project. Send the resume request there. If no parent survives, adopt the run in the current task; if multiple runs match, ask the user which one.
2. Recover the requested amount, selected issue numbers, worker IDs, worktrees, branches, publish permission, and last `wait_threads` cursors from the parent history. Cross-check with `list_threads`, Git worktrees and branches, `DRAFT.md`/`REVIEW.md`, and GitHub issues and PRs. Match by repo, issue, and branch—not title alone.
3. Never create a replacement task, branch, worktree, review request, push, or PR when an equivalent already exists. Treat the current branch head SHA as the identity of a review result; an older pass marker is stale. Before posting to a user-owned published PR, check whether a `Codex reviewed this PR at <shortHeadSha>.` review already exists for that head and do not duplicate it. Never post review activity to an external repo.
4. Reconcile each issue by its furthest durable stage:
   - `setup`: finish the seed, then create only missing sibling workers.
   - `build`, `check`, or `fix`: continue or wait for the existing implementation task.
   - `draft` or `review`: restore the existing review wait; resend only when no request exists for the current head SHA.
   - `pr` or `ci`: inspect the existing draft PR and checks; never open another PR.
   - `done`: verify the final head, review pass, and any required CI result. Stop the heartbeat but leave tasks pinned and unarchived and preserve the worktree and branch. On follow-up changes, wake the existing workers, resume at `fix`, and repeat checks, draft update, review, and any authorized publish/CI gates for the new head. Never create a replacement issue group.
   - `block`: preserve every artifact and report the exact decision or authority needed.
5. Restore one run-level heartbeat on the orchestration task if unattended work remains, updating the existing automation instead of creating a duplicate. Each wake performs one reconciliation and one bounded `wait_threads` snapshot over active workers, then returns control. Delete the heartbeat when all requested issues are terminal, without archiving tasks or removing worktrees.
6. Restore compact task titles from observed state rather than trusting stale title text. Continue sequential or parallel scheduling from the original request; never increase the requested amount during resume.
