***

name: ticket-report
description: >
Generates two human-facing HTML reports for a work item (ticket/issue), saved to a per-ticket folder
under a user-provided output directory. Analysis mode: given a ticket key (+ optional design-doc/wiki
reference and extra context), investigates the codebase directly and writes a comprehensive
pre-implementation HTML analysis — step-by-step requirement walkthrough, codebase/risk/edge-case
findings (each as issue/fix/impact), a workflow diagram, and an incremental implementation plan —
before any code is touched. Test-guide mode, run later once implementation is done, writes a second
HTML report into the same folder: an intro, a summary of what was actually built, a diagram, a local
testing walkthrough, and one fully self-contained testing procedure per real environment (detected
from the repo's own env config). Never starts implementation or commits — pure analysis/reporting.
Invoke on "/ticket-report", "I want to work on <ticket>", "start ticket <ticket>", "kick off <ticket>", "/ticket-report test-guide", "generate the test guide for this ticket".
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Ticket Report

Produces two standalone HTML deliverables around a work item's lifecycle, both landing in a
user-chosen folder outside any repo. This skill does its own investigation and writing — it is
not a wrapper around anything else.

Two independent modes, chosen from `{args}`/the user's message — ask if genuinely ambiguous:

* **Analysis mode** (default) — starting work on a ticket, before any code changes. Steps 1-4.
* **Test-guide mode** — implementation for the ticket is already done this session. Step 5 only.

## Configuration placeholders

Ask the user for these the first time they're needed, then reuse them for the rest of the session:

* `<REPORT_OUTPUT_DIR>` — the base folder where per-ticket report folders are created. Never assume
  one.
* `<TRACKER_BASE_URL>` — (optional) the issue tracker's base URL, used only to build ticket links when
  the tracker integration doesn't return a URL itself (e.g. `<TRACKER_BASE_URL>/browse/<KEY>` for
  Jira-style trackers).

## Step 1 — Collect inputs (Analysis mode)

Ask for whatever isn't already given or inferable from the conversation:

* **Ticket key** (e.g. a Jira key) — build the ticket link from the tracker integration, or from
  `<TRACKER_BASE_URL>` if needed.
* **Design-doc / wiki reference URL(s)** (e.g. Confluence) — optional.
* **Additional context** — optional free text (constraints, related tickets, anything the user
  wants factored in).

If no tracker integration is available in the environment, ask the user to paste the ticket's title,
description, acceptance criteria, and any relevant comments, and work from that. Say in the report
that the ticket content was provided by the user rather than fetched.

Always weigh whether a diagram would make the workflow easier to understand — default to yes
unless the change is trivially simple (see Step 4.3).

## Step 2 — Resolve the ticket folder

* Base: `<REPORT_OUTPUT_DIR>` (ask for it if it hasn't been given).
* Folder name: `<TICKET-KEY> <Ticket Title>` (pull the title from the tracker, or from what the user
  pasted; sanitize characters invalid in filenames).
* **If the folder already exists, reuse it** — add new files, never overwrite or duplicate what's
  there. This is the one place both this ticket's analysis and, later, its test guide live, and
  it's the path used to track implementation progress against this ticket afterward.

## Step 3 — Investigate

Do the real investigation yourself — this is the substance the HTML in Step 4 is built from, not
a formatting pass over someone else's output:

1. **Read the full ticket** — acceptance criteria, description, comments, any linked PRs/MRs
   (read those for prior art).
2. **Read the design-doc / wiki reference**, if one was given.
3. **Investigate the codebase**: search for relevant existing code and patterns, identify every
   implementation the requirements will touch, and pin down the specific files/functions in scope.
   Dispatch subagents for broad searches when that's faster than reading everything yourself, but
   read in full any file you expect to name concretely in the analysis.
4. **Identify risks** — anything the change could break, and how to avoid it.
5. **Identify edge cases and scenarios** that aren't immediately obvious from the ticket text
   alone (state-dependent behavior, boundary values, concurrent/async timing, existing tests that
   assume current behavior).

## Step 4 — Write the pre-implementation HTML analysis (Analysis mode)

Save `<TICKET-KEY> analysis.html` into the Step 2 folder. Self-contained HTML (inline CSS, no
external requests).

Build it incrementally — skeleton, then styling, then each section filled in one at a time,
reviewing after each addition that nothing gathered in Step 3 got dropped:

1. **Step-by-step walkthrough** — an in-depth, plain-language explanation of each requirement/AC
   item in turn.
2. **Codebase and risk analysis** — the relevant existing code and patterns, every affected
   implementation, the specific areas of the code that will be impacted, the risk of breaking
   existing functionality for each (with a strategy to avoid it), and any edge cases or scenarios
   from Step 3.5 that aren't immediately obvious.
3. **Workflow diagram**, when one earns its place — **inline SVG by default**, drawn directly in the
   HTML so the report stays fully self-contained and works offline. It should show the real
   mechanism (the actual components, calls, and state changes), stay legible in both light and dark
   themes, and use a diagramming skill's guidance if one is available. Mermaid is an optional
   alternative only when network access to load its script is acceptable; say so if used.
4. **Incremental implementation plan** — the work broken into logical, incremental steps, each one
   independently verifiable and committable on its own. This is also the trail used later to track
   implementation progress against this same ticket folder.
5. **Findings** — one issue/fix/impact entry per finding: the issue or observation, the suggested
   fix or approach, and the impact of that fix on other parts of the codebase, if any.

Report the saved path to the user and stop — this is analysis only. No implementation, no
commits, no code changes happen as part of this skill.

## Step 5 — Write the post-implementation HTML test guide (Test-guide mode)

Save `<TICKET-KEY> test-guide.html` into the same Step 2 folder (reuse the existing ticket folder;
don't create a second one).

Sections, in order, each fully self-contained — a reader must be able to jump straight to one
environment's steps and finish them without flipping back to another section:

1. **Intro** — what this ticket was about.
2. **Summary** — what was actually implemented. Base this on the real diff/commits from this
   session, not the original plan — call out anything that changed along the way.
3. **Diagram** of the change/implementation workflow (same approach as Step 4.3 — inline SVG by
   default).
4. **Local testing approach** — how to verify the change on a local dev environment.
5. **Per-environment testing approach** — one complete, sequential, independent procedure per real
   environment. Detect the actual environments and their config from the repo's own env/config
   files (environment config files, deployment/CI configuration, build or README docs) rather than
   assuming generic names — be specific to each environment's URL/config. No "see Step 4" or "same
   as above" shortcuts between environments; each one stands alone.

Report the saved path to the user.

## Don'ts

* Don't start implementation or commit anything — both modes are analysis/reporting only.
* Don't overwrite an existing ticket folder or its files — add to them.
* Don't invent ticket details, wiki content, or codebase findings — read the real sources;
  if something can't be reached (wiki link broken, ticket field empty), say so in the report
  rather than filling it in.
* Don't invent environment names, URLs, or config — read them from the repo's actual env files.
* Don't assume an output folder or tracker URL — use the configuration placeholders and ask when
  a value is missing.
