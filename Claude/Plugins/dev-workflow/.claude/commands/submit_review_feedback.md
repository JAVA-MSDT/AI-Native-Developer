You are executing the `/submit-review-feedback` workflow. Follow every step in order.

## Step 1 — Read State

Read the active state file. First read `<codebase_path>/.dev-workflow/active_state.json` to find the current state file path. If `active_state.json` does not exist, look for any `*_state.json` file in `.dev-workflow/` and use the most recently modified one.

If no state file is found, stop and tell the user: "No active analysis found. Run `/start-ticket-analysis` first."

Read the state file and extract: `report_path`, `codebase_path`, `implementation_plan`, `review_iterations`, `state_path`.

**Validate required fields.** If any of these are missing or null, stop and tell the user:
> "The state file is incomplete (missing: `<field list>`). This usually means `/start-ticket-analysis` did not finish successfully. Re-run it to create a fresh analysis."

Required: `codebase_path`, `state_path`, `report_path`, `implementation_plan`.

## Step 2 — Collect Findings

If the user did not pass findings as an argument, ask:
> "What gaps, errors, or areas need more analysis in the current report?"

Accept any free-form text. The user may paste section excerpts, point to specific requirements, or describe missing scenarios.

## Step 3 — Read the Current Report

Read the current report from `report_path` to understand what was already found and what the open questions were.

## Step 4 — Re-analyze Targeted Areas

First, load codebase context without re-exploring from scratch:
1. Read `<codebase_path>/.dev-workflow/codebase_context.md` — this is the codebase snapshot written during initial analysis. Tell the user: **"Reading from codebase snapshot (`codebase_context.md`) for base context."**

Then for each gap or finding the user identified:
2. Do targeted reads only — use Glob and Grep to find files **directly related to the specific gap**. Do NOT re-explore the full codebase.
3. For each file you read directly from the codebase (not the snapshot), tell the user: **"Reading `<file path>` directly from codebase."**
4. Answer open questions where possible.
5. Check if the gap reveals additional affected files or risks not in the previous analysis.
6. Check if the implementation plan needs to be updated (new steps, reordered steps, modified file lists).

## Step 5 — Update the Report

Edit the existing report file in place — do not create a new one.

Changes to make:
- Add a **"Review Iteration N"** section near the top (after the header, before Section 1) summarizing what was addressed
- Update the relevant sections with the new analysis
- Mark previously open questions as ✅ Resolved or ❌ Unresolved
- Add any new open questions discovered during re-analysis
- If the implementation plan changed, update Section 4 in full and update the `implementation_plan` in the state file

## Step 6 — Update State

Update the state file at `state_path`:
- Increment `review_iterations` by 1
- If implementation plan changed, overwrite `implementation_plan` with the updated array

## Step 7 — Present to User

Show:
1. What was updated in the report (bullet list)
2. Which findings were resolved vs still open
3. Whether the implementation plan changed

Then say: "Continue reviewing the updated report. Use `/submit-review-feedback` again with more notes, or `/approve-step` when you're satisfied and ready to implement."
