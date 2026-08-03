---
name: pr
description: "Review committed changes and publish a maintainer-ready draft PR in the current branch's writable remote repository. Use when the user invokes \"$pr\" or asks to draft, create, or open a PR."
---

## Preflight

1. Treat the current branch's Git remote as the sole publication target. Never infer a repository from GitHub CLI context, follow a parent/upstream repository, create or synchronize a fork, or publish anywhere else. Require authenticated write access; otherwise stop and ask the user to run from a branch whose remote is their writable fork or repository.
2. Run the bundled `bash scripts/pr-preflight` from the target worktree before review and again before publication. Never review or publish while blocked.
3. Inspect its prospective commits and complete diff for task scope.
4. If the branch is default or spent, rebuild from the reported base with only intended commits, then rerun preflight.
5. If base or head changes, repeat the affected review.

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

For UI changes, run the bundled `bash scripts/pr-media-upload --available`. When it succeeds, publish each file with `bash scripts/pr-media-upload --repo OWNER/REPO --pr NUMBER <file>`; otherwise use the GitHub editor. Replace only local media references with the returned Markdown or attachment URLs, and verify no local paths remain.

Push to the exact `remote` reported by preflight. Create or update the branch's draft PR against the reported `repo` and `base_branch`, passing `--repo OWNER/REPO` explicitly to every GitHub CLI command; never rely on its inferred repository. Stop without modification if the existing PR is not a draft. Verify the saved PR's base and head repositories both equal the reported `repo`, and its base, head, and commits match the reviewed scope.

Only create or update drafts. Never mark ready, merge, request reviewers, enable auto-merge, or post GitHub review activity.
