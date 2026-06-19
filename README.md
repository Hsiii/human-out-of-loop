# Skills

Three small Codex skills for GitHub issue-building loops.

## Skills

- `solve-issue`: triggered by "solve issue" or "building loops"; fetches open issues labeled `ready`, determines internal/external mode, creates two ponytail threads per issue, and calls `$solve-issue-loop` with a task packet.
- `solve-issue-loop`: runs one issue only; it receives the task packet, coordinates implementation/review heartbeat, and stops on the explicit `PASS` marker.
- `pr`: triggered by "pr"; internal mode uses the repo template and Yeet to create a real draft PR, while external mode writes gitignored `DRAFT.md` and `.codex-pr-media/` without opening a GitHub PR.

## Why

The orchestrator owns issue queues. The loop skill owns only one two-thread interaction. The PR skill owns maintainer-focused PR output, so internal repos can publish draft PRs while external repos can prepare reviewable drafts safely.
