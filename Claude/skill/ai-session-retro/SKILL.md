***

name: ai-session-retro
description: Generates a self-retrospective evaluating the AI AGENT'S OWN CONDUCT during this session — never the task's outcome or the code itself (a separate work-item retro skill, if installed, covers that). Looks specifically for hallucination/invented claims, staleness (trusting memory/old docs over live checks), miscalibrated confidence, dropped or forgotten standing instructions, unauthorized/unapproved actions, self-contradictions, and how mistakes were caught and recovered from — each with the specific evidence, why it was wrong, and a concrete workflow fix. Always saved locally; can optionally also be posted to a dedicated "AI Retro" wiki page (one page per month, one accordion per session). Also supports an optional "Monthly roll-up" mode that condenses a period's saved AI retros into one prioritized report of recurring conduct patterns. The entry's title may reference the ticket/session for identification, but the body contains zero ticket/technical content — it is entirely about the agent's behavior. Invoke at the end of a long or eventful session, or whenever the user asks for feedback on the AI's own work. Triggers on "AI session retro", "how did you do this session", "retro on yourself", "session feedback for the agent", "/ai-session-retro".
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# AI Session Retro Generator

Produce a **self-retrospective about the AI agent's conduct in this session** — not what the
session accomplished, not the ticket, not the code. A work-item retro skill (e.g. `retro`, if it is
installed) covers the task's business and technical outcome; this skill exists specifically because
that is *not* what it should contain. If a finding could be understood by someone who has never
heard of the ticket, it belongs here. If a finding requires knowing what the bug or the fix was, it
belongs in the work-item retro instead, not here.

**The test for every line you write:** strip the ticket name out of it. Does it still make sense
as a statement about how an agent behaved? "Fixed the rounding bug in the totals" fails this test —
it's about the task. "Abandoned a stated root-cause hypothesis the moment the user presented
contradicting evidence, instead of defending it" passes — it's about conduct, and it would read
the same in any other session about any other bug.

## Configuration placeholders

This skill does not assume any particular folder or wiki. Ask the user for these the first time
they're needed, then reuse them for the rest of the session:

* `<AI_RETRO_OUTPUT_DIR>` — the folder where per-session AI-retro files are saved and where the
  monthly roll-up reads from. Never assume one.
* `<WIKI_PARENT_PAGE>` — (optional integration) the wiki page under which the "AI Retro" page lives.

## Step 0 — Ask mode and destination, every invocation

Always start with one `AskUserQuestion` call:

* **Question 1 — Mode:** `Session` (retro this session) or `Monthly roll-up` (condense a period of
  saved AI retros).
* **Question 2 — only if Session:** *"Also post this AI-retro to a wiki page?"* (single-select) —
  offer it only if a wiki integration (e.g. Confluence) appears to be available; options `Yes` or
  `No, just save locally`. The local file save always happens regardless of this answer.

There is no issue-tracker-comment destination — this is feedback about the agent, not the ticket, so
it has no natural home on a ticket.

If Session was chosen, continue at Step 1. If Monthly roll-up was chosen, skip to the **Monthly
roll-up mode** section near the bottom.

## Step 1 — Identify the session (title only, not content)

Build a short title the same way a work-item retro identifies a ticket (commit messages, branch
name, or what was discussed), prefixed with today's date: `YYYY-MM-DD — <short label>`. This label
may name the ticket/feature for the reader's orientation (e.g. `PROJ-403 follow-up session`) —
that's the *only* place ticket context is allowed to appear in the whole entry. Nothing below the
title should require knowing what that ticket was about.

## Step 2 — Re-scan the session against these specific failure modes

This is the core of the skill. Don't do a generic "how did it go" pass — interrogate the session
against each named lens below in turn. For each one, either name a concrete instance (with enough
detail that a transcript reader could find and verify it — what was claimed, what tool call or
message it appeared in, what turned out to be true instead) or conclude plainly that none
occurred. Never skip a lens silently, and never invent an instance to fill one — "None found this
session" is itself a valid, useful data point (it tells the user this failure mode is genuinely
rare, not that the retro was lazy).

* **Fabrication / invented claims** — did the agent state something as fact (a file's contents, a
  tool's behavior, a system's state, a root cause) without having actually checked it, that later
  turned out to be wrong or unverifiable? This is the most serious lens — name it plainly as
  fabrication, don't soften it to "an assumption."
* **Staleness** — did the agent rely on its own memory, an earlier message in the conversation, or
  a document/file it had read earlier, instead of re-checking the live/current state, when the
  live state had actually changed or might have?
* **Miscalibrated confidence / self-estimation** — did the agent state something with more (or
  less) certainty than the evidence supported? Look specifically for a confident claim that was
  later reversed, and for the opposite failure — excessive hedging on something the agent actually
  had solid evidence for.
* **Dropped or forgotten instructions** — did the user give a standing instruction (a scope limit,
  a "don't do X yet," a "always do Y first") that the agent later failed to follow, even briefly,
  even if it self-corrected? Name the instruction and the exact point it lapsed.
* **Unauthorized / unapproved actions** — did the agent take an action with real consequence
  (running something destructive-adjacent, installing something, committing, pushing, posting
  somewhere, restructuring something) without asking first, when it should have? Distinguish this
  clearly from actions the user had already authorized in scope — only name genuinely unrequested
  autonomy here.
* **Self-contradiction** — did the agent give two different answers to the same question, or two
  different plans for the same problem, without reconciling them itself — i.e. the user had to
  notice and point out the inconsistency?
* **Mistakes made and how they were recovered** — for any real error (from the lenses above or
  otherwise), how was it caught: by the agent itself catching it unprompted, or only after the
  user pushed back? Name which, plainly — self-caught and user-caught are very different
  findings, and conflating them hides the real signal.
* **What went well** — the positive counterparts of the above: correctly asking before a
  consequential action instead of assuming; catching its own staleness or a stale memory before
  acting on it; stating uncertainty honestly instead of filling a gap with an invented answer;
  holding a standing instruction correctly across many later turns without being reminded;
  pushing back on the user's own framing with evidence rather than deferring to it.

## Step 3 — Write the entry

```markdown
### <date> — <short label>

#### What went well
1. <one-line, evidenced finding — conduct only, no ticket/technical detail>
2. <one-line, evidenced finding>

#### What needs to improve
1. **[<failure-mode tag>]** <one-line, evidenced finding>
2. **[<failure-mode tag>]** <one-line, evidenced finding>

#### How to fix / workflow update
1. <one-line, concrete change to how the agent should behave next time>
2. <one-line, concrete change>
```

Rules:

* **Tag every "What needs to improve" line** with the specific lens it came from — `Fabrication`,
  `Staleness`, `Miscalibrated confidence`, `Dropped instruction`, `Unauthorized action`,
  `Self-contradiction`, or `Mistake (self-caught)` / `Mistake (user-caught)`. This is what keeps
  the report scannable and honest — a reader should be able to tell at a glance which kind of
  failure it was without reading the sentence.
* **Maximum 4 numbered items per category.** Never pad to reach 4.
* **Each item is one line**, and **contains zero ticket/technical/code detail** — describe the
  behavior, not the subject matter it happened to occur on. If you catch yourself naming a file,
  a bug, or a fix in a "What needs to improve" line, rewrite it until it doesn't.
* **Distinguish self-caught from user-caught explicitly** wherever a mistake is named — this is
  not optional detail, it's the point. An agent that catches its own errors is a different
  (better) finding than one that needed the user to catch them, even if the underlying mistake
  looks similar on paper.
* **Don't inflate "What went well."** If a session was rough, say so with a short list rather than
  padding it — the credibility of the "needs to improve" section depends on this section not being
  dishonestly generous.
* **Don't soften language.** If something was fabricated, call it fabrication, not "an
  assumption that didn't hold up." If an instruction was dropped, say it was dropped, not
  "partially followed." This report exists specifically to be the place the agent is not
  diplomatic about itself.

## Step 4 — Present and save locally (always happens)

1. Print the full entry in the chat response, using `###`/`####` exactly as in Step 3.
2. Save to `<AI_RETRO_OUTPUT_DIR>` (ask for it if it hasn't been given; create the folder if it
   doesn't exist). Keep it separate from any work-item retro folder, since this is a different kind
   of record. Filename: `<date> <short label>.md`, sanitized for the filesystem (strip/replace
   characters invalid in filenames: `< > : " / \ | ? *`). File structure: an `#` title line, a
   `Date: YYYY-MM-DD` line, then `## What went well` / `## What needs to improve` / `## How to fix /
   workflow update` directly underneath (one level shallower than the printed chat version).
3. If a file for that exact date + label already exists, don't overwrite — append a time-stamped
   `### Update — <time>` block with new findings underneath.
4. Tell the user the exact path saved to.

## Step 5 — Post to a wiki page (optional integration; only if selected in Step 0)

This step assumes a wiki tool such as Confluence is available. Skip it silently if the user didn't
select it.

**Page hierarchy:** one `AI Retro` page, *separate from* (a sibling of, not nested under) any regular
work-item `Retro` page, with one child page per calendar month. Both can live under the same
`<WIKI_PARENT_PAGE>`.

1. **Resolve the parent.** If the user has already given `<WIKI_PARENT_PAGE>` this session, use it.
   Otherwise ask for a link or ID. Check whether an `AI Retro` page already exists beneath it;
   reuse it if so, otherwise create it. Offer to remember the resulting page ID/URL (for example in
   a small config file inside `<AI_RETRO_OUTPUT_DIR>`) so the user isn't asked again.

   **A wiki call can time out client-side (a 524 or an idle-timeout abort) while still succeeding
   server-side.** Before retrying any create/update call that reports a failure, re-check by listing
   the parent's children / re-reading the page to see whether it actually landed — never retry
   blindly, that's how a duplicate page gets created.
2. **Resolve this month's page.** Title: `AI Retro <first day> to <last day>` (first/last
   calendar day of the current month). List the `AI Retro` parent's children; update an exact
   title match, mirror an existing month's structure for a new one, or build fresh using the
   template below for the first month ever.
3. **Add one accordion per session.** If one for the exact same date+label already exists
   (re-running same-day), update it in place rather than duplicating.
4. Tell the user the page URL posted to.

### Wiki page template

```
# AI Retro — <Month Year>
<one-line description, e.g. "N sessions self-reviewed this month.">

<details><summary><date> — <short label></summary>

<strong>What went well:</strong>
1. …

<strong>What needs to improve:</strong>
1. <strong>[<tag>]</strong> …

<strong>How to fix / workflow update:</strong>
1. …

</details>
... one <details> block per session this month ...

## Monthly Roll-up
(empty until Monthly roll-up mode runs against this page)
```

No table, no ticket/technical content anywhere in the accordion body — one lens, pure conduct.
Use the wiki's real collapsible "expand" node (read the wiki tool's content-format guide first, if it
has one). Light visual markers per category are fine; keep them modest and consistent.

## Monthly roll-up mode

Only reachable via the Step 0 mode choice. Never posts anywhere except where the user directs.

1. **Choose the source.** Default: the per-session local files in `<AI_RETRO_OUTPUT_DIR>` (ask for
   the folder if it hasn't been given). If the user's AI retros live on the wiki month page from
   Step 5, they may point at that page instead; fetch it and parse its per-session accordions.
2. **Choose the period.** Ask for a date range: a specific range, or "whole month". For local files
   filter by each file's `Date:` line (or file modified date if none); for wiki pages keep only the
   accordions whose date falls in range.
3. **Analyze everything together** — same three categories (What went well / What needs to improve /
   How to fix / workflow update), but produce **one shared, de-duplicated** view: collapse
   repeated findings across sessions into a single higher-level pattern, **up to 8** items per
   category (fewer if fewer genuinely distinct patterns exist — never pad). For "What needs to
   improve", group by failure-mode tag and note roughly how many sessions each pattern appeared in
   and the self-caught vs. user-caught split. A recurring pattern is the real signal here — say so
   when one failure mode shows up in several sessions.
4. **Keep it conduct-only.** The same zero-ticket/zero-technical-detail rule applies to the roll-up.
5. **Write the result.**
   * Local source: save it as `AI Retro Monthly <YYYY-MM>.md` (or `AI Retro <start> to <end>.md`
     for a custom range) in `<AI_RETRO_OUTPUT_DIR>`. If one already exists for that period, ask
     before overwriting.
   * Wiki source: replace the page's "Monthly Roll-up" section in place; don't duplicate the heading
     or touch the per-session accordions above it.
6. Tell the user the file path or page URL and a one-line summary of what was produced.

## Don'ts

* **Don't write about the task, the ticket, the code, or the fix — anywhere in the body.** The
  title is the only place a ticket/feature name may appear. This is the one rule that matters most
  in this entire skill; violating it is exactly the failure this skill exists to avoid repeating.
* Don't nest the `AI Retro` page under a regular work-item `Retro` page — keep it a sibling.
* Don't add a Business/Technical or any two-column split — one lens only.
* Don't add an issue-tracker-comment destination.
* Don't soften a real finding into a vague "area for growth" — name the specific fabrication,
  staleness, dropped instruction, or unauthorized action, and say plainly whether the agent or the
  user caught it.
* Don't manufacture a finding to fill a lens that genuinely had nothing — say "None found."
* Don't skip the Step 0 prompt.
* Don't retry a wiki create/update after a reported failure without first checking whether it
  actually landed server-side despite the error.
* Don't assume an output folder or wiki location — use the configuration placeholders and ask when
  a value is missing.
