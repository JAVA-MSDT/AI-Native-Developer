You are executing the `/submit-review-feedback` workflow. Follow every step in order.

## Step 1 — Read State

Read the active state file. First read `<codebase_path>/.dev-workflow/active_state.json` to find the current state file
path. If `active_state.json` does not exist, look for any `*_state.json` file in `.dev-workflow/` and use the most
recently modified one.

If no state file is found, stop and tell the user: "No active analysis found. Run `/start-ticket-analysis` first."

Read the state file and extract: `report_path`, `codebase_path`, `implementation_plan`, `review_iterations`,
`state_path`, `completed_steps`.

**Validate required fields.** If any of these are missing or null, stop and tell the user:
> "The state file is incomplete (missing: `<field list>`). This usually means `/start-ticket-analysis` did not finish
> successfully. Re-run it to create a fresh analysis."

Required: `codebase_path`, `state_path`, `report_path`, `implementation_plan`.

`completed_steps` defaults to `[]` if absent — this is not an error.

## Step 2 — Collect Findings

If the user did not pass findings as an argument, ask:
> "What gaps, errors, or areas need more analysis in the current report?"

Accept any free-form text. The user may paste section excerpts, point to specific requirements, or describe missing
scenarios.

## Step 3 — Read the Current Report

Read the current report from `report_path` to understand what was already found and what the open questions were.

## Step 4 — Re-analyze Targeted Areas

First, load codebase context without re-exploring from scratch:

### 4.0 — Patch snapshot for completed steps (if any)

Before reading the snapshot, check whether any implementation steps have already been executed.

If `completed_steps.length > 0`:

1. Collect every unique file path across all `completed_steps[].files` entries into a `changed_files` list.
2. Tell the user: **"N step(s) already implemented — re-reading changed files to update snapshot before analysis:
   `<changed_files list>`"**
3. Read each file in `changed_files` directly from `codebase_path` (live filesystem).
4. For each file, find its row in the **Relevant File Map** table inside
   `<codebase_path>/.dev-workflow/codebase_context.md` and update the **Key symbols** column to reflect the
   current state of that file. Use Edit to patch only the affected rows — do not rewrite the full snapshot.
5. After patching, add a line at the top of `codebase_context.md` (below the `Generated:` header):
   ```
   Patched: <date> | Steps applied: <completed step numbers>
   ```

If `completed_steps` is empty, skip this sub-step entirely.

### 4.1 — Load snapshot

1. Read `<codebase_path>/.dev-workflow/codebase_context.md` — now accurate for all implemented files.
   Tell the user: **"Reading from codebase snapshot (`codebase_context.md`) for base context."**

### 4.2 — Targeted re-analysis

Then for each gap or finding the user identified:

2. Do targeted reads only — use Glob and Grep to find files **directly related to the specific gap**. Do NOT re-explore
   the full codebase.
3. For each file you read directly from the codebase (not the snapshot), tell the user: **"Reading `<file path>`
   directly from codebase."**
4. Answer open questions where possible.
5. Check if the gap reveals additional affected files or risks not in the previous analysis.
6. Check if the implementation plan needs to be updated (new steps, reordered steps, modified file lists).

## Step 5 — Update the Report

Edit the existing report file in place — do not create a new one.

Changes to make:

- Add a **"Review Iteration N"** section near the top (after the header, before Section 1) summarizing what was
  addressed
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

Then say: "Continue reviewing the updated report. Use `/submit-review-feedback` again with more notes, or
`/approve-step` when you're satisfied and ready to implement."
