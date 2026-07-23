---
name: solve-issues
description: "Turn one exact or a picked batch of eligible GitHub issues into independently reviewed draft PRs, or local drafts for external repos, using isolated Ponytail developers and on-demand reviewers. Use for \"solve issue\", \"solve issues\", \"pick N\", building loops, issue-to-PR automation, or continuing an existing run; batches default to parallel."
---

# Solve Issues

Invocation:

- `$solve-issues` selects one eligible open issue.
- `$solve-issues <issue-number>` targets that exact positive issue number.
- `$solve-issues pick [amount]` selects distinct eligible open issues in ascending issue-number order. Treat an omitted amount as `1`, require a positive integer, and process all eligible issues when fewer than the requested amount remain.

An issue is eligible only when open and not covered by an open or draft PR through a GitHub Development link or closing keyword; an incidental mention does not count. For an exact target, never substitute another issue. Report `ISSUE_SKIPPED issue=<number> reason=<not_found|not_open|covered_by_pr>` and include the covering PR URL.

Let `$pr` determine whether publication creates a draft PR or a local PR package; do not store or pass publication mode.

Treat “continue,” “keep going,” “finish it,” an unfinished status request, or an interrupted wait as continuation behavior. Read [references/resume.md](references/resume.md) and reconcile existing tasks and artifacts before creating anything.

Run picked batches in parallel by default, dispatching every selected issue immediately. Run sequentially only when the user says “one by one,” “sequentially,” or equivalent; then finish one issue before starting the next.

Use the invoking task only as orchestrator: it owns selection and keyed lifecycle state. Give each issue one sidebar-visible `DEV` task in a dedicated worktree and one independent, code-read-only `REV/PR` task created only when a committed head is ready. `DEV` owns source changes, commits, checks, and fixes; `REV/PR` owns the DEV feedback loop and runs `$pr` for review and publication. Never create a neutral seed, activate `$ponytail` in `REV/PR`, fork `REV/PR` from `DEV`, or use transient subagents.

Use `set_thread_title` once after selection: title a single-issue orchestrator `Solve #<issue>`, a picked batch orchestrator `Solve <amount> issues`, each developer `#<issue>: <short-description>`, and each reviewer/publisher `Review #<issue>`. Keep these titles static.

Maintain one state record per repo + issue containing the two task IDs, absolute worktree, current accepted head, PR URL or draft path, lifecycle state, and both wait cursors.

For each repo:

1. Query issues and open/draft PRs, then select without replacement. Recheck immediately before a sequential dispatch or once before a parallel batch.
2. Derive `<type>/<short-kebab-description>` using the narrowest prefix from `fix`, `feat`, `docs`, `refactor`, `test`, or `chore`; never use a namespace such as `codex/` or a username.
3. Create `DEV` directly in the dedicated worktree, set its static title, and send the repo, issue and URL, branch, base, worktree, and optional media path:
   - Activate `$ponytail` at full intensity.
   - Trust the orchestrator's just-completed coverage check. Recheck only after a delayed start, an interruption, or a resumed run; if covered, stop unchanged with `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>`.
   - For every UI change, add `.codex-pr-media/` to `.git/info/exclude` and capture reproducible before and after media. Use matched videos whenever the change involves interaction, motion, or multiple steps; otherwise use matched images.
   - Return `{ state: "ready_for_review", branch, headSha, checks, media }`. Do not publish a PR.
   - After the first ready result, stand by without changing the branch until `REV/PR` sends review findings or the orchestrator sends a requested change.
   - After addressing one, send the replacement ready result only to `REV/PR`.
4. Store and verify the first ready head, then create `REV/PR`, set its static title, and send the issue, DEV task ID, repo, absolute worktree, base, head, checks, and media.
5. `REV/PR` owns the loop:
   - For each ready head, verify the worktree head, report its accepted head and lifecycle state to the orchestrator, then run `$pr`.
   - When `$pr` reports findings, send the actionable request to `DEV`, report `fix` state to the orchestrator, and wait for DEV's replacement ready head.
   - Repeat until `$pr` publishes successfully, then report the head and PR URL or draft path to the orchestrator.
6. After the first ready head, accept lifecycle and head updates only from `REV/PR`. A matching successful publication is terminal.
7. For requested changes after publication, set the lifecycle to `fix` and send the request to `DEV`; its replacement returns only to the same `REV/PR`.
