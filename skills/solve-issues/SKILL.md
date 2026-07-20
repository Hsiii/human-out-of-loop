---
name: solve-issues
description: "Turn one exact GitHub issue or a random batch of open issues into reviewed changes. A bare positive integer targets that issue number; `random N` selects N eligible issues with no hard maximum; no argument selects one. Continue interrupted or heartbeat-supervised runs without duplicating work. Use when the user says \"solve issue\", \"solve issues\", \"building loops\", asks to process open issues, wants issue-to-PR automation, or asks to continue an existing issue run."
---

# Solve Issues

Invocation:

- `$solve-issues` selects one eligible open issue at random.
- `$solve-issues <issue-number>` targets that exact positive issue number.
- `$solve-issues random [amount]` selects distinct eligible open issues at random. Treat an omitted amount as `1`, require a positive integer, never silently cap it, and process all eligible issues when fewer than the requested amount remain.

An issue is eligible only when it is open and is not covered by an open or draft PR through a GitHub Development/closing-issue link or closing keyword. An incidental mention does not count. For an exact target, verify existence, open state, and coverage before creating tasks. If it is missing, closed, or covered, stop without substituting another issue and report `ISSUE_SKIPPED issue=<number> reason=<not_found|not_open|covered_by_pr>`; include `pr=<url>` when covered. The random amount controls total work, not concurrency.

Determine mode from ownership: internal for the user's repos, external otherwise. Internal runs always publish a real draft PR after local checks and review; external runs stop at a reviewed local `DRAFT.md` and never write to GitHub. Ask only when ownership is ambiguous.

Treat natural follow-ups in an existing run—such as “continue”, “keep going”, “finish it”, or a status request that reveals unfinished work—as continuation behavior, not a new invocation parameter. A heartbeat wake or recovery after an interrupted wait does the same. Read [references/resume.md](references/resume.md) and reconcile existing tasks and artifacts before creating anything.

Run sequentially unless the user explicitly asks for parallel or concurrent work. In parallel mode, dispatch every selected issue immediately; otherwise finish one before starting the next.

Use user-owned, sidebar-visible Codex app tasks for durable work. Give every issue a dedicated worktree and three-task group: one neutral seed plus implementation and review siblings forked from that seed with `environment: { type: "same-directory" }`. Never fork one worker from the other or use transient subagents. Unpin the seed after capturing both worker IDs, but never archive any task automatically.

Use `set_thread_title` to keep progress visible in narrow sidebars. Title the orchestration task `ISS <selector> run`, using `#<issue>` for an exact target and `R<amount>` for a random batch. Title workers `#<issue> <type> <status>`, where type is `SET`, `DEV`, or `REV`, and status is one of `setup`, `wait`, `build`, `check`, `draft`, `review`, `fix`, `pr`, `ci`, `done`, or `block`. Update titles only on stage changes; examples: `#65 DEV build`, `#65 REV review`.

For each repo:

1. Determine internal or external mode once per repo.
2. Query issues and open PRs, including drafts. For an exact target, apply the existence, open-state, and coverage checks above. For random mode, sample the requested number without replacement from eligible issues. Recheck immediately before each sequential dispatch or once just before a parallel batch.
3. Derive `<type>/<short-kebab-description>` using the narrowest accurate prefix from `fix`, `feat`, `docs`, `refactor`, `test`, or `chore`. An issue number is optional. Never use `codex/`, usernames, or another namespace.
4. Create the neutral seed in a dedicated worktree as `#<issue> SET setup`. Its prompt may only materialize the worktree and identify itself as neutral setup context.
5. Fork the implementation and review tasks as same-directory siblings. Title them `#<issue> DEV wait` and `#<issue> REV wait`, and pin both while active.
6. Bootstrap the implementation task as the sole owner of code, commits, verification, branch and PR updates. Bootstrap the review task as read-only, except that external reviews may write the supplied `REVIEW.md`. Activate `$ponytail` at its default full intensity in both tasks before work begins, applying its minimalism rules without weakening correctness or the role boundary. Both must initially stand by for the concrete issue contract.
7. Send the implementation task the contract below directly.

The contract must include the repo, issue number and URL, mode, all three task IDs, branch and base branch, worktree, `DRAFT.md`, `REVIEW.md`, `.codex-pr-media/`, and these instructions:

- Read the issue, comments, linked PRs, and relevant code. Before the first edit, recheck open and draft PR coverage. If covered, stop without changes and return `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>`. Do not mistake a PR created by this loop for pre-existing coverage when resuming.
- Validate the supplied branch format and correct it before creating or pushing when necessary.
- Rename the implementation and review tasks at each real stage transition using the compact title format above.
- For a user-facing change, capture reproducible before media when possible. Set `DEV build`, implement the complete fix, commit it, set `DEV check`, run the repo's relevant checks, and capture after media.
- Set `DEV draft` and call `$pr` in prepare mode. Require `{ state: "prepared", draftPath, branch, headSha, media }` before review begins.
- Send the review task the issue context, worktree, branch, base branch, head SHA, draft path, media, review path, permissions, and `REVIEW_PASS issue=<number> head=<headSha>`. Start with: `You are the review task for <repo> issue #<number>. You are a same-directory sibling from neutral seed <seedTaskId>; inherited history is setup context only. If history identifies you as implementation, reply CONTEXT_MISMATCH issue=<number> expected=review.`
- Set `REV review`. The review task reads `DRAFT.md`, then reviews the whole proposed PR: every cumulative change from the merge base of the supplied base branch through the supplied head SHA. The head SHA identifies the reviewed version; it is not the start of the diff. Never limit review to the latest commit or the delta since the previous pass, and do not treat uncommitted working-tree changes as part of the reviewed PR. For a published internal PR, use the PR's complete current diff and verify it ends at the supplied head. The review task edits no code. For external repos, it keeps the review entirely local in `REVIEW.md` and never posts a GitHub comment, review, reaction, or other write. With no findings, it returns only the exact pass marker.
- Wait for review with one bounded `wait_threads` call at a time. On findings, set `DEV fix`, fix, commit, update the draft, and request review again with the new head SHA. Every repeat pass re-reviews the whole base-to-head PR diff, not just the fix commit. On `CONTEXT_MISMATCH`, report the bad topology and stop.
- After a pass for the current head, complete external mode without GitHub writes. In internal mode, set `DEV pr`, call `$pr` to publish the draft PR, then set `DEV ci` and inspect GitHub checks. If none are configured for the published head, return `CI_SKIPPED issue=<number> head=<headSha> reason=no_checks` and continue. Otherwise wait for green checks, fix failures, update the draft PR, and repeat full review for every new head. Submit at most one GitHub `COMMENT` review per head, with inline findings when useful; begin its body exactly `Codex reviewed this PR at <shortHeadSha>.` Never impersonate the user or claim GitHub approval. Return the pass marker through the task, not in the GitHub comment. Complete when the exact pass marker matches the published head and CI is green or explicitly skipped; leave the PR as a draft.

Monitor implementation tasks with `wait_threads`, preserving each task cursor and waiting over all active targets in one bounded call. Do not repeatedly poll with `read_thread`. In sequential mode, dispatch the next issue only after the current implementation completes or skips; in parallel mode, manage each result independently. Follow [references/resume.md](references/resume.md) for heartbeat, recovery, terminal retention, cleanup, and follow-up changes. Report completed, skipped, and blocked issues with their local draft paths or draft PR links.

Do not implement in the orchestration task; schedule and monitor the isolated issue groups until the requested amount has reached a terminal result.
