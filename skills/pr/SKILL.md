---
name: pr
description: "Create a maintainer-ready draft PR for the user's repo, or a gitignored external-repo draft. Uses the repository template, conventional title, issue closure, user-facing comparison media, and a structured result for the issue loop. Use when the user says \"pr\", asks to draft or create a PR, or an implementation task hands off PR creation."
---

# PR

Use the supplied mode, branch, worktree, and artifact paths. When called directly, choose internal for the user's repos and external otherwise; ask only when ownership is ambiguous. Worktree creation and branch switching belong to `$solve-issues`.

Use the repository's PR template exactly when present. Title the PR `<type>(<scope>): <imperative summary>` with the narrowest meaningful Conventional Commits type and scope; omit scope only when none is useful, and reserve `style` for formatting-only changes rather than CSS or behavior. Include appropriate issue-closing syntax. Do not invent sections or add generic local command output unless the template requests it.

For user-facing changes, include clearly labeled `Before:` and `After:` states under the best template heading. Use captured media when available; when only after media exists, describe the prior state in text. Without a template, write a concise `## Description` and add `## Comparison` with the same labels only for user-facing changes.

Internal: use the GitHub Yeet skill to publish the branch and open a real draft PR. Upload media through the in-app GitHub editor, save the description, and verify it contains GitHub attachment URLs and no local paths. If upload is unavailable, keep local media paths only in the returned result; never place them in the PR body.

External: do not open a GitHub PR. Ignore `DRAFT.md`, `REVIEW.md`, and `.codex-pr-media/`, preferring `.git/info/exclude`, then write the title, body, and local media references to `DRAFT.md`.

Return `{ mode, prUrl | draftPath, branch, headSha, media }`: internal results contain `prUrl`; external results contain `draftPath`. Do not hand off to review until every field is present and the draft body has been verified.
