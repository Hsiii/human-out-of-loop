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

`$pr` reviews committed changes and publishes a draft PR in the current branch's writable remote repository. For an external contribution, make the branch track your fork remote; `$pr` pins every GitHub command to that fork and never follows the parent/upstream repository.

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

For UI changes, `$pr` adds matched before-and-after images or videos. Without a
configured media service, Codex falls back to GitHub's editor and uses Computer
Use to attach each file. That works, but takes more time and tokens.

The optional media service lets Codex upload files directly and insert their
Markdown into the PR body. Authenticate once through the shared media CLI:

```bash
npx sago-media auth login
```

The browser flow identifies you with GitHub and queues the device for the
service owner to approve. The PR skill invokes a pinned version of the
same CLI and never handles upload tokens itself.

When supported by the configured service, uploading a short video returns a GIF
preview for inline review followed by a link to the full-quality recording. The
skill uses that response directly and does not generate a duplicate preview on
the client.

Media hosted by this service is temporary and may expire. To preserve it on a
long-lived PR, open the draft before it expires, copy the rendered image from
the PR body, edit the body on GitHub, and paste it back into the editor. GitHub
will upload its own copy; replace the temporary Markdown with the new GitHub
attachment. For video, download the file and upload it through the editor
instead.

## Automation

Ask Codex to create an automation that runs `$solve-issue [amount]` for each Git repository in a workspace, then choose the schedule and workspace folder.

I use [Hsiii/fish-alias](https://github.com/Hsiii/fish-alias) to collect suitable ongoing projects into one directory with symlinks.
