***

name: demo-guide
description: Generates a polished, self-contained HTML demo/walkthrough guide for a sprint or period demo — commit history and issue-tracker work items across one or more repos (local checkout or remote git host, any mix), for a given date range, turned into a numbered "Recommended Demo Path" that walks the real user journey (or API/CLI flow) with per-step ticket badges, plus backend-only/non-visual callouts and a full commit-log appendix. Reuses the exact visual style/structure of the reference example bundled with this skill. Saves the result to a user-provided output folder. Invoke on "/demo-guide", "generate the demo guide", "build a demo guide for this sprint", "create the HTML demo guide", "make me a demo walkthrough".
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Demo Guide Generator

Produces the same kind of artifact as `assets/example-demo-guide.html` (read that file now if
you haven't already — it's the literal style and structure reference, not just inspiration) for
whatever repos/date-range/project the user names this time. The whole point is a presenter can
follow it top to bottom in a live demo without re-deriving anything or getting caught by a claim
that turns out not to hold up when clicked on live.

The example describes a fictional product ("Task Tracker"). None of its names, tickets, people,
URLs, or test data are real — replace all of it with the actual project's content.

## Configuration placeholders

* `<DEMO_OUTPUT_DIR>` — the folder where finished guides are saved. Never assume one; ask the user
  for it if it hasn't been provided.

## Arguments

`{args}` may be empty. It may contain the date range, repo paths, and/or project name inline (e.g.
`2026-03-02 to 2026-03-09 BE:C:\repos\foo FE:C:\repos\foo-ui`). Parse whatever's given; ask for
whatever's missing (Step 1).

## Step 1 — Collect inputs

Required, ask if not supplied and not inferable from context:

1. **Date range** — from/to. Accept natural language ("last week", "since Monday") and convert to
   absolute dates before using them anywhere.
2. **Repos** — one or more, each needs:
   * A **path or URL**. A local filesystem path means a local checkout is available; a git-host URL
     (or a name you can resolve via whatever git-host tooling is available) with no local path means
     remote-only.
   * A **role label** (BE, FE, API, CLI, or whatever the user calls it — a repo can have more than
     one, e.g. a monorepo isn't unusual).
   * A **branch** to read commits from — resolve the repo's default branch rather than assuming
     `main`/`develop`; ask if it's unclear.
3. **Live environment URL**, if one exists — used for the hero "Open live demo environment" button
   and as the base for every step's URL. Optional; if absent, drop the button and use relative/repo
   paths in the steps instead.

Nice-to-have, don't block on these — ask only if a later step needs one and it's not obvious:

* An account/tenant/workspace ID or other path segment needed to reach the flow (e.g. the
  `acme` in `/workspaces/acme/tasks`).
* Known-good test data (accounts, names, amounts, etc.) for reaching specific screens or triggering
  specific branches (error vs. success, role-specific views, etc.). **Ask the user or check the
  project's test fixtures for this — never invent plausible-looking test data.** Fabricated test
  data silently breaks the demo the first time someone actually types it in.
* Output location override — default is `<DEMO_OUTPUT_DIR>` (Step 8).

## Step 2 — Gather commits per repo

For each repo, get every commit on the target branch within the date range, oldest metadata
preserved (SHA, date, author, subject) — this is the raw material for both the demo path and the
appendix.

* **Local checkout**: `git fetch origin <branch>`, then
  `git log origin/<branch> --since="<from> 00:00:00" --until="<to> 23:59:59" --date=format:"%Y-%m-%d %H:%M" --pretty=format:"%h|%ad|%an|%s"`.
  Also run `git status` first — never assume the checkout is clean or up to date, and never discard
  anything uncommitted you find sitting there.
* **Remote-only** (no local path given): use whatever git-host tooling is available (GitLab, GitHub,
  etc. — list commits for a ref within a time range). If no such access is configured, say so and ask
  the user for a local path instead of guessing at history.
* A commit's **author date** can predate its **commit date** (rebases, cherry-picks, squash-merges).
  Git's `--since/--until` filters on commit date by default — trust that, and if a commit's position
  in the list looks surprising, check both dates with `git show -s --format="%ad / %cd" --date=iso <sha>` before treating it as an anomaly.

## Step 3 — Classify commits: signal vs. noise

Every commit goes in the appendix (Step 6e). Only **signal** commits earn a spot in the Demo Path or
the headline stats. A commit is noise if it matches any of:

* CI/automation: "Automated version update", `[ci-skip]`, bot/pipeline author.
* Placeholder ticket keys with no real number tracking the change (`XXX-000`-style, generic project
  prefixes used as a catch-all rather than a real ticket).
* Pure chore/doc/refactor with no user-visible or API-contract change (rename, dependency bump,
  comment cleanup) — read the diff summary if the subject alone doesn't make this clear.
* Commits belonging to a **different project or epic** sharing the same repo (e.g. an unrelated
  initiative living in the same UI repo as the main product). These aren't noise exactly — they're
  real work — but they don't belong in the main Demo Path. Route them to Step 6d instead.

Extract ticket keys with a general `[A-Z]{2,}-\d+` pattern per commit subject (or the tracker's own
key format, e.g. `#123`); a commit can carry more than one key.

## Step 4 — Resolve ticket metadata

For each distinct real ticket key, pull `summary`, `status`, `issuetype` (and `description` only
when you actually need it to write an accurate step) from the issue tracker, if one is available.
**Batch in small groups (4-5 keys per call) and request only the fields you need** — batching many
tickets with large description fields in one call can silently truncate results; smaller batches
come back complete. If a key doesn't resolve, say so in the guide rather than inventing a summary
for it. If no tracker is available, build the guide from commit messages and say that ticket
metadata wasn't verified.

## Step 5 — Build the Recommended Demo Path (the part that must not be wrong)

This is the section a presenter actually reads from live, so unverified claims here are the failure
mode that matters most. Before writing any step that asserts "this proves ticket X" or "this should
now do Y":

1. **Confirm the commit actually touches the code you're describing.** Run
   `git show --stat <sha>` and cross-check the files against the claim. A ticket's description
   can list several acceptance criteria where only some were actually built in the commit tied to
   this period's range — don't credit the commit with everything the ticket description says, only
   with what the diff shows.
2. **Trace *where and when* the behavior actually executes**, not just that the code exists.
   A validation function existing in the codebase doesn't mean it fires at the UI step that would
   intuitively trigger it — e.g. a per-step form page may make no backend call at all, with the real
   check only firing once, later, on final submit. Read the calling code (the component/controller
   that invokes it, and what triggers that call) before writing "do X here and it'll be blocked."
   Getting this wrong produces a demo step that fails live in front of the audience for a reason
   that has nothing to do with whether the feature actually works.
3. **Don't assume the default branch is what's deployed to the live URL.** If the user cares whether
   something is actually live at the environment URL from Step 1, check whether the org uses a
   separate long-lived deploy branch distinct from the one you read commits from, and compare the
   commit's ancestry against *that* branch, not just the branch you happened to read history from.
4. **Order steps by the real user journey, not commit chronology.** Find the actual routes (frontend
   router config, backend controller mappings, CLI command definitions) so each step's URL or
   command is real and in the order a user would actually hit them — don't infer the flow order from
   when tickets happened to ship.
5. If a step needs specific test data to reach a particular branch (an edge case, a specific state,
   an error vs. success outcome), name the exact values inline in that step, and add a dedicated
   "Test data" card near the top of the Demo Path section if more than one run/scenario is needed
   (see the example's Run A / Run B pattern).
6. If something you're inclined to claim turns out — on verification — to be **not new this
   period**, already broken, or not wired up the way the ticket implies, **say so plainly in a
   `callout note`** rather than quietly dropping the step or, worse, asserting it anyway. A guide
   that corrects itself in front of the user is more useful than one that's silently wrong in front
   of an audience.

Each step gets: a title with role badge(s) (`b-fe` / `b-be` / `b-both` — relabel these to match the
project's own roles, e.g. API / CLI / Web) and one `.tix` chip per ticket key that step actually
demonstrates (use `.tix.muted` with text like "no ticket this week" for pure-continuity steps that
exist only to keep the flow moving); the URL; 2-5 talking points as plain click/say instructions;
and a `callout` (amber, for something to watch out for) or `callout.note` (gray, for a correction or
aside) where warranted.

**Adapting to non-web projects.** The example shows a web UI, but the structure works for any
project. For an API, steps can be requests (method + path + example payload, with the expected
response) instead of screens and URLs; for a CLI, steps can be commands with expected output; for a
library or SDK, steps can be short code snippets and their results. Keep the same step layout, badges,
ticket chips and callouts — only the "URL" line changes to whatever the presenter actually runs
(use the `.url` class for a command or request line too). If a period has no presentable surface at
all, say so, and lean on the Backend-Only / Non-Visual section.

## Step 6 — Write the remaining sections, in this fixed order

Skip any section that has nothing to put in it — don't force an empty section just to match the
template.

**a. Stats row** — 3-5 numbers that give an at-a-glance sense of scope (screens touched, pages
shipped, backend-only fixes, tickets, whatever's most informative for this project — mirror the
example's `.stats` grid, don't invent new stat categories beyond what naturally summarizes this run).

**a2. "What We're Demoing Today" summary** — always present, right after the stats row and before
the Demo Path. A single `.card` with one flat `<ul class="talk">`, one line per ticket (or tightly
related ticket group) across the *whole* guide — main path, secondary work, and backend-only alike.
This is the presenter's 30-second cold open, read aloud verbatim to set audience expectations before
Step 1 starts, so keep each bullet to the headline outcome only (not the click-by-click detail that
belongs in the step itself), and collapse multiple backend-only items into one bullet rather than one
per ticket. Prefix each bullet with its `.tix` chip(s), same as everywhere else. No new CSS — reuses
`card`/`talk`/`tix`. Add a `<a href="#summary">Today's Summary</a>` entry at the front of the TOC
(before the numbered sections) pointing at this section's `id="summary"`.

**b. Recommended Demo Path** — from Step 5. Always present, always first among the numbered sections.

**c. Backend-Only / Non-Visual Changes to Narrate** — anything from Step 3's signal commits that
has no clickable UI surface (bug fixes, contract/schema changes, analytics-only changes,
infra/release-prep work). One `.card` per item: a `.tag` naming the ticket(s), a short `h3`, and a
sentence or two of plain explanation of what changed and why it matters even though it's not visible.

**d. Secondary Pages / Adjacent Work** — optional. If this period's commits touched non-core pages
(marketing/support pages, admin tooling) or a different project sharing the repo, give each its own
small section using `.grid2` cards. Don't merge these into the main Demo Path.

**e. Full Commit Log appendix** — one collapsible `<details>` block per repo, table columns SHA
(mono) / Date / Author / Message, **every commit from Step 2 including noise** — this is the
traceability net, not a curated list.

## Step 7 — Match the reference style exactly

Copy the `<style>` block from `assets/example-demo-guide.html` **verbatim** — same CSS variable
names, same class names (`hero`, `toc`, `stats`/`stat`, `step`/`step-num`/`step-body`/`step-title`,
`url`, `badge`/`b-fe`/`b-be`/`b-both`, `tix`/`tix.muted`, `talk`, `callout`/`callout.note`, `grid2`,
`card`, `details`/`summary`, `footer`, `pill`). Don't redesign it — the value of this skill is that
every demo guide looks and behaves the same way, so a repeat presenter never has to re-learn the
layout. Only the content changes: hero title/subtitle/meta chips, the live-env link, the TOC section
list (renumber to match however many sections this run actually has), and everything inside `<body>`.
Make sure no content from the fictional example (names, tickets, URLs, test data) survives into the
real guide.

Self-contained HTML, no external requests — same constraint as the example.

## Step 8 — Save

Save to `<DEMO_OUTPUT_DIR>/<end-date>/<ProjectName>-demo-<end-date>.html` (create the dated
subfolder if it doesn't exist). If the user hasn't given `<DEMO_OUTPUT_DIR>`, ask for it first. Use
whatever project name the user's repos most naturally suggest (e.g. the shared prefix of the repo
names, or ask if it's ambiguous). If the user gave an explicit output location in Step 1, use that
instead.

## Step 9 — Report back

State the saved path. If Step 5 surfaced any corrections (a claim that didn't hold up, a step you
had to reframe, test data you had to ask for instead of assuming), summarize those in the chat
response too — don't bury a correction only inside the HTML's callout text where the user might not
notice it before presenting.

## Don'ts

* Don't credit a ticket with behavior its commit didn't actually touch — verify with `git show --stat` first, every time.
* Don't assume a check fires where intuition suggests; trace the actual call path before writing a
  "do X here, it'll block" instruction.
* Don't assume the branch you read commit history from is the branch actually deployed to any live
  URL the user cares about — check if the org runs a separate deploy branch.
* Don't invent test data, ticket summaries, or URLs. Ask, or verify against actual route/controller
  code.
* Don't drop noise/chore commits entirely — they still belong in the full commit-log appendix, just
  not in the headline stats or Demo Path.
* Don't redesign the CSS or section structure per run — reuse the bundled example verbatim so every
  guide this skill produces looks and reads the same way.
* Don't silently correct a wrong assumption without telling the user — a callout in the file plus a
  line in your chat response, not one or the other.
* Don't leave any of the example's fictional content in a real guide.
