---
name: solve-issues
description: "Turn exact or picked GitHub issues into independently reviewed draft PRs or local packages using isolated Ponytail developers and reviewers. Use for issue-to-PR runs, picked batches, or continuation; batches default to parallel."
---

# Solve Issues

## Select

- `$solve-issues`: select one eligible issue.
- `$solve-issues <positive-issue-number>`: select that issue only; never substitute another.
- `$solve-issues pick [positive-amount]`: select distinct issues in ascending order; default to `1`, or use all remaining when fewer qualify.
- Dispatch batches immediately in parallel unless the user requests sequential execution; then finish each issue before dispatching the next.

An issue is eligible when it is open and no open or draft PR covers it through a GitHub Development link or closing keyword. Ignore incidental mentions.

For an ineligible exact target, stop with:

```text
ISSUE_SKIPPED issue=<number> reason=<not_found|not_open>
ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>
```

## Invariants

- Let `$pr` choose draft-PR versus local-package publication; never store or pass that mode.
- Give each issue one sidebar-visible `DEV` in an isolated worktree and one independent, source-read-only `REV`, created only after `DEV` produces a committed head.
- Use no neutral seed, transient subagent, Ponytail-enabled `REV`, or `REV` forked from `DEV`.
- Keep review feedback inside Codex tasks; never post GitHub review activity.
- Call `set_thread_title` once per task and never change it:
  - Orchestrator: `Solve #<issue>` or `Solve <amount> issues`
  - Developer: `#<issue>: <short-description>`
  - Reviewer: `Review #<issue>`
- Key each run record by repo + issue. Store DEV/REV task IDs and cursors, absolute worktree, accepted head, PR URL or draft path, and lifecycle state.

```text
ORCH: selection + keyed state only
DEV:  source + commits + checks + media
REV:  verify head + review/fix loop + $pr
```

## Run

For each repository:

1. Query issues and open/draft PRs. Select without replacement; recheck before each sequential dispatch or once per parallel batch.
2. Create branch `<type>/<short-kebab-description>` using the narrowest of `fix`, `feat`, `docs`, `refactor`, `test`, or `chore`. Never namespace it.
3. Create `DEV` with repo, issue + URL, branch, base, absolute worktree, and optional media path. Require `DEV` to:
   - Activate `$ponytail` at full intensity.
   - Trust ORCH's coverage check. Recheck only after delay, interruption, or resumption; if covered, stop unchanged with `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>`.
   - For UI changes, exclude `.codex-pr-media/` through `.git/info/exclude` and capture matched, reproducible before/after media: video for interaction, motion, or multiple steps; images otherwise.
   - Implement, commit, check, and return `{ state: "ready_for_review", branch, headSha, checks, media }`; never publish.
   - Leave the branch unchanged after readiness. Send replacement results only to REV.
4. Store and verify DEV's first ready head. Create `REV` with issue, DEV task ID, repo, worktree, base, head, checks, and media.
5. Let REV own the loop:
   - Verify each head; report accepted head and lifecycle state to ORCH; run `$pr`.
   - On findings, send them to DEV, report `fix` to ORCH, and wait for DEV's replacement head.
   - Repeat until publication; report the published head and PR URL or draft path to ORCH.
6. After the first ready head, accept head and lifecycle updates only from REV. Matching publication is terminal. For later user-requested changes, set `fix`, send the request to DEV, and route DEV's replacement head to the same REV.

## Continue

Treat “continue,” “keep going,” “finish it,” unfinished status, and interrupted waits as continuation. Read [references/resume.md](references/resume.md), reconcile existing state, and resume without duplicating tasks or artifacts.
