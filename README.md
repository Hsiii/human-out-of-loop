# human-out-of-loop

Codex skills for human-out-of-loop GitHub issue-building loops.

## Skills

- `solve-issues`: triggered by "solve issues" or "building loops"; fetches open issues labeled `ready`, determines internal/external mode, creates two ponytail threads per issue, and calls `$solve-issue` with a task packet.
- `solve-issue`: runs one issue only; it receives the task packet, coordinates implementation/review heartbeat, and stops on the explicit `PASS` marker.
- `pr`: triggered by "pr"; internal mode uses the repo template, Yeet, and browser media upload to create a real draft PR, while external mode writes gitignored `DRAFT.md` and `.codex-pr-media/` without opening a GitHub PR.

## Why

This makes my issue-to-PR workflow require less human-in-the-loop work.
