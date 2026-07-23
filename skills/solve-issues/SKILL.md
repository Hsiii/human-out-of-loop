---
name: solve-issues
description: "Turn exact or picked GitHub issues into independently reviewed draft PRs or local packages using isolated Ponytail developers and reviewers. Use for issue-to-PR runs, picked batches, or continuation; batches default to parallel."
---

# Solve Issues

## Invocation and Eligibility

- `$solve-issues` selects one eligible open issue.
- `$solve-issues <issue-number>` targets that exact positive issue number.
- `$solve-issues pick [amount]` selects distinct eligible issues in ascending order. Default to `1`, require a positive integer, and use all remaining issues when fewer qualify.

Eligible issues are open and not covered by an open or draft PR through a GitHub Development link or closing keyword; incidental mentions do not count. Never substitute an exact target. Report `ISSUE_SKIPPED issue=<number> reason=<not_found|not_open|covered_by_pr>` with the covering PR URL.

Dispatch picked issues immediately in parallel. When asked for sequential execution, finish one before dispatching the next.

## Ownership and Topology

Let `$pr` determine whether publication creates a draft PR or a local PR package; do not store or pass publication mode.

```text
ORCH
 ├─ creates DEV ── owns source, commits, checks, and media
 └─ creates REV ── source-read-only; invokes $pr

DEV ── ready(head) ────────────> REV
DEV <── actionable findings ─── REV
REV ── head + state + result ──> ORCH
$pr ──> draft PR | local PR package
```

`ORCH` owns only selection and keyed state. Each issue gets one sidebar-visible `DEV` in a dedicated worktree and one independent `REV`, created only for a committed head. Never create a neutral seed, activate `$ponytail` in `REV`, fork `REV` from `DEV`, or use transient subagents.

## Run Record

Call `set_thread_title` once: `Solve #<issue>` for one issue, `Solve <amount> issues` for a batch, `#<issue>: <short-description>` for `DEV`, and `Review #<issue>` for `REV`. Keep titles static.

Key one record by repo + issue: both task IDs and cursors, absolute worktree, accepted head, PR URL or draft path, and lifecycle state.

## Protocol

For each repo:

1. Query issues and open/draft PRs; select without replacement. Recheck before each sequential dispatch or once per parallel batch.
2. Derive `<type>/<short-kebab-description>` using the narrowest prefix from `fix`, `feat`, `docs`, `refactor`, `test`, or `chore`; never use a namespace such as `codex/` or a username.
3. Create `DEV` in its worktree with the repo, issue and URL, branch, base, worktree, and optional media path:
   - Activate `$ponytail` at full intensity.
   - Trust `ORCH`'s coverage check. Recheck only after delay, interruption, or resumption; if covered, stop unchanged with `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>`.
   - For UI changes, exclude `.codex-pr-media/` through `.git/info/exclude`. Capture matched, reproducible before/after videos for interaction, motion, or multiple steps; use images otherwise.
   - Return `{ state: "ready_for_review", branch, headSha, checks, media }`. Do not publish a PR.
   - Then leave the branch unchanged until `REV` sends findings or `ORCH` requests a change. Send replacement ready results only to `REV`.
4. Store and verify the first ready head. Create `REV` with the issue, DEV task ID, repo, worktree, base, head, checks, and media.
5. `REV` owns the loop:
   - Verify each head, report its accepted head and lifecycle state to `ORCH`, then run `$pr`.
   - On findings, request action from `DEV`, report `fix` to `ORCH`, and await its replacement head.
   - Repeat until publication; report the head and PR URL or draft path to `ORCH`.
6. After the first ready head, accept head and lifecycle updates only from `REV`; matching publication is terminal. For later requested changes, set `fix`, send the request to `DEV`, and route its replacement to the same `REV`.

## Resume

Treat “continue,” “keep going,” “finish it,” unfinished status, or interrupted waits as continuation. Read [references/resume.md](references/resume.md) and reconcile before creating anything.
