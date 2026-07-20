---
name: pr
description: "Create a maintainer-ready draft PR for the user's repo, or a reviewed local PR draft for an external repo. Uses the repository template, conventional title, issue closure, full-diff review, comparison media, and ownership-based publishing. Use when the user says \"pr\", asks to draft or create a PR, or an implementation task hands off PR preparation."
---

# PR

When `$solve-issues` calls this skill, use its supplied mode, repo, base, branch, worktree, artifact paths, and review result. In prepare mode, draft only and return for the separate reviewer; in publish mode, require that review's exact current-head pass.

When called directly:

1. Use the current repository, worktree, and branch; derive the base from the remote default branch and artifact paths from the worktree. Choose internal for the user's repos and external otherwise; ask only when ownership is ambiguous.
2. Inspect the complete base-to-head diff, commits, linked issue, repository instructions, and working-tree state. Require the intended PR changes to be committed before publication. Never stage or commit unrelated changes; ask if intended changes remain uncommitted.
3. Run relevant local checks, capture comparison media for user-facing changes, and prepare `DRAFT.md` with the framework below.
4. Review the complete base-to-head diff read-only. If it has findings, report them and stop with the local draft. Otherwise treat the current head as the direct call's review pass.

Use the repository's PR template exactly when present. Title the PR `<type>(<scope>): <imperative summary>` with the narrowest meaningful Conventional Commits type and scope; omit scope only when none is useful, and reserve `style` for formatting-only changes rather than CSS or behavior. Include appropriate issue-closing syntax. Do not invent sections or add generic local command output unless the template requests it.

For user-facing changes, include clearly labeled `Before:` and `After:` states under the best template heading. Use captured media when available; when only after media exists, describe the prior state in text. Without a template, write a concise `## Description` and add `## Comparison` only for user-facing changes.

Prepare `DRAFT.md` for both modes, ignoring `DRAFT.md`, `REVIEW.md`, and `.codex-pr-media/` through `.git/info/exclude`. In orchestrated prepare mode, return `{ state: "prepared", draftPath, branch, headSha, media }` for review.

External: stop after the reviewed local draft. Never push, open a PR, or post GitHub review activity.

Internal: after checks and a review pass covering the whole base-to-current-head diff, use available authenticated Git and GitHub tooling to push and open a real draft PR. GitHub Yeet is optional, not required. Upload media in the GitHub editor and verify the saved body contains GitHub attachment URLs and no local paths. Never mark ready, merge, request reviewers, or enable auto-merge. In direct mode, inspect checks after publication: skip when none are configured, otherwise wait for green and report failures without editing implementation code. Return `{ state: "published", prUrl, branch, headSha, media }` after verifying the URL and body.

When Codex reviews the published PR, review the complete PR diff from its base through the current head, never only the latest commit. Submit at most one `COMMENT` review per head and begin its body exactly `Codex reviewed this PR at <shortHeadSha>.` Never impersonate the user or claim GitHub approval. In orchestrated mode, the caller owns CI monitoring and full-PR review of the final head.
