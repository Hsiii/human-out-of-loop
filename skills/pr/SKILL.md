---
name: pr
description: "Draft pull request output in internal or external mode: for the user's repos, create real draft GitHub PRs using the repo template, GitHub Yeet skill, and browser media upload; for external repos, write gitignored DRAFT.md and media without opening a PR. Use when the user says \"pr\", asks to draft or create a PR, or another skill hands off PR creation."
---

# PR

Use the supplied mode. If called directly, auto-determine mode: internal for the user's own repos; external otherwise. Ask only if ownership is ambiguous.

Use the supplied branch, worktree, and artifact paths when provided. Do not create or switch worktrees here; worktree ownership belongs to `$solve-issues`.

Use the repo's template exactly when present. Make the title convey importance and maintainer value. Do not add sections the template does not ask for. For user-facing changes, include UI comparison under the most suitable heading.

Internal: use the GitHub Yeet skill for branch publish and real draft PR creation. If media exists, use the logged-in browser to edit the PR body or add a PR comment, paste/upload files from `.codex-pr-media/`, and keep the inserted GitHub media URLs in the PR. Browser upload is best-effort; if the logged-in browser is unavailable, still create the draft PR and return local media paths for follow-up.

External: do not open a GitHub PR. Ensure `DRAFT.md`, `REVIEW.md`, and `.codex-pr-media/` are ignored, then write the PR title/body and media references to `DRAFT.md`. Prefer `.git/info/exclude` for these workflow artifacts; patch `.gitignore` only when that repository-level ignore change should be committed.

Return a structured result to the caller:

```json
{ "mode": "internal", "prUrl": "<url>", "branch": "<branch>", "headSha": "<sha>", "media": ["<url-or-path>"] }
```

or:

```json
{ "mode": "external", "draftPath": "DRAFT.md", "branch": "<branch>", "headSha": "<sha>", "media": ["<path>"] }
```

When there is no template, use:

```markdown
## Description

<Concise summary of the issue and implemented change. Include issue closure syntax when appropriate.>

## Comparison

<Before/after screenshot links, or video links if the change is only visible in video or motion.>
```

Omit `## Comparison` for non-user-facing changes.
