# human-out-of-loop

Codex skills that turn GitHub issues and committed changes into reviewed draft PRs, so you can go touch some grass.

## Install

Requires the Codex desktop app, GitHub access, and [Ponytail](https://github.com/DietrichGebert/ponytail).

Ask Codex:

> Install the skills from `Hsiii/human-out-of-loop`.

## Use

| Goal | Prompt |
| --- | --- |
| Create a draft PR | `$pr` |
| Solve issue #65 | `$solve-issue #65` |
| Solve five issues in parallel | `$solve-issue 5` |
| Solve and independently review issue #65 | `$solve-issue-review 65` |
| Solve and independently review five issues | `$solve-issue-review pick 5` |

## What to expect

`$pr` reviews committed changes and publishes a draft PR in the current repository. For an external contribution, run it from your writable fork; it never follows or publishes to another repository.

`$solve-issue` gives each issue one task that implements it and runs `$pr`.

`$solve-issue-review` keeps the full workflow: separate developer and reviewer
tasks in isolated branches and worktrees, with `$pr` publication after review.

The skills may create branches, worktrees, commits, and comparison media. They never merge or mark a PR ready.

### Follow up on a PR

If a produced PR needs changes, reply in the original **Solve #…** or
**Solve N issues** task—not the developer or reviewer task. The original task
routes the change through the existing developer and reviewer, then updates the
same draft PR.

For example:

> PR #123 has an issue with the empty state. Fix it, review the new commit, and
> update the draft PR.

## Optional PR media

The PR skill can publish images and videos through a compatible self-hosted
media service. Ask its operator for a two-line configuration, save it at
`~/.config/pr-media/config`, and make it readable only by your user:

```text
url=https://media.example.com
token=replace-with-your-token
```

```bash
chmod 600 ~/.config/pr-media/config
```

Without that configuration, the skill uses GitHub's media uploader.

## Automation

Ask Codex to create an automation that runs `$solve-issue [amount]` for each Git repository in a workspace, then choose the schedule and workspace folder.

I use [Hsiii/fish-alias](https://github.com/Hsiii/fish-alias) to collect suitable ongoing projects into one directory with symlinks.
