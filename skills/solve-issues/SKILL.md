---
name: solve-issues
description: "Orchestrate GitHub issue processing for open issues labeled ready: auto-determine internal vs external repo mode, create two ponytail threads per issue, and call $solve-issue with an explicit task packet. Use when the user says \"solve issues\", \"building loops\", asks to process ready issues, or wants issue-to-PR automation."
---

# Solve Issues

Auto-determine mode once per repo: internal for the user's own repos; external otherwise. Ask only if ownership is ambiguous.

For each open issue labeled `ready`:

1. Fetch one issue with `state:open` and `label:ready`; determine the mode.
2. Create two ponytail threads: implementation and review.
3. Call `$solve-issue` with a task packet:
   - repo, issue number, issue URL, mode
   - implementation thread ID, review thread ID
   - branch/worktree path if already created
   - artifact paths: `DRAFT.md`, `REVIEW.md`, `.codex-pr-media/`
   - review pass marker: `PASS`
   - stop protocol: send the implementation thread a stop message after pass
4. Move to the next ready issue only after that loop stops.

Do not implement inside this skill; it only schedules isolated per-issue loops.
