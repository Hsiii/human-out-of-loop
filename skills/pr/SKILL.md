---
name: pr
description: "Review committed changes and publish a maintainer-ready draft PR in the current branch's writable remote repository. Use when the user invokes \"$pr\" or asks to draft, create, or open a PR."
---

## Preflight

1. Treat the current branch's Git remote as the sole publication target. Never infer a repository from GitHub CLI context, follow a parent/upstream repository, create or synchronize a fork, or publish anywhere else. Require authenticated write access; otherwise stop and ask the user to run from a branch whose remote is their writable fork or repository.
2. Run the bundled `bash scripts/pr-preflight` from the target worktree before review and again before publication. Never review or publish while blocked.
3. Inspect its prospective commits and complete diff for task scope.
4. If the branch is default or spent, rebuild from the reported base with only intended commits, then rerun preflight.
5. If base or head changes, repeat the affected review.

## Review

1. Use the preflight base and head; verify worktree HEAD before each review.
2. Review the complete `base...head` diff. Report actionable findings and stop.
3. For every UI change, reuse supplied media or capture matched, reproducible before/after media from base and head. Capture `Before` from the exact base commit the PR will merge into and `After` from the reviewed head. On follow-ups, reuse or recapture `Before` from that base; never use an earlier feature-branch revision or a prior `After` as the new `Before`:
   - Use video for interaction, motion, or multiple steps; images otherwise.
   - Match viewport, state, data, and action sequence.
   - Frame screenshots with enough surrounding UI to make the changed element's location and purpose clear, including a page or section landmark and relevant adjacent controls. Prefer viewport- or section-level framing; use tight element crops only when context is genuinely unnecessary.
   - Keep media untracked through `.git/info/exclude`.
   - Never publish with either side missing.

## Package

- Use the repository PR template exactly when present.
- Before writing the title and description, inspect recently merged PRs and Git history for the repository's vocabulary, scope, and level of detail.
- Title: `<type>(<scope>): <imperative summary>` using the narrowest Conventional Commits type. Omit an unhelpful scope; reserve `style` for formatting-only changes.
- Prefer a concise, human-readable title that describes the meaningful effect of the change, not merely the implementation mechanism.
- Add issue-closing syntax when the change resolves an issue.
- Do not invent template sections or include generic command output.
- Open the description with a short prose explanation of the problem or user need and how the change addresses it. Follow with implementation bullets when they help reviewers; do not lead with an implementation inventory.
- Put both UI `Before:` and `After:` media under the best template heading; never publish a comparison with either side missing. Without a template, use concise `## Description` and optional `## Comparison`.

## Publish

Immediately before publication, verify worktree HEAD equals the reviewed head. If not, restart preflight.

For UI changes, run the bundled `bash scripts/pr-media-upload --available`. When it succeeds, publish each file with `bash scripts/pr-media-upload --repo OWNER/REPO --pr NUMBER <file>`; otherwise use the GitHub editor. Use the returned Markdown exactly as emitted: short videos may already include a generated GIF preview above the full-video link, so do not create or upload a second preview. Replace only local media references with the returned Markdown or attachment URLs, and verify no local paths remain.

Push to the exact `remote` reported by preflight. Create or update the branch's draft PR against the reported `repo` and `base_branch`, passing `--repo OWNER/REPO` explicitly to every GitHub CLI command; never rely on its inferred repository. Stop without modification if the existing PR is not a draft. Before changing an existing PR body, fetch and snapshot it. Preserve any user-authored or provenance-unknown content; replace a body only when it is empty or exactly matches content written earlier in the current run. Re-fetch immediately before writing and compare it byte-for-byte with the snapshot; if it changed, stop without modifying it and ask the user how to integrate the proposed update. After saving the PR, re-fetch its body and verify that the Markdown renders correctly, including real newlines, headings, lists, tables, and media. Verify the saved PR's base and head repositories both equal the reported `repo`, and its base, head, and commits match the reviewed scope.

Only create or update drafts. Never mark ready, merge, request reviewers, enable auto-merge, or post GitHub review activity.
