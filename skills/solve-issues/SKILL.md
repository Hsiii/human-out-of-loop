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

Use internal mode for the user's repos and external mode otherwise; ask only when ownership is ambiguous. Internal mode publishes a real draft PR. External mode writes reviewed local `DRAFT.md` without GitHub activity.

Treat “continue,” “keep going,” “finish it,” an unfinished status request, or an interrupted wait as continuation behavior. Read [references/resume.md](references/resume.md) and reconcile existing tasks and artifacts before creating anything.

Run picked batches in parallel by default, dispatching every selected issue immediately. Run sequentially only when the user says “one by one,” “sequentially,” or equivalent; then finish one issue before starting the next.

Use the invoking task only as orchestrator: it owns selection and keyed lifecycle state. Give each issue one sidebar-visible `DEV` task in a dedicated worktree and one independent, code-read-only `REV/PR` task created only when a committed head is ready. `DEV` owns source changes, commits, checks, and fixes; `REV/PR` uses `$pr` to own review, draft publication, PR metadata, and CI inspection. Never create a neutral seed, activate `$ponytail` in `REV/PR`, fork `REV/PR` from `DEV`, or use transient subagents.

Use `set_thread_title` once after selection: title a single-issue orchestrator `Solve #<issue>`, a picked batch orchestrator `Solve <amount> issues`, each developer `#<issue>: <short-description>`, and each reviewer/publisher `Review #<issue>`. Keep these titles static.

Maintain one state record per repo + issue containing the developer and reviewer task IDs, absolute worktree, branch, base, ready head, checks, media, reviewed head, PR title, PR URL or draft path, buffered reviewer results, both wait cursors, and lifecycle state. Key buffered and processed results by task ID + result kind + head SHA, ignore exact replays, and never use one global current issue or overwrite one issue with another.

For each repo:

1. Determine mode once. Query issues and open/draft PRs, then select without replacement. Recheck immediately before a sequential dispatch or once before a parallel batch.
2. Derive `<type>/<short-kebab-description>` using the narrowest prefix from `fix`, `feat`, `docs`, `refactor`, `test`, or `chore`; never use a namespace such as `codex/` or a username.
3. Create `DEV` directly in the dedicated worktree, set its static title, and send the repo, issue and URL, mode, branch, base, worktree, and optional media path:
   - Activate `$ponytail` at full intensity.
   - Trust the orchestrator's just-completed coverage check. Recheck only after a delayed start, an interruption, or a resumed run; if covered, stop unchanged with `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>`.
   - For every UI change, add `.codex-pr-media/` to `.git/info/exclude` and capture reproducible before and after media. Use matched videos whenever the change involves interaction, motion, or multiple steps; otherwise use matched images.
   - Return `{ state: "ready_for_review", branch, headSha, checks, media }`. Do not publish a PR.
   - After returning a ready head, stand by without changing the branch until review findings, a CI fix, or a requested change arrives.
   - After addressing one, send the new ready result to `REV/PR` and the orchestrator.
4. Store each ready result in its issue record and verify its head equals `git -C <worktree> rev-parse HEAD`. On unexpected change, discard any pass and request a new ready result.
5. Create `REV/PR` independently on the first ready head, title it, and send the issue, both task IDs, repo, absolute worktree, base, mode, head, checks, and media. Tell it to run `$pr`:
   - Remain code-read-only and keep review communication inside Codex tasks.
   - Send findings or failing CI directly to `DEV`, report the matching intermediate state to the orchestrator, and stand by for the replacement ready head.
   - On a review pass, publish or update the draft through `$pr` and return its `PR_READY` result to the orchestrator.
6. Reuse `REV/PR` for every head. If its result arrives before the matching DEV ready result is stored, buffer it by issue + head and reconcile it after that DEV result and worktree head are verified; discard it only when a newer verified ready head supersedes it. Otherwise accept a result only when its issue, task ID, and head match the current record. A matching `PR_READY` is terminal when checks are skipped, passing, or pending; report an unfixable failure as blocked.
7. For requested changes after publication, send the request to `DEV`; route its new ready head to the same `REV/PR` for review and draft update.
