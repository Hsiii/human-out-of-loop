# Skills

Two small Codex skills for a review-gated GitHub issue workflow.

## Skills

- `solve-issue-loop`: asks for assignee and label, fetches one matching issue, asks for context if needed, implements, verifies, commits, then waits for user review before handing off to `$pr`.
- `pr`: creates draft PRs using the repository template, or a minimal fallback with `Description` and optional `Comparison`.
