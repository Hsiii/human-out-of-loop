---
name: pr
description: "Prepare a maintainer-ready local PR draft, or publish a real GitHub draft PR only with explicit user permission. Uses the repository template, conventional title, issue closure, comparison media, and a structured result for issue loops. Use when the user says \"pr\", asks to draft or create a PR, or an implementation task hands off PR preparation or publication."
---

# PR

Use the supplied ownership, publish permission, branch, worktree, and artifact paths. When called directly, default to prepare-only and ask about ownership only when ambiguous. Only an explicit request to publish or open a draft PR grants publish permission; `pr` or `draft PR` alone means prepare locally. Worktree and branch ownership belong to `$solve-issues`.

Use the repository's PR template exactly when present. Title the PR `<type>(<scope>): <imperative summary>` with the narrowest meaningful Conventional Commits type and scope; omit scope only when none is useful, and reserve `style` for formatting-only changes rather than CSS or behavior. Include appropriate issue-closing syntax. Do not invent sections or add generic local command output unless the template requests it.

For user-facing changes, include clearly labeled `Before:` and `After:` states under the best template heading. Use captured media when available; when only after media exists, describe the prior state in text. Without a template, write a concise `## Description` and add `## Comparison` with the same labels only for user-facing changes.

Prepare: ignore `DRAFT.md`, `REVIEW.md`, and `.codex-pr-media/`, preferring `.git/info/exclude`, then write the title, body, and local media references to `DRAFT.md`. Do not push or contact GitHub. Return `{ state: "prepared", draftPath, branch, headSha, media }`.

Publish: require explicit publish permission from the current user request, a user-owned repo, a prepared `DRAFT.md`, passing local checks, and review approval for the current head. Use the GitHub Yeet skill to push and open a real draft PR. Upload media through the in-app GitHub editor, then verify the saved body contains GitHub attachment URLs and no local paths. If upload is unavailable, keep local media paths only in the result. Never mark ready, merge, request reviewers, or enable auto-merge.

Return `{ state: "published", prUrl, branch, headSha, media }` only after verifying the draft PR URL and body. The caller owns CI monitoring and final-head review.
