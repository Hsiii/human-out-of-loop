---
name: solve-issues
description: "Orchestrate GitHub issue processing for up to five open issues per repo: skip issues already covered by open or draft PRs, use conventional branch names, isolate issue groups in separate worktrees, create a neutral seed plus two visible ponytail Codex app threads per issue, and call $solve-issue with an explicit task packet. Supports sequential scheduling by default and immediate five-issue fan-out when the user asks to work in parallel, concurrently, simultaneously, at once, or with equivalent intent. Use when the user says \"solve issues\", \"building loops\", asks to process open issues, or wants issue-to-PR automation."
---

# Solve Issues

Use user-owned, sidebar-visible Codex app threads for durable work. Create the neutral seed with `create_thread` and the two worker threads as same-directory sibling forks from that seed; do not use transient subagents, hidden worker agents, or multi-agent tools for the implementation/review loop.

Thread topology is correctness-critical. Never fork the review thread from the implementation thread, and never fork the implementation thread from the review thread. Forks copy completed conversation history, so a worker forked from another worker inherits the wrong role. To share one dedicated Codex worktree without role contamination, create a neutral worktree seed thread first, then fork the implementation and review threads as same-directory siblings from that neutral seed. Archive or unpin the seed after both worker thread IDs are captured.

Do not busy-wait on visible threads. Use heartbeat automation or scheduled wakeups for monitoring; direct `read_thread` checks are a fallback and must be limited to one check per thread group per wakeup. Start with a 10 minute minimum cadence, back off to 30 minutes after two unchanged checks, and return control between checks.

Choose the scheduling mode from the user's request:

- Use sequential mode by default: start one issue group and wait for it to stop before starting the next.
- Use parallel mode whenever the user expresses parallel or concurrent intent anywhere in the request. Match meaning, not only exact words; examples include `parallel`, `parallely`, `parallelize`, `concurrently`, `simultaneously`, `at once`, `all five now`, `fan out`, or equivalent phrasing.
- Treat parallel intent as authorization to start every selected issue group immediately and as a signal that the user accepts the token usage. Do not ask for quota confirmation. Start all five `$solve-issue` loops at once when five eligible issues exist; otherwise start all available loops, still capped at five per repo.
- Parallel mode changes scheduling only. Keep every issue in its own dedicated worktree and preserve the neutral-seed and sibling-worker topology below.

For up to five open issues per repo:

1. Determine mode once per repo: internal for the user's own repos; external otherwise.
2. Before selecting work, query the repository's open PRs, including drafts. Exclude every issue covered by an open or draft PR. Treat an issue as covered only when GitHub exposes an explicit Development/closing-issue link or the PR body uses a closing keyword for that issue; do not treat an incidental plain-text mention as coverage. Record the covering PR URL and skip reason, and do not create a seed, worktree, branch, or worker threads for that issue. Recheck immediately before each sequential dispatch; in parallel mode, run one fresh check immediately before the batch fan-out.
3. Derive a conventional branch name for each eligible issue as `<type>/<short-kebab-description>`. Choose the prefix from the work: `fix/` for bugs, `feat/` for features or enhancements, `docs/`, `refactor/`, `test/`, or `chore/` when those are more accurate. Keep the description lowercase and concise; an issue number may follow the prefix when useful, for example `fix/67-null-autofill` or `feat/manual-entry-spacing`. Do not use `codex/`, usernames, or other non-conventional namespace prefixes.
4. Allocate one dedicated Codex worktree per issue thread group, even when no other issue is active for the repo. No other issue may use it.
5. Create a neutral worktree seed thread with `create_thread` using the project worktree target. The seed prompt must only materialize the worktree and identify itself as a neutral seed; it must not invoke `$solve-issue`, use `$ponytail`, assign implementation/review responsibilities, or ask for code changes.
6. After the seed completes, create two user-owned, visible worker threads by calling `fork_thread` with `threadId` set to the seed thread ID and `environment: { type: "same-directory" }`. Do not call `fork_thread` on the implementation thread, review thread, or current orchestration thread for these workers. Title and pin both worker threads while active when the Codex app tools are available:
   - implementation thread: owns all code edits, commits, verification, branch updates, PR/draft updates, and review-response changes.
   - review thread: reads the PR/draft and writes only review comments. In external mode it may write only `REVIEW.md`; otherwise it must not modify repo files.
7. Send each worker a role bootstrap as its first follow-up message. Put the role statement before any skill invocation. The implementation bootstrap must say it is the implementation thread for the repo issue and should stand by for the full `$solve-issue` task packet. The review bootstrap must say it is the review thread, that any inherited neutral seed history is only worktree setup context, and that it must report `CONTEXT_MISMATCH issue=<number> expected=review` if the conversation history identifies it as an implementation thread. Do not send `$ponytail` or any review request as a bare first message.
8. Send `$solve-issue` to the implementation thread with an explicit task packet:
   - repo, issue number, issue URL, mode
   - implementation thread ID, review thread ID
   - neutral seed thread ID
   - conventional branch name (`<type>/<short-kebab-description>`), base branch, worktree path
   - artifact paths: `DRAFT.md`, `REVIEW.md`, `.codex-pr-media/`
   - PR result contract: `{ mode, prUrl | draftPath, branch, headSha, media }`
   - coverage preflight and skip marker: `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>`
   - review pass marker: `REVIEW_PASS issue=<number> head=<headSha>`
   - review start protocol: implementation thread sends the PR result and review instructions to the review thread
   - monitor protocol: implementation thread uses heartbeat automation; fallback `read_thread` checks are one-per-wakeup, at least 10 minutes apart, and back off to 30 minutes after two unchanged checks
   - stop protocol: after pass, send a final stop/archive message to the implementation thread and unpin/archive finished visible threads when appropriate
9. Dispatch according to the selected scheduling mode:
   - sequential: fully initialize and dispatch one issue group, then continue to the next issue only after that group stops.
   - parallel: initialize and dispatch every selected issue group without waiting for another group to finish. Run all independent seed creation, worker setup, titles, pins, and task-packet sends concurrently when the app tools allow it.
10. Monitor active issue groups through the same heartbeat cadence. On each wakeup, read each active implementation thread at most once, handle completed or skipped groups independently, and schedule the next check for unfinished groups. In parallel mode, keep monitoring all groups concurrently; one group's completion, skip, or failure must not stop the others.

Ask only if ownership is ambiguous. Do not implement here; only schedule isolated per-issue loops.
