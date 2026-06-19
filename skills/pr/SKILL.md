---
name: pr
description: "Draft pull request output in internal or external mode: for the user's repos, create real draft GitHub PRs using the repo template and the GitHub Yeet skill; for external repos, write gitignored DRAFT.md and media without opening a PR. Use when the user says \"pr\", asks to draft or create a PR, or another skill hands off PR creation."
---

# PR

Use the supplied mode. If called directly, auto-determine mode: internal for the user's own repos; external otherwise. Ask only if ownership is ambiguous.

Use the repo's template exactly when present. Make the title convey importance and what the PR helps maintainers with. Do not add sections the template does not ask for. For user-facing changes, include UI comparison under the most suitable template heading.

Internal: use the GitHub Yeet skill for branch publish and real draft PR creation.

External: do not open a GitHub PR. Ensure `DRAFT.md`, `REVIEW.md`, and `.codex-pr-media/` are gitignored, then write the PR title/body and media references to `DRAFT.md`.

When there is no template, use:

```markdown
## Description

<Concise summary of the issue and implemented change. Include issue closure syntax when appropriate.>

## Comparison

<Before/after screenshot links, or video links if the change is only visible in video or motion.>
```

Omit `## Comparison` for non-user-facing changes.
