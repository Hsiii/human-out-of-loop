---
name: pr
description: "Review the current committed changes and create a maintainer-ready draft PR for the user's repository or a local DRAFT.md for an external repository. Use when the user invokes \"$pr\" or asks to draft, create, or open a PR for the current branch."
---

# PR

1. Use the current repository, worktree, and branch. Derive the base from the remote default branch and choose internal for the user's repos or external otherwise; ask only when ownership is ambiguous.
2. Inspect the complete base-to-head diff, commits, linked issue, repository instructions, and working-tree state. Require intended changes to be committed; never stage or commit unrelated changes.
3. Run relevant checks. For a user-facing change, ignore `.codex-pr-media/` through `.git/info/exclude` before capturing comparison media.
4. Review the complete base-to-head diff read-only. On findings, report them and stop before drafting or publication.
5. Use the repository's PR template exactly when present. Title the PR `<type>(<scope>): <imperative summary>` with the narrowest Conventional Commits type; omit scope only when none is useful and reserve `style` for formatting-only changes. Include issue-closing syntax. Do not invent sections or generic command output.
6. For user-facing changes, include `Before:` and `After:` under the best template heading. Use captured media when available; when only after media exists, describe the prior state. Without a template, use concise `## Description` and optional `## Comparison` sections.
7. External: write `DRAFT.md`, ignore it through `.git/info/exclude`, report its path and head, and stop without GitHub writes.
8. Internal: do not create `DRAFT.md`. Push and open a real draft PR, upload media in the GitHub editor, and verify the saved body contains GitHub attachment URLs and no local paths. Inspect checks once; report the verified URL, head, and check state without polling pending checks unless asked.

Never mark ready, merge, request reviewers, enable auto-merge, or post a GitHub review, review comment, reaction, or approval.
