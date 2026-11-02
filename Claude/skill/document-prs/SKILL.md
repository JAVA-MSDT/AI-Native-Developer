***

name: document-prs
description: >
Appends a just-finished pull request / merge request to a running PR log file (a user-provided
markdown file), in that file's existing format — dated period sections (calendar months by default, or
a custom start day of the month), each entry a Title/Link/Description block (max 4 short, plain-English
bullets, each ≤8 words, no technical/class/API names).
Triggers on "document this MR", "document this PR", "log this MR/PR", "add this to the PR log",
"document the MR I just created", "/document-prs".
--------------------------------------------------

# Document PRs / MRs

Appends one (or more) finished pull requests / merge requests to a running log. This file is
append-only history — never rewrite or reformat existing entries, and never touch the `## Template`
section at the very end (it must stay last, empty, and intact for future entries).

## Configuration placeholders

Ask the user for these the first time they're needed, then reuse them for the rest of the session:

* `<PR_LOG_FILE>` — full path of the markdown log file. Never assume one.
* `<PERIOD_START_DAY>` — optional. The day of the month on which each reporting period starts.
  Default: **calendar months** (1st to last day). Set it to e.g. `18` for cycles that run from the
  18th of one month to the 18th of the next.

## Step 0 — First run: make sure the log file exists

If `<PR_LOG_FILE>` doesn't exist yet, create it (and its folder if needed) with this content, then
continue:

```markdown
# PR Log

## Template

### Title: [TICKETNUMBER] Title text

- Link: PR_LINK

- Description: 
  - Bullet one
  - Bullet two
```

`## Template` always stays the last section; new dated sections are inserted before it.

## Step 1 — Gather the entry's fields

Need per PR/MR: **title**, **link**, up to **4 short description bullets**, and — only if the work
has one — a **ticket/work-item key**.

Resolve from cheapest/most reliable source first, only asking the user for what's genuinely missing:

1. **From the conversation** — if a PR/MR was just created or discussed in this session, its title,
   link, and ticket are already known. Reuse them verbatim; don't re-derive.
2. **From git** — `git log` on the current branch and the branch name itself usually carry the ticket key
   (e.g. `[A-Z]+-\d+` or `#123`) and a summary. If no ticket key exists (chore/infra work with no
   ticket), **omit the ticket prefix** entirely: write `### Title: Title text`, not a placeholder key.
3. **From the git host** — if the link is still unknown, use whatever git-host tooling is available
   (an integration, the `gh`/`glab` CLI, an API) to find the PR/MR for the current branch.
4. **Ask the user** — only for whatever steps 1–3 couldn't resolve (most often: which of several recent
   commits/changes to summarize, or confirming the ticket when it's ambiguous).

For the description bullets: write them yourself from the actual diff/commit messages/conversation —
short, past-tense, one line each, **maximum 4**, most important change first. Do not pad to 4 if
there's less to say — omit unused bullets entirely, never leave a blank bullet.

Each bullet is **plain English, non-technical, and at most 8 words**. This is documentation for a
general reader, not a commit message or a code review note:

* No class names, method names, API/endpoint paths, field names, config keys, or other identifiers a
  non-engineer wouldn't recognize.
* No code snippets, file paths, or technical jargon (HTTP verbs, status codes, annotations, protocol
  names, etc.).
* Describe the *effect* or *outcome* in everyday words — what changed for the product/user/process — not
  the mechanism. E.g. "Fixed a bug where order totals sometimes failed to load" rather than "Removed
  duplicate parameter mapping in the gateway integration."
* 8 words is a hard cap per bullet, not a target — trim rather than exceed it.

## Step 2 — Determine the target period section

Sections are headed `## DD.MM.YYYY - DD.MM.YYYY` (2-digit day/month, 4-digit year, matching the file's
existing headers exactly). Use the PR's merge/creation date if the user is back-filling an older PR;
otherwise use today's date.

**Calendar months (default, no `<PERIOD_START_DAY>`):** the section runs from the 1st to the last day
of the date's month, e.g. `01.03.2026 - 31.03.2026`.

**Custom start day `N`:** the section runs from day `N` of one month to day `N` of the next:

* If the date's day-of-month is **≥ N**: `N.<this month>.<this year> - N.<next month>.<year>`.
* If the date's day-of-month is **< N**: `N.<previous month>.<year> - N.<this month>.<this year>`.

(e.g. with `N = 18`: `18.08.2026 - 18.09.2026`.) Roll the year over correctly across December/January.

If the file already has section headers in a different date format, match that format instead.

## Step 3 — Insert the entry

Read the file, then:

* If a section with that exact header already exists, append the new entry as the **last** entry in that
  section (right before the next `## ` header, or before `## Template` if it's the last section so far).
* If no section with that header exists yet, create a new dated section — placed immediately before the
  `## Template` section, after every existing dated section — and add the entry under it.

Entry format — match this exactly, including the blank line after `Description:` and 2-space bullet
indent (use the format of the most recent entries; don't copy older inconsistent spacing if the file has
any):

```markdown
### Title: [TICKETNUMBER] Title text

- Link: PR_LINK

- Description: 
  - Bullet one
  - Bullet two
  - Bullet three
  - Bullet four
```

For an entry with no ticket, the title line is `### Title: Title text` (no brackets).

## Step 4 — Confirm

Show the user the exact block that was added (and the section header it landed under, especially if a new
section was just created). Keep it brief — this is a low-risk, append-only doc edit, not a code change.

## Boundaries

* Never edit or reformat any existing entry — only append new ones.
* Never touch the `## Template` section's content; only ever insert new dated sections *before* it.
* Max 4 description bullets per entry, no exceptions.
* Each description bullet: plain English, no technical/class/API/field names, **max 8 words** — no exceptions.
* No ticket? Omit the prefix — never invent a placeholder key.
* If the same PR/MR link already appears in the file, point this out and ask before adding a duplicate.
* Never assume the log file's location — use `<PR_LOG_FILE>` and ask when it's missing.
