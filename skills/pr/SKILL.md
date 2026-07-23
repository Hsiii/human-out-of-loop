---
name: pr
description: "Review committed changes and publish a maintainer-ready draft PR for the user's repository or a local PR package for an external repository. Use when the user invokes \"$pr\" or asks to draft, create, or open a PR."
---

## Preflight

1. Run the bundled `scripts/pr-preflight` from the target worktree before review and again before publication. Never review or publish while blocked.
2. Inspect its prospective commits and complete diff for task scope.
3. If the branch is default or spent, rebuild from the reported base with only intended commits, then rerun preflight.
4. If base or head changes, repeat the affected review.

## Review

1. Use the preflight base and head; verify worktree HEAD before each review.
2. Review the complete `base...head` diff. Report actionable findings and stop.
3. For every UI change, reuse supplied media or capture matched, reproducible before/after media from base and head:
   - Use video for interaction, motion, or multiple steps; images otherwise.
   - Match viewport, state, data, and action sequence.
   - Keep media untracked through `.git/info/exclude`.
   - Never publish with either side missing.

## Package

- Use the repository PR template exactly when present.
- Title: `<type>(<scope>): <imperative summary>` using the narrowest Conventional Commits type. Omit an unhelpful scope; reserve `style` for formatting-only changes.
- Add issue-closing syntax when the change resolves an issue.
- Do not invent template sections or include generic command output.
- Put UI `Before:` and `After:` media under the best template heading. Without a template, use concise `## Description` and optional `## Comparison`.

## Publish

Immediately before publication, verify worktree HEAD equals the reviewed head. If not, restart preflight.

| Mode | Action |
| --- | --- |
| External repository | Write the exact body with local media references to `DRAFT.md`; write the exact title to `.codex-pr-media/title` and full reviewed head to `.codex-pr-media/reviewed-head`; ignore both paths through `.git/info/exclude`; report title, absolute draft path, and reviewed head. Never write to GitHub. |
| User-owned repository | Push and create or update the branch's draft PR with the same title and body. Stop without modification if its existing PR is not a draft. For UI changes, prefer `pr-media-upload --repo OWNER/REPO --pr NUMBER <file>` when available and replace local references with its returned Markdown; otherwise use the GitHub editor. Verify no local paths remain. |

For a user-owned repository, verify the saved draft PR's base, head, and commits match the reviewed scope.

Only create or update drafts. Never mark ready, merge, request reviewers, enable auto-merge, or post GitHub review activity.
