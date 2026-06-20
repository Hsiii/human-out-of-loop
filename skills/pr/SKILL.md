---
name: pr
description: "Draft pull request output in internal or external mode: for the user's repos, create real draft GitHub PRs using the repo template, GitHub Yeet skill, and browser media upload; for external repos, write gitignored DRAFT.md and media without opening a PR. Use when the user says \"pr\", asks to draft or create a PR, or another skill hands off PR creation."
---

# PR

Use the supplied mode. If called directly, auto-determine mode: internal for the user's own repos; external otherwise. Ask only if ownership is ambiguous.

Use the supplied branch, worktree, and artifact paths when provided. Do not create or switch worktrees here; worktree ownership belongs to `$solve-issues`.

Use the repo's template exactly when present. Make the title convey importance and maintainer value. Do not add sections the template does not ask for. Do not add generic local verification notes such as `bun run build`, `bun run lint`, or test output unless the repo template explicitly asks for them; CI should report verification. For user-facing changes, include UI comparison under the most suitable heading with explicit `Before:` and `After:` labels. If only after media exists, describe the before state in text and place the uploaded image under `After:`.

Internal: use the GitHub Yeet skill for branch publish and real draft PR creation. Do not put local filesystem paths in the PR body. If media exists, use the in-app browser GitHub editor: open the draft PR page, open `Show options` -> `Edit comment` on the PR description, paste/upload files from `.codex-pr-media/` into the description textarea, wait for GitHub to insert `https://github.com/user-attachments/...` URLs or image tags, save with `Update comment`, then verify the saved PR body contains the GitHub attachment URLs and no local paths such as `/Users/`, `/home/`, worktree paths, or `.codex-pr-media/`. Browser upload is best-effort; if the logged-in browser is unavailable, still create the draft PR and return local media paths for follow-up, but do not include those local paths in the PR body.

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

**Before:** <previous behavior/state, with uploaded media if available.>

**After:** <new behavior/state, with uploaded media if available.>
```

Omit `## Comparison` for non-user-facing changes.
