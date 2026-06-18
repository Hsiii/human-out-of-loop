---
name: solve-issue-loop
description: "Run the user's \"solve issue\" workflow for GitHub issues: ask for assignee and label, fetch one matching issue, clarify only when needed, implement the fix, wait for user review approval, then hand off draft PR creation to $pr. Use when the user says \"solve issue\", asks Codex to solve a labeled issue for an assignee, or wants an issue-to-review loop."
---

# Solve Issue Loop

Ask for missing assignee and label, then loop:

1. Fetch one open GitHub issue assigned to `<assignee>` and labeled `<label>`; infer the repo from the workspace remote when possible.
2. Read the issue, comments, linked PRs, and relevant code.
3. If unclear, ask focused questions and wait.
4. Implement the smallest coherent fix, verify with repo checks, and capture before/after media for user-facing changes.
5. Commit with a conventional commit, using patch staging around unrelated edits.
6. Ask the user to review and wait.
7. On approval, use `$pr` to create a draft PR. If changes are requested, address them, commit, and ask for review again.
8. After the draft PR, ask whether to continue before fetching another issue.
