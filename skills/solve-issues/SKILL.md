---
name: solve-issues
description: "Turn a requested number of open GitHub issues into reviewed changes. Accepts an optional positive integer amount, defaults to 1, has no hard maximum, and can resume interrupted or heartbeat-supervised runs without duplicating work. Use when the user says \"solve issues\", \"building loops\", asks to process open issues, wants issue-to-PR automation, or asks to resume a solve-issues run."
---

# Solve Issues

Invocation: `$solve-issues [amount] [publish]`. Treat an omitted amount as `1`; require a positive integer and never silently cap it. The amount is the number of eligible issues to process per repo, or all available eligible issues when fewer remain. It controls total work, not concurrency.

Default to prepare-only: local commits, checks, comparison media, `DRAFT.md`, and independent review, with no push or GitHub PR. Publish only when the current user request explicitly says `publish`, `open draft PRs`, or equivalent. Publish permission applies only to the user's repos, must survive resume unchanged, and never authorizes marking ready, merging, requesting reviewers, or enabling auto-merge.

Invocation: `$solve-issues resume`. On an explicit resume, a heartbeat wake, or recovery after interruption, read [references/resume.md](references/resume.md) and reconcile existing tasks and artifacts before creating anything.

Run sequentially unless the user explicitly asks for parallel or concurrent work. In parallel mode, dispatch every selected issue immediately; otherwise finish one before starting the next.

Use user-owned, sidebar-visible Codex app tasks for durable work. Give every issue a dedicated worktree and three-task group: one neutral seed plus implementation and review siblings forked from that seed with `environment: { type: "same-directory" }`. Never fork one worker from the other or use transient subagents. Archive or unpin the seed after capturing both worker IDs.

Use `set_thread_title` to keep progress visible in narrow sidebars. Title the orchestration task `ISS <amount> run`. Title workers `#<issue> <type> <status>`, where type is `SET`, `DEV`, or `REV`, and status is one of `setup`, `wait`, `build`, `check`, `draft`, `review`, `fix`, `pr`, `ci`, `done`, or `block`. Update titles only on stage changes; examples: `#65 DEV build`, `#65 REV review`.

For each repo:

1. Determine ownership and publish permission once. Ask only when ownership is ambiguous; never ask merely to upgrade a prepare-only run into publish mode.
2. Query open issues and open PRs, including drafts. Select the requested number of issues not covered by a GitHub Development/closing-issue link or closing keyword; an incidental mention does not count. Recheck immediately before each sequential dispatch or once just before a parallel batch.
3. Derive `<type>/<short-kebab-description>` using the narrowest accurate prefix from `fix`, `feat`, `docs`, `refactor`, `test`, or `chore`. An issue number is optional. Never use `codex/`, usernames, or another namespace.
4. Create the neutral seed in a dedicated worktree as `#<issue> SET setup`. Its prompt may only materialize the worktree and identify itself as neutral setup context.
5. Fork the implementation and review tasks as same-directory siblings. Title them `#<issue> DEV wait` and `#<issue> REV wait`, and pin both while active.
6. Bootstrap the implementation task as the sole owner of code, commits, verification, branch and PR updates. Bootstrap the review task as read-only, except that external reviews may write the supplied `REVIEW.md`. Both must initially stand by for the concrete issue contract.
7. Send the implementation task the contract below directly; do not invoke `$solve-issue` inside worker tasks.

The contract must include the repo, issue number and URL, ownership, publish permission, all three task IDs, branch and base branch, worktree, `DRAFT.md`, `REVIEW.md`, `.codex-pr-media/`, and these instructions:

- Read the issue, comments, linked PRs, and relevant code. Before the first edit, recheck open and draft PR coverage. If covered, stop without changes and return `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>`. Do not mistake a PR created by this loop for pre-existing coverage when resuming.
- Validate the supplied branch format and correct it before creating or pushing when necessary.
- Rename the implementation and review tasks at each real stage transition using the compact title format above.
- For a user-facing change, capture reproducible before media when possible. Set `DEV build`, implement the complete fix, commit it, set `DEV check`, run the repo's relevant checks, and capture after media.
- Set `DEV draft` and call `$pr` in prepare mode. Require `{ state: "prepared", draftPath, branch, headSha, media }` before review begins.
- Send the review task the issue context, worktree, branch, head SHA, draft path, media, review path, permissions, and `REVIEW_PASS issue=<number> head=<headSha>`. Start with: `You are the review task for <repo> issue #<number>. You are a same-directory sibling from neutral seed <seedTaskId>; inherited history is setup context only. If history identifies you as implementation, reply CONTEXT_MISMATCH issue=<number> expected=review.`
- Set `REV review`. The review task reads the branch and `DRAFT.md` and edits no code. For external repos, it keeps the review entirely local in `REVIEW.md` and never posts a GitHub comment, review, reaction, or other write. With no findings, it returns only the exact pass marker.
- Wait for review with one bounded `wait_threads` call at a time. On findings, set `DEV fix`, fix, commit, update the draft, and request review again with the new head SHA. On `CONTEXT_MISMATCH`, report the bad topology and stop.
- After a pass for the current head, complete a prepare-only run without external writes. With explicit publish permission, set `DEV pr`, call `$pr` in publish mode, then set `DEV ci` and wait for GitHub checks. Fix failures, update the draft PR, and repeat review for every new head. On the user's published PR, submit at most one GitHub `COMMENT` review per head, with inline findings when useful; begin its body exactly `Codex reviewed this PR at <shortHeadSha>.` Never impersonate the user or claim GitHub approval. Return the pass marker through the task, not in the GitHub comment. Complete only when CI is green and the exact pass marker matches the published head; leave the PR as a draft.

Monitor implementation tasks with `wait_threads`, preserving each task cursor and waiting over all active targets in one bounded call. Do not repeatedly poll with `read_thread`. Keep one run-level heartbeat attached to the orchestration task while unattended work remains; every wake runs the resume procedure, and terminal completion deletes the heartbeat. In sequential mode, dispatch the next issue only after the current implementation completes or skips; in parallel mode, manage each result independently. On completion, set worker titles to `done`, unpin or archive them, and report completed, skipped, and blocked issues with their local draft paths or draft PR links.

Do not implement in the orchestration task; schedule and monitor the isolated issue groups until the requested amount has reached a terminal result.
