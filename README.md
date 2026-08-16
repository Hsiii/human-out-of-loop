# human-out-of-loop

Codex skills for turning completed work—or a fresh GitHub issue—into a reviewed, maintainer-ready draft PR.

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

If the review finds an actionable problem, the PR is not published until it is fixed and reviewed again.

## Use it

Already finished the work and committed it:

> `$pr`

Starting from a GitHub issue:

> `$solve-issue #65`

Codex implements the issue, runs the relevant checks, reviews the complete result, fixes any findings, and opens the draft PR.

For a separate developer and reviewer:

> `$solve-issue-review 65`

You can also process multiple eligible issues in parallel:

> `$solve-issue 5`

## Install

Requires the Codex desktop app, GitHub access, and [Ponytail](https://github.com/DietrichGebert/ponytail).

Ask Codex:

> Install the skills from `orangesago/human-out-of-loop`.
