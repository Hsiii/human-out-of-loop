---
name: solve-issues
description: "Orchestrate GitHub issue processing for up to five open issues per repo: auto-determine internal vs external repo mode, isolate same-repo issue groups in separate worktrees, create two visible ponytail Codex app threads per issue, and call $solve-issue with an explicit task packet. Use when the user says \"solve issues\", \"building loops\", asks to process open issues, or wants issue-to-PR automation."
---

# Solve Issues

Use user-owned, sidebar-visible Codex app threads for durable work. Create them with `create_thread`; do not use transient subagents, hidden worker agents, or multi-agent tools for the implementation/review loop.

Do not busy-wait on visible threads. Use heartbeat automation or scheduled wakeups for monitoring; direct `read_thread` checks are a fallback and must be limited to one check per thread group per wakeup. Start with a 10 minute minimum cadence, back off to 30 minutes after two unchanged checks, and return control between checks.

For up to five open issues per repo:

1. Determine mode once per repo: internal for the user's own repos; external otherwise.
2. Allocate one dedicated Codex worktree per issue thread group, even when no other issue is active for the repo. The implementation thread and review thread for that issue share this worktree; no other issue may use it.
3. Create two user-owned, visible ponytail Codex app threads in that worktree. Title and pin both threads while active when the Codex app tools are available:
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
   - monitor protocol: implementation thread uses heartbeat automation; fallback `read_thread` checks are one-per-wakeup, at least 10 minutes apart, and back off to 30 minutes after two unchanged checks
   - stop protocol: after pass, send a final stop/archive message to the implementation thread and unpin/archive finished visible threads when appropriate
5. Monitor the issue thread group through the same heartbeat cadence. On each wakeup, read the implementation thread once, then either handle completion or schedule the next check. Continue to the next issue only after that group stops.

Ask only if ownership is ambiguous. Do not implement here; only schedule isolated per-issue loops.
