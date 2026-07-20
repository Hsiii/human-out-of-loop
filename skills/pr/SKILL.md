---
name: pr
description: "Create a maintainer-ready draft PR for the user's repo, or a reviewed local PR draft for an external repo. Uses the repository template, conventional title, issue closure, comparison media, and ownership-based publishing. Use when the user says \"pr\", asks to draft or create a PR, or an implementation task hands off PR preparation."
---

# PR

Use the supplied mode, branch, worktree, and artifact paths. When called directly, choose internal for the user's repos and external otherwise; ask only when ownership is ambiguous. Worktree and branch ownership belong to `$solve-issues`.

Use the repository's PR template exactly when present. Title the PR `<type>(<scope>): <imperative summary>` with the narrowest meaningful Conventional Commits type and scope; omit scope only when none is useful, and reserve `style` for formatting-only changes rather than CSS or behavior. Include appropriate issue-closing syntax. Do not invent sections or add generic local command output unless the template requests it.

For user-facing changes, include clearly labeled `Before:` and `After:` states under the best template heading. Use captured media when available; when only after media exists, describe the prior state in text. Without a template, write a concise `## Description` and add `## Comparison` only for user-facing changes.

Prepare `DRAFT.md` for both modes, ignoring `DRAFT.md`, `REVIEW.md`, and `.codex-pr-media/` through `.git/info/exclude`. Return `{ state: "prepared", draftPath, branch, headSha, media }` for review.

External: stop after the reviewed local draft. Never push, open a PR, or post GitHub review activity.

Internal: after checks and review pass for the current head, use GitHub Yeet to push and open a real draft PR. Upload media in the GitHub editor and verify the saved body contains GitHub attachment URLs and no local paths. Never mark ready, merge, request reviewers, or enable auto-merge. Return `{ state: "published", prUrl, branch, headSha, media }` after verifying the URL and body.

When Codex reviews the published PR, submit at most one `COMMENT` review per head and begin its body exactly `Codex reviewed this PR at <shortHeadSha>.` Never impersonate the user or claim GitHub approval. The caller owns CI monitoring and final-head review.
