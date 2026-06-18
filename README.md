# Solve Issue + PR Skills

Two small Codex skills for a review-gated GitHub issue workflow.

## Skills

- `solve-issue-loop`: triggered by "solve issue"; asks for assignee and label, fetches one matching issue, clarifies if needed, implements, verifies, commits, then waits for user review before handing off to `$pr`.
- `pr`: triggered by "pr"; creates draft PRs using the repository template, or a minimal fallback with `Description` and optional `Comparison`.

## Why

The issue-solving loop and PR-writing rules are separate so either can be reused independently. Issue work stays focused on implementation and review approval, while PR creation consistently follows the repo template without adding random extra sections.
