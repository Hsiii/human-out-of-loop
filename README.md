# human-out-of-loop

Codex skills for turning completed work—or a fresh GitHub issue—into a reviewed, maintainer-ready draft PR.

## Install

Requires the Codex desktop app, GitHub access, and [Ponytail](https://github.com/DietrichGebert/ponytail).

Ask Codex:

> Install the skills from `sago-cream/human-out-of-loop`.

## What it catches

Before opening the PR, Codex checks the complete change for:

- Incorrect or incomplete behavior
- Unintended changes and scope creep
- Branch and commit hygiene
- Repository conventions and PR templates
- Clear, project-appropriate titles and descriptions
- Correct issue linking
- Matched before-and-after screenshots or videos for UI changes
- Correct base branch, head branch, commits, and target repository
- Broken PR formatting or missing media

If `$pr` finds an actionable problem, it stops before publication so you can fix and rerun it. The issue workflows handle that fix-and-review loop for you.

## Use it

| Goal | Prompt |
| --- | --- |
| Review committed work and open a draft PR | `$pr` |
| Solve issue #65 | `$solve-issue #65` |
| Solve five issues in parallel | `$solve-issue 5` |
| Solve and independently review issue #65 | `$solve-issue-review 65` |
| Solve and independently review five issues | `$solve-issue-review pick 5` |

`$pr` reviews the complete diff and publishes it to the current branch's writable remote. For an external contribution, make the branch track your fork; the skill never follows the parent repository.

`$solve-issue` starts from an open issue, implements it in an isolated branch and worktree, runs the relevant checks, fixes review findings, and invokes `$pr`.

`$solve-issue-review` adds a separate, independent reviewer before `$pr` publication. Batch runs process eligible issues in parallel.

The skills may create branches, worktrees, commits, and comparison media.

### Follow up on a PR

Reply in the original **Solve #…** or **Solve N issues** task—not the developer or reviewer task. It reuses the existing tasks, branch, worktree, and draft PR instead of duplicating the run.

> PR #123 has an issue with the empty state. Fix it, review the new commit, and update the draft PR.

## UI comparisons

For UI changes, `$pr` adds matched before-and-after images or videos as GitHub-owned attachments. Install the [`gh-image`](https://github.com/drogers0/gh-image) extension once to let the skill upload them directly:

```bash
gh extension install drogers0/gh-image
```

The skill requires `gh-image` v1.3.0 or later. It uses your existing GitHub CLI token when possible and may fall back to your local GitHub browser session. Images render from returned Markdown; videos render inline from the returned GitHub attachment URL. If the extension is unavailable, the skill uses the GitHub editor instead.

This is not backward-compatible with releases that used `npx sago-media`: the Sago Media login and CLI are no longer used for PR attachments.

## Automation

Ask Codex to create an automation that runs `$solve-issue [amount]` for each Git repository in a workspace, then choose the schedule and workspace folder.
