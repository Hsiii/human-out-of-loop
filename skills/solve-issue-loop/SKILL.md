---
name: solve-issue-loop
description: "Run one GitHub issue loop from an explicit task packet containing issue, repo, mode, implementation ponytail thread, review ponytail thread, artifact paths, pass marker, and stop protocol. Implement, verify, call $pr, then heartbeat until review passes. Use when $solve-issue hands off one open issue labeled ready or when the user provides one complete task packet."
---

# Solve Issue Loop

Input: one task packet from `$solve-issue`. Do not fetch queues or infer missing context.

Implementation thread:

1. Read the issue, comments, linked PRs, and relevant code.
2. Implement, verify with repo checks, and capture before/after media for user-facing changes.
3. Use `$pr` with the supplied mode.
4. Start the review thread only after `$pr` returns a draft PR URL or writes `DRAFT.md`.
5. Enter heartbeat mode and resolve review comments until the review thread returns the pass marker.

Review thread:

1. Internal: review the draft GitHub PR and comment on it.
2. External: review `DRAFT.md` and write gitignored `REVIEW.md`.
3. If changes are needed, write actionable comments.
4. If review passes, write `PASS` in the PR comment or `REVIEW.md`, then send the implementation thread the stop message from the task packet.
