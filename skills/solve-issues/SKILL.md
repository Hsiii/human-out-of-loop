---
name: solve-issues
description: "Turn a requested number of open GitHub issues into reviewed PRs. Accepts an optional positive integer amount, defaults to 1, has no hard maximum, skips issues already covered by open or draft PRs, and isolates each issue in visible implementation and review Codex tasks. Use when the user says \"solve issues\", \"building loops\", asks to process open issues, or wants issue-to-PR automation."
---

# Solve Issues

Invocation: `$solve-issues [amount]`. Treat an omitted amount as `1`; require a positive integer and never silently cap it. The amount is the number of eligible issues to process per repo, or all available eligible issues when fewer remain. It controls total work, not concurrency.

Run sequentially unless the user explicitly asks for parallel or concurrent work. In parallel mode, dispatch every selected issue immediately; otherwise finish one before starting the next.

Use user-owned, sidebar-visible Codex app tasks for durable work. Give every issue a dedicated worktree and three-task group: one neutral seed plus implementation and review siblings forked from that seed with `environment: { type: "same-directory" }`. Never fork one worker from the other or use transient subagents. Archive or unpin the seed after capturing both worker IDs.

For each repo:

1. Determine mode once: internal for the user's repos, external otherwise. Ask only when ownership is ambiguous.
2. Query open issues and open PRs, including drafts. Select the requested number of issues not covered by a GitHub Development/closing-issue link or closing keyword; an incidental mention does not count. Recheck immediately before each sequential dispatch or once just before a parallel batch.
3. Derive `<type>/<short-kebab-description>` using the narrowest accurate prefix from `fix`, `feat`, `docs`, `refactor`, `test`, or `chore`. An issue number is optional. Never use `codex/`, usernames, or another namespace.
4. Create the neutral seed in a dedicated worktree. Its prompt may only materialize the worktree and identify itself as neutral setup context.
5. Fork the implementation and review tasks as same-directory siblings. Title and pin both while active.
6. Bootstrap the implementation task as the sole owner of code, commits, verification, branch and PR updates. Bootstrap the review task as read-only, except that external reviews may write the supplied `REVIEW.md`. Both must initially stand by for the concrete issue contract.
7. Send the implementation task the contract below directly; do not invoke `$solve-issue` inside worker tasks.

The contract must include the repo, issue number and URL, mode, all three task IDs, branch and base branch, worktree, `DRAFT.md`, `REVIEW.md`, `.codex-pr-media/`, and these instructions:

- Read the issue, comments, linked PRs, and relevant code. Before the first edit, recheck open and draft PR coverage. If covered, stop without changes and return `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>`. Do not mistake a PR created by this loop for pre-existing coverage when resuming.
- Validate the supplied branch format and correct it before creating or pushing when necessary.
- For a user-facing change, capture reproducible before media when possible. Implement the complete fix, commit it, run the repo's relevant checks, and capture after media.
- Call `$pr` with the supplied mode and paths. Require `{ mode, prUrl | draftPath, branch, headSha, media }` before review begins.
- Send the review task the issue context, worktree, branch, head SHA, PR URL or draft path, media, review path, permissions, and `REVIEW_PASS issue=<number> head=<headSha>`. Start with: `You are the review task for <repo> issue #<number>. You are a same-directory sibling from neutral seed <seedTaskId>; inherited history is setup context only. If history identifies you as implementation, reply CONTEXT_MISMATCH issue=<number> expected=review.`
- The review task comments on the internal draft PR, or writes actionable external findings only to `REVIEW.md`. It edits no code. With no findings, it returns only the exact pass marker.
- Wait for review with one bounded `wait_threads` call at a time. On findings, fix, commit, update the PR or draft, and request review again with the new head SHA. On `CONTEXT_MISMATCH`, report the bad topology and stop. On the exact pass marker, report the completed PR result.

Monitor implementation tasks with `wait_threads`, preserving each task cursor and waiting over all active targets in one bounded call. Do not create heartbeat automations or repeatedly poll with `read_thread`. In sequential mode, dispatch the next issue only after the current implementation completes or skips; in parallel mode, manage each result independently. On completion, unpin or archive finished workers and report completed, skipped, and blocked issues with their PR or draft links.

Do not implement in the orchestration task; schedule and monitor the isolated issue groups until the requested amount has reached a terminal result.
