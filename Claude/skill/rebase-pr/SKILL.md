***

name: rebase-pr
description: >
Rebases an open pull request's / merge request's source branch onto its target branch when the git host
blocks the merge because the branch is out of date — e.g. GitLab's "Fast forward merge is not possible.
Please rebase." or "The source branch is N commits behind the target branch.", or GitHub's "This branch
is out-of-date with the base branch". Snapshots current approvals first, rebases locally (resolving any
conflicts by actually understanding both sides — never blind ours/theirs), asks for confirmation, then
force-pushes with --force-with-lease and reports whether approvals survived and who needs to
re-approve if not.
Triggers on "/rebase-pr \<PR/MR link> \[commits behind]", and on requests shaped like "I would like to
merge this PR/MR <link> but can't because it needs a rebase / the branch is N commits behind — can you
rebase and preserve approvals".
-------------------------------

# Rebase a PR / MR

Args: `<PR_LINK> [COMMITS_BEHIND]`. `PR_LINK` (a pull request or merge request URL) is required.
`COMMITS_BEHIND` is optional — if omitted, compute it yourself in Step 3 rather than asking the user.

Goal: get the PR/MR to a state where the host's up-to-date/fast-forward check passes, with zero
unintended conflict resolutions, and with the human able to see exactly what happened to the approvals
before it's pushed.

The steps below name *what* to look up, not a specific tool. Use whatever tooling the environment
provides for the PR's host (a GitLab or GitHub integration, the `glab`/`gh` CLI, an API client, etc.).
**If no host tooling is available**, you can still do Steps 2–5 purely with local git; in that case ask
the user to read the PR's source/target branches and its current approvers from the web UI and give them
to you, and ask them to check the approvals again themselves after the push (Step 6).

Throughout, `origin` means the remote that hosts the PR — confirm the remote name with `git remote -v`
if it isn't `origin`.

## Step 1 — Resolve the PR and record its current state

Parse the project path and PR/MR number out of the link. Fetch:

* The PR's **source branch**, **target branch**, title, and current mergeability/conflict flags.
* The **current approvals** — who has approved right now, before anything is touched. Keep this list —
  Step 6 needs it.

If the project has a "reset/dismiss approvals on new push" setting you can read (e.g. GitLab's project
approval settings, GitHub's "dismiss stale pull request approvals" branch-protection rule), check it now
so you know in advance whether preservation is even possible; if you can't determine it, say so and fall
back to the before/after comparison in Step 6.

## Step 2 — Get the repo into a clean, correct local state

Determine whether the PR's project is already checked out in the current working directory (compare
remote URL / project path). If not, ask the user for the local path rather than guessing or cloning
somewhere new.

Run `git status`. If there are uncommitted changes, stop and ask the user how to handle them (stash,
commit, or abort) — never stash or discard silently. Then:

```
git fetch origin <source_branch> <target_branch>
git checkout <source_branch>   # create local tracking branch if it doesn't exist
git reset --hard origin/<source_branch>   # only if source_branch already existed locally and is behind its own remote
```

## Step 3 — Confirm how far behind it is

If `COMMITS_BEHIND` was given, use it only as a sanity check. Compute the real number yourself:

```
git rev-list --count origin/<source_branch>..origin/<target_branch>
```

Mention the actual count to the user if it differs meaningfully from what they guessed.

## Step 4 — Rebase

```
git rebase origin/<target_branch>
```

If it goes clean, skip to Step 5.

If there are conflicts: resolve each one by actually reading both sides — the incoming target-branch
change and the source branch's own change — and combine their intent. Do not resolve mechanically with
`--ours`/`--theirs` and do not delete the other side's change just to make the marker disappear. After
each file: `git add <file>`, then `git rebase --continue`. If a conflict is genuinely ambiguous (e.g. both
sides rewrote the same logic differently), stop and ask the user rather than guessing.

After the rebase completes, build/lint/test if the repo has a fast local check available, so a conflict
resolution mistake surfaces before it's pushed rather than after.

## Step 5 — Confirm before pushing

Force-pushing rewrites the remote branch's history and is hard to reverse — always pause here. Show the
user: the new commit count/SHA, a one-line summary of any conflicts you resolved and how, and the
approval snapshot from Step 1. Ask for explicit go-ahead before pushing.

Before pushing, re-fetch `origin/<source_branch>` once more and confirm its SHA still matches what you
started from — if someone else pushed to the branch while you were working, stop and re-rebase on top of
that instead of overwriting it.

```
git push --force-with-lease origin <source_branch>
```

Never use plain `--force`.

## Step 6 — Report the approval outcome

Re-fetch the PR's approval state and diff it against the Step 1 snapshot (or, with no host tooling, ask
the user to check the approvals in the web UI and tell you what they see):

* **Preserved** — say so plainly.
* **Reset** — the host did this itself (a project/branch-protection setting, not something the rebase
  controlled); list exactly who needs to re-approve and, if useful, offer to ping them or leave a
  comment on the PR — but don't do either without asking.

Finish by confirming the PR is now mergeable (fetch its status again) and reporting pipeline/check
status if one kicked off. Don't merge it yourself unless the user separately asks for that.
