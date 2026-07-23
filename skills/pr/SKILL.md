---
name: pr
description: "Review committed changes and publish a maintainer-ready draft PR for the user's repository or a local PR package for an external repository. Use when the user invokes \"$pr\" or asks to draft, create, or open a PR."
---

# PR

Use explicitly supplied repository, worktree, and branch context; otherwise use the current thread's.

## Publication Preflight

1. Run the bundled `scripts/pr-preflight` with the target worktree as its working directory before review and again before publication. Never publish after `PR_PREFLIGHT_BLOCKED`.
2. Inspect its prospective commit list and diff for task scope.
3. For a spent or default branch, rebuild from the reported base with only the intended commits, then rerun preflight. Ask when that scope is ambiguous.
4. If the reported base or head changes, repeat the affected review. After publication, verify the PR base, head, and commits match the reviewed scope.

## Review and Publication

1. Use the preflight base and head. Verify the worktree head before each review.
2. Review the complete `base...head` diff. On actionable findings, report them and stop.
3. For every UI change, reuse supplied media or capture reproducible before and after media from the base and head. Use matched videos whenever the change involves interaction, motion, or multiple steps; otherwise use matched images. Keep media untracked through `.git/info/exclude`, match viewport, state, data, and action sequence, and never publish with either side missing.
4. Build one PR package:
   - Use the repository PR template exactly when present.
   - Title it `<type>(<scope>): <imperative summary>` with the narrowest Conventional Commits type. Omit scope when none is useful and reserve `style` for formatting-only changes.
   - When the change resolves an issue, put issue-closing syntax in the body.
   - Do not invent template sections or include generic command output.
   - For UI changes, put `Before:` and `After:` media under the best template heading. Without a template, use concise `## Description` and optional `## Comparison` sections.
5. Immediately before publication, verify the worktree `HEAD` still equals the reviewed head; otherwise publish nothing and return to the preflight. Use internal mode for the user's repositories and external mode otherwise; ask only when ownership is ambiguous:
   - External: write the exact body to `DRAFT.md` with local media references. Store the exact title in `.codex-pr-media/title` and the full reviewed head in `.codex-pr-media/reviewed-head`. Ignore `DRAFT.md` and `.codex-pr-media/` through `.git/info/exclude`, then report the title, absolute draft path, and reviewed head. Never write to GitHub.
   - Internal: push and create or update the branch's draft PR with the same title and body. If its existing PR is not a draft, stop without modifying it. For UI changes, upload media through the GitHub editor, replace only local media references with attachment URLs, and verify no local paths remain. Verify the saved draft PR head equals the reviewed head before reporting success.
6. For internal mode, inspect checks once and report skipped, passing, pending, or failing without polling.
