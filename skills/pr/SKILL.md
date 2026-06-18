---
name: pr
description: "Draft pull requests using the repository's PR template when present, or a concise fallback with Description and optional Comparison. Use when the user says \"pr\", asks to draft or create a PR, or another skill hands off draft PR creation."
---

# PR

Create a draft PR. Use the repo's template exactly when present; do not add sections the template does not ask for.

If there is no template, use:

```markdown
## Description

<Concise summary of the issue and implemented change. Include issue closure syntax when appropriate.>

## Comparison

<Before/after screenshot links, or video links if the change is only visible in video or motion.>
```

Omit `## Comparison` for non-user-facing changes.
