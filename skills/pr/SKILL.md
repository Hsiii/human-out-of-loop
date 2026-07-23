---
name: pr
description: "Review committed changes and publish a maintainer-ready draft PR for the user's repository or a local PR package for an external repository. Use when the user invokes \"$pr\", asks to draft, create, or open a PR, or when a solve-issues reviewer must review and publish a ready head."
---

# PR

A direct `$pr` call after finishing work in a thread uses that thread's current repository, worktree, and branch and runs the complete flow. When `$solve-issues` launches `$pr` in `REV/PR`, use the supplied issue record and run the same flow; only findings and failing CI are routed to `DEV`.

## Publication Preflight

Before reviewing or publishing:

1. From the target worktree, run `scripts/pr-preflight`. Treat any `PR_PREFLIGHT_BLOCKED` result as a hard stop. The script fetches the remote default branch, rejects dirty or default-branch work, queries all PR history for the head branch, and prints the exact base, head, and prospective commits.
2. Apply the branch-history result:
   - Reuse an existing open draft only when that branch has no separate closed or merged PR.
   - Stop without modifying an existing non-draft PR.
   - Treat a branch with any separate closed or merged PR as spent, even when it also has a newer open PR. Never publish from it again.
3. When the branch is spent, is the default branch, or contains commits outside the current task, create a fresh `<type>/<short-description>` branch from the fetched base and transplant only the intended task commits. If that commit scope is ambiguous, stop and ask instead of guessing.
4. Inspect the prospective `base...head` commit list and diff. Do not publish when either contains unrelated commits or commits already represented by a prior merged PR.
5. Record the fetched base SHA and intended head. Immediately before publication, run `scripts/pr-preflight` again; if the base or head changed, repeat the scope check and any affected review. After creating or updating a PR, verify its base branch, head SHA, and commit list match the reviewed scope.

## Review and Publication

1. Use the preflight base and head. Verify the worktree head before each review.
2. Review `base...head` on the first pass. On a replacement head, review `oldHead...newHead`, confirm prior findings, and repeat the full review only after context loss or a broad change. Never edit source code or commits.
3. On actionable findings:
   - With a developer task, send the findings there, report `PR_FINDINGS issue=<number|none> head=<headSha>` to the orchestrator, and stand by for a new ready head.
   - Without a developer task, report the findings and stop.
4. For every UI change, reuse supplied media or capture reproducible before and after media from the base and head. Use matched videos whenever the change involves interaction, motion, or multiple steps; otherwise use matched images. Keep media untracked through `.git/info/exclude`, match viewport, state, data, and action sequence, and never publish with either side missing.
5. Build one PR package:
   - Use the repository PR template exactly when present.
   - Title it `<type>(<scope>): <imperative summary>` with the narrowest Conventional Commits type. Omit scope when none is useful and reserve `style` for formatting-only changes.
   - When the change resolves an issue, put issue-closing syntax in the body.
   - Do not invent template sections or include generic command output.
   - Put `Before:` and `After:` media under the best template heading. Without a template, use concise `## Description` and optional `## Comparison` sections.
6. Immediately before publication, verify the worktree `HEAD` still equals the reviewed head; otherwise publish nothing and return to the preflight. Use internal mode for the user's repositories and external mode otherwise; ask only when ownership is ambiguous:
   - External: write the exact body to `DRAFT.md` with local media references. Store the exact title in `.codex-pr-media/title` and the full reviewed head in `.codex-pr-media/reviewed-head`. Ignore `DRAFT.md` and `.codex-pr-media/` through `.git/info/exclude`, then report the title, absolute draft path, and reviewed head. Never write to GitHub.
   - Internal: push and create or update the branch's draft PR with the same title and body. If its existing PR is not a draft, stop without modifying it. For UI changes, upload media through the GitHub editor, replace only local media references with attachment URLs, and verify no local paths remain. Verify the saved draft PR head equals the reviewed head before reporting success.
7. For internal mode, inspect checks once:
   - When skipped, passing, or pending, report `PR_READY issue=<number|none> head=<headSha> title=<title> target=<url> checks=<state>` and stop without polling.
   - When failing with a developer task, send the failure there, report `PR_CI_FIX issue=<number|none> head=<headSha>` to the orchestrator, and stand by for a new ready head.
   - When failing without a developer task, report the failure and stop.

Use `issue=none` when no linked issue exists. For a new head after findings, CI failure, or requested changes, return to step 1 and update an existing draft only after the new head passes review. In external mode, report `PR_READY` with the draft path and `checks=skipped`.
