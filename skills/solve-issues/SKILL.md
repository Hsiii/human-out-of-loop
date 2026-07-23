---
name: solve-issues
description: "Turn one exact or a picked batch of eligible GitHub issues into independently reviewed draft PRs, or local drafts for external repos, using isolated Ponytail developers and on-demand reviewers. Use for \"solve issue\", \"solve issues\", \"pick N\", building loops, issue-to-PR automation, or continuing an existing run; batches default to parallel."
---

# Solve Issues

Invocation:

- `$solve-issues` selects one eligible open issue.
- `$solve-issues <issue-number>` targets that exact positive issue number.
- `$solve-issues pick [amount]` selects distinct eligible open issues. Treat an omitted amount as `1`, require a positive integer, and process all eligible issues when fewer than the requested amount remain.

An issue is eligible only when open and not covered by an open or draft PR through a GitHub Development link or closing keyword; an incidental mention does not count. For an exact target, never substitute another issue. Report `ISSUE_SKIPPED issue=<number> reason=<not_found|not_open|covered_by_pr>` and include the covering PR URL.

Use internal mode for the user's repos and external mode otherwise; ask only when ownership is ambiguous. Internal mode publishes a real draft PR. External mode writes reviewed local `DRAFT.md` without GitHub activity.

Treat “continue,” “keep going,” “finish it,” an unfinished status request, or an interrupted wait as continuation behavior. Read [references/resume.md](references/resume.md) and reconcile existing tasks and artifacts before creating anything.

Run picked batches in parallel by default, dispatching every selected issue immediately. Run sequentially only when the user says “one by one,” “sequentially,” or equivalent; then finish one issue before starting the next.

Use the invoking task only as orchestrator: it owns selection, keyed state, PR finalization, publication, and CI inspection. Give each issue one sidebar-visible `DEV` task in a dedicated worktree and one independent, read-only `REV` task created only when a committed head is ready. `DEV` solely owns implementation, commits, checks, and fixes; `REV` solely owns review. Never create a neutral seed, activate `$ponytail` in `REV`, fork `REV` from `DEV`, or use transient subagents.

Use `set_thread_title` once after selection: title a single-issue orchestrator `Solve #<issue>`, a picked batch orchestrator `Solve <amount> issues`, each developer `#<issue>: <short-description>`, and each reviewer `Review #<issue>`. Keep these titles static.

Maintain one state record per repo + issue containing the developer and reviewer task IDs, absolute worktree, branch, base, ready head, checks, media, reviewed head, PR URL, both wait cursors, and lifecycle state. Queue every unseen result by its task ID; never use one global current issue or overwrite one issue with another.

For each repo:

1. Determine mode once. Query issues and open/draft PRs, then select without replacement. Recheck immediately before a sequential dispatch or once before a parallel batch.
2. Derive `<type>/<short-kebab-description>` using the narrowest prefix from `fix`, `feat`, `docs`, `refactor`, `test`, or `chore`; never use a namespace such as `codex/` or a username.
3. Create `DEV` directly in the dedicated worktree, set its static title, and send the repo, issue and URL, mode, branch, base, worktree, and optional media path:
   - Activate `$ponytail` at full intensity. Read the issue, comments, linked PRs, repository instructions, and relevant code.
   - Trust the orchestrator's just-completed coverage check. Recheck only after a delayed start, an interruption, or a resumed run; if covered, stop unchanged with `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>`.
   - For a user-facing change, add `.codex-pr-media/` to `.git/info/exclude` and capture reproducible before media when possible.
   - Implement the smallest correct fix, run relevant checks, capture after media when applicable, and commit.
   - Return `{ state: "ready_for_review", branch, headSha, checks, media }`. Do not finalize a PR yet.
   - After returning a ready head, stand by without changing the branch until review findings or a CI fix request arrives.
   - On findings from `REV`, fix them, check and commit again, then return the new ready head.
4. Store each ready result in its issue record and verify its head equals `git -C <worktree> rev-parse HEAD`. On unexpected change, discard any pass and request a new ready result.
5. Create `REV` independently on the first ready head, title it, and send the issue, task IDs, repo, absolute worktree, base, and head:
   - Remain read-only. Never edit files or post GitHub comments, reviews, reactions, or approvals.
   - Verify the worktree head equals the requested head and ignore uncommitted changes.
   - On the first pass, inspect the complete committed `base...head` diff and relevant tests for correctness, regressions, scope, and the smallest correct solution.
   - On a later head, inspect `oldHead...newHead` and confirm prior findings. Repeat the full review only after context loss, reviewer replacement, or a broad change.
   - Send actionable findings directly to `DEV` and return `REVIEW_FINDINGS issue=<number> head=<headSha>` to the orchestrator. With no findings, return only `REVIEW_PASS issue=<number> head=<headSha>` to the orchestrator.
6. Reuse the reviewer for every head. Accept a pass only when its issue, reviewer task ID, and head match the current record.
7. Reverify the reviewed head, then finalize in the orchestrator:
   - Use the repository PR template exactly when present. Use `<type>(<scope>): <imperative summary>` with the narrowest Conventional Commits type, reserve `style` for formatting-only changes, include issue-closing syntax, and add no invented sections or generic command output.
   - For user-facing changes, include `Before:` and `After:` with captured media; when only after media exists, describe the prior state. Without a template, use concise `## Description` and optional `## Comparison`.
   - External: write `DRAFT.md`, ignore it and `.codex-pr-media/` through `.git/info/exclude`, and never write to GitHub.
   - Internal: scope Git to the absolute worktree and GitHub to the explicit repo/branch, push, and create or update a real draft PR without `DRAFT.md`. Upload media and verify the saved body has GitHub attachment URLs and no local paths.
   - Never mark ready, merge, request reviewers, enable auto-merge, or post GitHub review activity.
8. For internal mode, inspect CI once and report skipped, pending, or passing. Send an already-failing check to that issue's `DEV`, route the new head to the same `REV`, and update the PR after a new pass. Do not poll pending checks unless asked.
9. Let workers run in parallel; serialize only state transitions and finalization.

Monitor all active developer and reviewer tasks with `wait_threads`, preserving every per-task cursor and waiting over all targets in one bounded call. Do not repeatedly poll with `read_thread`. Ordinary foreground runs need no heartbeat; create one only when work must continue unattended across wakeups. In sequential mode, dispatch the next issue only after the current issue completes or skips. Follow [references/resume.md](references/resume.md) after interruption. Report completed, skipped, and blocked issues with their local draft paths or draft PR links.

Do not implement or review in the orchestration task. A reviewed draft PR with CI checked once is terminal even when checks are still pending; report that status instead of polling. Continue pending CI only when the user asks.
