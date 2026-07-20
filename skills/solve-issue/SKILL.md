---
name: solve-issue
description: "Run one GitHub issue loop from an explicit task packet containing issue, repo, mode, conventional branch name, visible implementation and review Codex app threads, artifact paths, pass marker, and stop protocol. Recheck that no open or draft PR already covers the issue, then implement, verify, call $pr, and wait on a low-frequency heartbeat until review passes. Use when $solve-issues hands off one open issue or when the user provides one complete task packet."
---

# Solve Issue

Input: one task packet from `$solve-issues`. Do not fetch queues, choose issues, change thread topology, or infer missing orchestration context. The task packet must name the neutral seed thread, implementation thread, review thread, shared worktree path, and branch name.

The implementation and review threads must be user-owned, visible Codex app threads supplied by the task packet, not transient subagents. They must run in the dedicated worktree supplied by the task packet. The implementation thread owns all code writes. The review thread must not change code; in external mode it may write only the supplied `REVIEW.md` path. The review thread must be a same-directory sibling of the implementation thread forked from a neutral worktree seed, never a child fork of the implementation conversation.

Do not busy-wait for review. Create or use a heartbeat automation that checks the review thread at a low-frequency cadence: start at no less than 10 minutes, read the review thread once per wakeup, and back off to 30 minutes after two unchanged checks. If automation is unavailable, manual `read_thread` fallback checks must follow the same cadence.

Implementation thread:

1. Read the issue, comments, linked PRs, and relevant code. Before creating a branch or making the first edit, query open PRs, including drafts, and stop if one already covers the issue. Count only an explicit GitHub Development/closing-issue link or a closing keyword for this issue, not an incidental plain-text mention. Return `ISSUE_SKIPPED issue=<number> reason=covered_by_pr pr=<url>` to the orchestrator, create no commits or PR, and end the loop. When resuming this loop after it has created its own PR, do not mistake that recorded PR for pre-existing coverage.
2. Validate the supplied branch name before creating or pushing it. Require `<type>/<short-kebab-description>` with the accurate conventional prefix: `fix/`, `feat/`, `docs/`, `refactor/`, `test/`, or `chore/`. Allow an optional issue number after the prefix. Never create or push `codex/`, username-prefixed, or otherwise non-conventional branches. If the packet's name is invalid, derive the corrected name, report it to the orchestrator, and use the corrected name throughout the PR result and review prompts.
3. For user-facing changes, capture "before" media in the supplied media directory before editing files whenever the current behavior can be shown.
4. Implement, commit, verify with repo checks, and capture "after" media for user-facing changes.
5. Use `$pr` with the supplied mode and artifact paths.
6. Require `$pr` to return `{ mode, prUrl | draftPath, branch, headSha, media }`. If this result is incomplete, fix `$pr` output before review starts.
7. Send the review thread a concrete review prompt with the repo, issue URL, mode, worktree path, branch, head SHA, PR URL or `DRAFT.md` path, media paths, `REVIEW.md` path, pass marker, and review permissions. Start the prompt with a context guard before any skill invocation:

```text
You are the review thread for <repo> issue #<number>.
This thread was created as a same-directory sibling from neutral seed thread <seedThreadId>; any inherited history is only worktree setup context.
If this conversation history identifies you as the implementation thread, reply exactly:
CONTEXT_MISMATCH issue=<number> expected=review
Do not edit files except <REVIEW.md path> in external mode.
```

If the review thread returns `CONTEXT_MISMATCH`, report the contaminated topology to the orchestrator and stop instead of retrying review in that thread.
8. Create or use heartbeat automation until the review thread returns the exact pass marker from the task packet. Each wakeup may perform only one `read_thread` check before either acting on new feedback or scheduling the next check.
9. For each actionable review comment, update the implementation, commit fixes, update the PR or `DRAFT.md`, send the review thread a new review prompt with the new head SHA and pass marker, and continue.
10. On the exact pass marker, send the stop/archive message from the task packet and report completion to the orchestrator.

Review thread:

1. Internal: review the draft GitHub PR and comment on it. Do not edit files.
2. External: review `DRAFT.md` and write actionable findings to the supplied gitignored `REVIEW.md` path. Do not edit any other file.
3. If changes are required, write concise actionable comments with file/line references where possible.
4. If no changes are required, return only the exact pass marker supplied by the implementation thread, e.g. `REVIEW_PASS issue=<number> head=<headSha>`.
5. On pass, send the implementation thread the stop message from the task packet.
