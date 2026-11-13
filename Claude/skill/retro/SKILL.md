***

name: retro
description: Generates a retrospective entry for one or more work items (tickets/issues) just implemented in this session — two lenses per item (Business Retro, Technical Retro), each with three categories (what went well, real obstacle, how to avoid it next time) broken into up to 4 short numbered findings, backed by evidence from the conversation, git history, and the issue tracker's changelog when available. Always saved locally; can optionally also be posted to a wiki page (e.g. Confluence) and/or as an issue-tracker comment (e.g. Jira). Also supports a "Monthly roll-up" mode that condenses a period's saved retros into one prioritized retro. Invoke after implementation work is done, typically right before or after opening the pull/merge request. Triggers on "generate a retro", "retro notes", "retrospective for this", "/retro".
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Retro Generator

Produce a **per-work-item retrospective entry** for the item(s) just worked on in this session, or
roll up a period of already-generated entries into one condensed report. Two modes:

* **Daily** — the per-item flow (Steps 1–4 below), always saved locally, optionally also published
  to a wiki page and/or as an issue-tracker comment.
* **Monthly roll-up** — reads a period's worth of already-generated retros (local files by default,
  or a wiki page) and produces one de-duplicated, prioritized roll-up.

## Configuration placeholders

This skill does not assume any particular folder, workspace, or tracker. Values the user must
supply the first time they are needed (ask once, then reuse for the rest of the session — and
offer to note them somewhere durable if the user wants):

* `<RETRO_OUTPUT_DIR>` — the folder where per-item retro files are saved and where the monthly
  roll-up reads from. Never assume one; if the user hasn't given it, ask.
* `<WIKI_PARENT_PAGE>` — (optional integration) the wiki page under which retro pages live.
* `<TRACKER_BASE_URL>` — (optional) used only to build item links when the tracker tools don't
  return a URL themselves.

## Step 0 — Ask mode and destination, every invocation

Always start with one `AskUserQuestion` call, even if `{args}` looks like work-item keys — the mode
choice is never assumed silently:

* **Question 1 — Mode:** `Daily` (retro one or more items just worked on) or `Monthly roll-up`
  (condense a period of existing retros).
* **Question 2 — only if Daily:** *"Also publish this retro to:"* (multiSelect) — offer only the
  integrations that appear to be available in the current environment (for example a wiki such as
  Confluence, an issue-tracker comment such as Jira), plus "neither". The local file save always
  happens regardless of this answer; it is not one of the choices because it is not optional.

If Daily was chosen, continue at Step 1. If Monthly roll-up was chosen, skip to the **Monthly
roll-up mode** section near the bottom — Steps 1–6 below don't apply to it.

Arguments (`{args}`, may be empty): explicit work-item keys to retro on, space- or comma-separated
(e.g. `PROJ-216 PROJ-217 PROJ-219`), only meaningful for Daily mode. If empty, infer the item set
yourself (see Step 1).

Each item gets **two retro lenses** — a **Business Retro** and a **Technical Retro** — and each
lens has exactly three categories: What went well, Real obstacle, How to avoid next time, each
broken into **up to 4 short, numbered findings**. The goal is a concise, skimmable artifact the
user can read aloud or paste into a retro doc — and one that carries **evidence**, not vibes, for
anything framed as a blocker or delay. "It was hard" is not a finding; "the AC was rewritten after
I'd already built against the first version, changing the redirect target entirely" is. Each
numbered finding is a one-line summary, not a paragraph — save the supporting detail for the
underlying evidence, not the retro entry itself.

**The two lenses look at the same work through different eyes, and most findings belong to only
one of them:**

* **Business Retro** — what a PM, product owner, or business stakeholder would care about:
  requirement/acceptance-criteria quality and completeness, description clarity, scope definition
  (was the item bundling unrelated asks, was a decision needed mid-flight that should have been
  settled upfront), time lost specifically to unclear/contradictory/missing requirements, whether
  stakeholder input had to be pulled in mid-implementation, whether the acceptance criteria matched
  what was actually built.
* **Technical Retro** — what an engineer/tech lead would care about: implementation correctness,
  tooling/environment friction, git/CI mechanics (rebases, conflicts, pipeline failures), code
  review feedback, test coverage and setup, concurrent-work collisions at the file level.

A finding about "no Acceptance Criteria were written, so target behavior had to be inferred from
the description" is a **Business** finding. A finding about "a rebase conflict in
`docs/architecture.md` cost time because a concurrent commit restructured the same section" is a
**Technical** finding. Don't force a finding into the wrong lens just to fill it out — leave a
lens's category thin or empty (see Step 3) rather than miscategorize.

## Step 1 — Identify the work-item set

If `{args}` gives explicit item keys, use those. Otherwise infer from, in order of preference:

1. Item-key prefixes (`[A-Z]+-\d+` or `#\d+`) in the last several commit messages (`git log --oneline -20`).
2. The current branch name (`git branch --show-current`) — many `feature/<KEY>-slug` branches
   name at least the lead item.
3. Item keys mentioned explicitly earlier in *this* conversation.

If you can't confidently identify any item, ask the user rather than guessing.

## Step 2 — Gather evidence, per item

Pull from every source that's actually available — don't skip a source just because it takes a
tool call, but don't stall on one that's clearly inaccessible (no tracker integration configured,
no network, etc.) — note the gap and move on rather than blocking the whole retro on it. As you
gather, keep half an eye on which lens each piece of evidence will feed — the same source (e.g. the
tracker changelog) often yields both a Business finding (AC changed) and a Technical one (which
files that touched) from the same event.

**a. This conversation (primary source, when this skill runs in the same session as the work).**
This is where the real texture lives: bugs found only via manual testing, tools that didn't behave
as documented, environment/corporate-policy friction, a design mockup that turned out different
from what the item text implied, requirements that shifted after you'd already built against them,
a decision the user had to make mid-session that the item should have already settled (e.g. "should
this be a new status or reuse an existing one?"). Re-scan the conversation for each item's arc, not
just its final diff — note both what a business reader and a technical reader would flag from the
same events.

**b. Issue-tracker changelog (objective proof of requirement changes — feeds mostly the Business
lens).** If an issue-tracker integration is available (e.g. Jira), fetch each item with its change
history expanded. Compare the description/Acceptance Criteria edit timestamps against when you
started work on that item (first commit or first message referencing it in this session). A
description/AC change *after* work started is hard, timestamped evidence of scope churn — call it
out by name (what changed, from what to what) rather than vaguely. Also check whether Acceptance
Criteria was populated at all — an item shipped with no AC, or a description that bundles several
unrelated asks without separating them, is itself a Business-lens finding worth naming even with no
post-hoc edit.

**c. Git history (objective proof of concurrent-work friction — feeds the Technical lens).** For
the relevant branch:

* `git log --oneline <branch>..<base-branch>` to see what landed on the base branch while this
  branch was in flight — a large or file-overlapping set of commits there is direct evidence a
  rebase/conflict-resolution cost was real, not self-inflicted.
* Whether a rebase actually happened and whether it was clean or required manual conflict
  resolution — if you did it in this session, you already know; otherwise check `git reflog` for
  `rebase` entries, or ask the user.
* Any commits whose message reads like rework (`fix`, `revert`, `correct`) on files this same
  session touched earlier — signals a wrong turn that cost real time, worth naming plainly rather
  than glossing over.

**d. Pull/merge-request and pipeline state, if one exists (feeds the Technical lens).** Merge
conflicts reported by the git host, failed pipeline runs that needed a fix, review comments that
triggered rework — all fair game as evidenced obstacles.

## Step 3 — Write the per-item entry: 2 lenses × 3 categories, each up to 4 numbered findings

For **each** item identified in Step 1, produce exactly this structure (heading levels shown
relative to the item header — see Step 4 for how those levels differ between the printed chat
response and the saved file):

```markdown
### <ITEM-KEY> — <short item title>

#### Business Retro
- **What went well:**
  1. <one-line finding>
  2. <one-line finding>
- **Real obstacle:**
  1. <one-line, evidenced finding>
  2. <one-line, evidenced finding>
- **How to avoid next time:**
  1. <one-line, actionable finding>
  2. <one-line, actionable finding>

#### Technical Retro
- **What went well:**
  1. <one-line finding>
  2. <one-line finding>
- **Real obstacle:**
  1. <one-line, evidenced finding>
  2. <one-line, evidenced finding>
- **How to avoid next time:**
  1. <one-line, actionable finding>
  2. <one-line, actionable finding>
```

Rules for the numbered findings under each category, in **either** lens:

* **Maximum 4 numbered items per category.** Use fewer if fewer genuine findings exist — never pad
  to reach 4. One real finding beats three padded ones.
* **Each item is one line — a headline, not a paragraph.** If a finding needs more than one
  sentence to state, it's not summarized yet; push the supporting detail (the exact AC diff, the
  exact timestamp, the exact file) into the surrounding evidence-gathering, not into the retro line
  itself. A reader should be able to skim one lens's three categories for one item in under 10
  seconds.
* **"Real obstacle" findings must be individually verifiable.** Each one names something concrete
  and checkable — a specific AC/tech-note change with a date, a specific concurrent commit/file
  overlap, a specific undocumented platform restriction hit directly, a specific missing-AC gap —
  never a vague feeling like "requirements were unclear" or "communication could improve."
* **"How to avoid next time" findings must be actionable**, not platitudes. "Confirm AC is locked
  before starting an edge-case item with active design review" beats "communicate better."
  "Split multi-item work into sub-tasks at grooming so scope-bundling decisions aren't made
  mid-implementation" beats "improve ticket quality."

If an item genuinely had no real obstacle in a given lens (rare, but possible — some items really
are as clean on the business side, or the technical side, as they look), say so plainly under that
category rather than inventing one:
`- **Real obstacle:**\n  1. None found — the AC/description gave everything needed to start.`
This applies **per lens independently** — an item can have a thin or empty Business Retro while its
Technical Retro is packed (or vice versa); don't manufacture a finding in one lens just because the
other lens was rich. Never pad a thin finding to look more dramatic than the evidence supports; the
whole point is that this entry is trustworthy enough to bring to a retro, and that trust breaks the
first time someone notices a finding was embellished or miscategorized into the wrong lens to make
a section look fuller.

## Step 4 — Present and save locally (always happens)

1. Print the full retro (all items in this run) in the chat response, ready to copy-paste. In the
   printed version, use `###` for each item header and `####` for the `Business Retro` /
   `Technical Retro` sub-headers, exactly as shown in Step 3.
2. Also save it to disk — **one file per item**, in `<RETRO_OUTPUT_DIR>` (see "Configuration
   placeholders"; ask the user for it if it hasn't been provided, and create the folder if it
   doesn't exist yet). Keeping every retro in one user-chosen folder, rather than inside the
   current repo, means retros from every project land in one place.
   * Filename per item: `<ITEM-KEY> <item title>.md` — the literal key, a space, then the item's
     title exactly as it appears in the tracker (fall back to the short title used in this retro's
     own `### <ITEM-KEY> — <title>` heading if the tracker isn't reachable). Sanitize the title for
     the filesystem: strip/replace characters invalid in filenames (`< > : " / \ | ? *`) with a
     space or dash, don't silently truncate otherwise. If two retros share the same key but are
     genuinely different bodies of work (e.g. a backend pass and a separate frontend pass),
     disambiguate the filename (e.g. append `— Frontend`) rather than colliding.
   * Each per-item file contains just that one item's content, with the key/title as an `#`
     document title and `## Business Retro` / `## Technical Retro` as the two top-level sections
     underneath (one level shallower than the printed chat version, since the file's own title
     already stands in for the `###` item header) — not the whole multi-item run. If this
     invocation covers several items, write several files.
   * Record the retro date near the top of each file (e.g. a `Date: YYYY-MM-DD` line under the
     title) so the Monthly roll-up can filter by period.
   * If a file for that exact item already exists (e.g. `/retro` was run again later, such as after
     the pull/merge request actually merged and something new came up), don't silently overwrite
     it — append a new, dated `### Update — <date>` block with any new findings underneath the
     existing `## Business Retro` / `## Technical Retro` sections instead, so earlier findings are
     never lost. Put each update's new findings under whichever lens they actually belong to.
   * Tell the user the exact path(s) you saved to.

## Step 5 — Publish to a wiki page (optional integration; Daily mode only, only if selected in Step 0)

This step assumes a wiki tool such as Confluence is available. Skip it silently if the user didn't
select it.

Suggested layout (adapt to whatever the user's wiki already uses — if retro pages already exist,
mirror their structure and naming instead of inventing a new one): one **"Retro"** parent page, with
one child page per calendar month (e.g. titled `Retro <first day> to <last day>`, first and last
day of that month). Each monthly page holds one collapsible ("expand"/accordion) block per item
retro'd that month, plus a standing "Monthly Roll-up" section at the bottom (left empty until
Monthly roll-up mode fills it in).

1. **Resolve the parent.** If the user has already given `<WIKI_PARENT_PAGE>` this session, use it.
   Otherwise ask for a link or ID to the page under which the "Retro" parent should live. Check
   whether a page titled "Retro" already exists beneath it; reuse it if so, otherwise create it.
   Offer to remember the resulting page ID/URL (for example in a small config file inside
   `<RETRO_OUTPUT_DIR>`) so the user isn't asked again.
2. **Resolve this month's page.** Compute this month's title and list the parent's children.
   * Exact title match exists → this is the page to update.
   * No match, but other monthly children exist → fetch one in HTML format and mirror its
     structure/style precisely for the new page (don't reinvent the template each month).
   * No monthly children at all (first month ever) → build the page from the template below.
3. **Add/update one accordion per item in this run.** If an accordion for that exact item (key +
   variant) already exists on the page, replace its body with the latest content in place rather
   than duplicating it (the wiki keeps its own version history).
4. Tell the user the page URL you posted to.

### Wiki page template

```
# Retro — <Month Year>
<one-line description of the month, e.g. "N items closed across backend/frontend/infra work.">

<details><summary><ITEM-KEY> — <item title></summary>
<short 1-line summary of what the item was about>
Item: <link, no status>
PR/MR: <link, no status — or "N/A" if none found>

| Business | Technical |
|---|---|
| **What went well:** …  **Real obstacle:** …  **How to avoid next time:** … | (same three, technical lens) |

</details>
... one <details> block per item ...

## Monthly Roll-up
(empty until Monthly roll-up mode runs against this page)
```

Use the wiki's real collapsible "expand" node for each item (not a plain heading) so the page is
scannable rather than a wall of text. Light visual structure is welcome (a marker per category, a
short intro line), but keep it modest and consistent with the wiki's existing style. If a whole lens
(Business or Technical) has no real content for an item, put a single **N/A** in that table cell
instead of the three category bullets — don't force empty categories.

Sourcing the accordion body's pieces:

* **Summary line:** the item's tracker description/summary in one line, or synthesized from the
  local retro file's own content if the tracker isn't reachable.
* **Item link:** the plain tracker URL for the key, no status/lozenge. Omit entirely (or mark "N/A")
  for a synthetic, non-tracker entry.
* **PR/MR link:** best-effort — search the git host (code/MR/commit search) for a pull/merge request
  referencing the item key in the relevant repo(s). If nothing turns up, write "N/A" — never block
  the retro on a missing PR/MR.
* **Business/Technical table:** copied verbatim from the item's already-saved local file (Step 4) —
  this step formats existing content, it does not re-derive it.

## Step 6 — Post to the issue tracker (optional integration; Daily mode only, only if selected in Step 0)

For each real tracker item in this run (skip synthetic/non-tracker entries silently), add a comment
on the item containing the same Business Retro + Technical Retro content generated in Step 3,
formatted in the tracker's own markup or plain text (not wiki HTML). Tell the user which items got
a comment and link each one.

## Monthly roll-up mode

Only reachable via the Step 0 mode choice. Ignores `{args}`; never posts to the issue tracker.

1. **Choose the source.** Default: the per-item local files in `<RETRO_OUTPUT_DIR>` (ask for the
   folder if it hasn't been given). If the user's retros live on a wiki page instead (the monthly
   page from Step 5), they may point at that page; fetch it in HTML format and parse out its
   per-item accordions.
2. **Choose the period.** Ask for a date range: a specific range, or "whole month". For local files
   filter by each file's recorded date (or file modified date if none is recorded); for wiki pages
   keep only the accordions whose item date falls in range.
3. **Analyze everything together** — same three categories (What went well / Real obstacle / How
   to avoid next time), each split Business | Technical — but this time produce **one shared,
   de-duplicated** view: collapse repeated/overlapping findings across items into a single
   higher-level point, keep only what's genuinely non-repeating and important, **up to 8** bullets
   per category (fewer if fewer genuinely distinct points exist — never pad to reach 8). When
   several items share a finding, note roughly how many items it came from.
4. **Write the result.**
   * Local source: save it as a new file in `<RETRO_OUTPUT_DIR>` named
     `Monthly Retro <YYYY-MM>.md` (or `Retro <start> to <end>.md` for a custom range). If one
     already exists for that period, ask before overwriting.
   * Wiki source: replace the page's existing "Monthly Roll-up" section in place; don't duplicate
     the heading or touch the per-item accordions above it.
5. Tell the user the file path or page URL and a one-line summary of what was produced.

## Don'ts

* Don't inflate an item that genuinely went smoothly in a given lens — an honest "no real obstacle"
  is more valuable than a manufactured one, and it's what keeps the rest of the entry credible.
* Don't write vague obstacles ("requirements were unclear") when a concrete one is available from
  evidence ("the item shipped with no Acceptance Criteria, and its description bundled 4 unrelated
  validation changes across two repos with no split called out") — always prefer the specific,
  sourced version.
* Don't cross-contaminate the lenses — a git-conflict-resolution finding belongs in Technical, not
  Business, even if the underlying cause (concurrent items landing at once) has a business-y
  flavor; a missing-AC finding belongs in Business, not Technical, even though it was discovered
  while reading code. Sort by "who would care about fixing this," not by where it was noticed.
* Don't exceed 4 numbered findings per category in Daily mode (8 in Monthly roll-up mode), and
  don't merge multiple items into one Daily entry even if they shipped in the same PR/MR — each
  item gets its own two-lens block.
* Don't let a numbered finding run past one line. If it needs a second sentence, it's not
  summarized — tighten it instead of letting it sprawl.
* Don't skip the Step 0 mode/destination prompt, even when `{args}` makes the item set obvious —
  the destination choice is never assumed.
* Don't re-derive Business/Technical content when publishing to a wiki or tracker — reuse exactly
  what Step 3 already produced; Steps 5/6 are formatting/publishing steps, not analysis steps.
* Don't block an entire run on a missing PR/MR link, an unreachable tracker item, or a synthetic
  non-tracker entry — degrade to "N/A" and keep going.
* Don't assume an output folder, wiki location, or tracker — use the configuration placeholders and
  ask the user when a value is missing.
