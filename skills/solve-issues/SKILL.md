---
name: solve-issues
description: "Orchestrate GitHub issue processing for up to five open issues labeled ready per repo: auto-determine internal vs external repo mode, isolate same-repo issue groups in separate worktrees, create two ponytail threads per issue, and call $solve-issue with an explicit task packet. Use when the user says \"solve issues\", \"building loops\", asks to process ready issues, or wants issue-to-PR automation."
---

# Solve Issues

Use Codex app background threads for durable work. Do not use transient subagents for the implementation/review loop.

For up to five open issues labeled `ready` per repo:

1. Determine mode once per repo: internal for the user's own repos; external otherwise.
2. Allocate one dedicated Codex worktree per issue thread group, even when no other issue is active for the repo. The implementation thread and review thread for that issue share this worktree; no other issue may use it.
3. Create two ponytail Codex app threads in that worktree:
   - implementation thread: owns all code edits, commits, verification, branch updates, PR/draft updates, and review-response changes.
   - review thread: reads the PR/draft and writes only review comments. In external mode it may write only `REVIEW.md`; otherwise it must not modify repo files.
4. Send `$solve-issue` to the implementation thread with an explicit task packet:
   - repo, issue number, issue URL, mode
   - implementation thread ID, review thread ID
   - branch name, base branch, worktree path
   - artifact paths: `DRAFT.md`, `REVIEW.md`, `.codex-pr-media/`
   - PR result contract: `{ mode, prUrl | draftPath, branch, headSha, media }`
   - review pass marker: `REVIEW_PASS issue=<number> head=<headSha>`
   - review start protocol: implementation thread sends the PR result and review instructions to the review thread
   - monitor protocol: implementation thread uses heartbeat automation or explicit `read_thread` checks until the review thread returns the pass marker
   - stop protocol: after pass, send a final stop/archive message to the implementation thread and archive finished background threads when appropriate
5. Monitor the issue thread group with `read_thread` until the implementation loop reports completion. Continue to the next issue only after that group stops.

Ask only if ownership is ambiguous. Do not implement here; only schedule isolated per-issue loops.
