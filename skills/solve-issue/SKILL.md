---
name: solve-issue
description: "Run one GitHub issue loop from an explicit task packet containing issue, repo, mode, implementation ponytail thread, review ponytail thread, artifact paths, pass marker, and stop protocol. Implement, verify, call $pr, then heartbeat until review passes. Use when $solve-issues hands off one open issue labeled ready or when the user provides one complete task packet."
---

# Solve Issue

Input: one task packet from `$solve-issues`. Do not fetch queues, choose issues, change thread topology, or infer missing orchestration context.

The implementation and review threads must run in the dedicated worktree supplied by the task packet. The implementation thread owns all code writes. The review thread must not change code; in external mode it may write only the supplied `REVIEW.md` path.

Implementation thread:

1. Read the issue, comments, linked PRs, and relevant code.
2. For user-facing changes, capture "before" media in the supplied media directory before editing files whenever the current behavior can be shown.
3. Implement, commit, verify with repo checks, and capture "after" media for user-facing changes.
4. Use `$pr` with the supplied mode and artifact paths.
5. Require `$pr` to return `{ mode, prUrl | draftPath, branch, headSha, media }`. If this result is incomplete, fix `$pr` output before review starts.
6. Send the review thread a concrete review prompt with the repo, issue URL, mode, worktree path, branch, head SHA, PR URL or `DRAFT.md` path, media paths, `REVIEW.md` path, pass marker, and review permissions.
7. Create or use heartbeat automation, or explicitly poll with `read_thread`, until the review thread returns the exact pass marker from the task packet.
8. For each actionable review comment, update the implementation, commit fixes, update the PR or `DRAFT.md`, send the review thread a new review prompt with the new head SHA and pass marker, and continue.
9. On the exact pass marker, send the stop/archive message from the task packet and report completion to the orchestrator.

Review thread:

1. Internal: review the draft GitHub PR and comment on it. Do not edit files.
2. External: review `DRAFT.md` and write actionable findings to the supplied gitignored `REVIEW.md` path. Do not edit any other file.
3. If changes are required, write concise actionable comments with file/line references where possible.
4. If no changes are required, return only the exact pass marker supplied by the implementation thread, e.g. `REVIEW_PASS issue=<number> head=<headSha>`.
5. On pass, send the implementation thread the stop message from the task packet.
