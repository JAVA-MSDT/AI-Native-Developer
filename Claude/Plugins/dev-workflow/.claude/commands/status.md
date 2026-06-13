You are executing the `/status` command. Display the current workflow state. This command is strictly read-only — do not write or edit any files.

## Step 1 — Read State File

Attempt to read `.claude/state/workflow_state.json`.

**Case A — File not found:**
Stop and display:
```
No active workflow found.
Run /start-ticket-analysis to begin.
```

**Case B — File exists but is not valid JSON:**
Stop and display:
```
Warning: .claude/state/workflow_state.json is not valid JSON — the file may be corrupted.
To reset: delete .claude/state/workflow_state.json and run /start-ticket-analysis.
```
Do not attempt to display partial content.

**Case C — Valid JSON but missing expected fields:**
Continue to Step 2, but for each missing field output `[field_name]: not found in state` in place of its value.

**Case D — Fully valid state:**
Continue to Step 2.

## Step 2 — Display Status

Print a formatted status summary using the fields below. Use clear labels and clean formatting.

```
─────────────────────────────────────────
  Dev Workflow — Current Status
─────────────────────────────────────────
  Phase:            <phase>
  Ticket source:    <ticket_source>
  Ticket ID:        <ticket_id or "n/a">
  Codebase:         <codebase_path>
  Report:           <report_path>
  Review cycles:    <review_iterations>

  Implementation Progress:
    <current_step> of <total steps> steps complete

  Completed Steps:
    <for each entry in completed_steps:>
    ✓ Step <step>: <title>  [<commit>]
    <if completed_steps is empty:>
    (none yet)

  All Commits:
    <for each hash in git_commits:>
    - <hash>
    <if git_commits is empty:>
    (none yet)
─────────────────────────────────────────
```

Where `<total steps>` is the length of the `implementation_plan` array.

For Case C (missing fields), substitute `[field_name]: not found in state` for any missing value rather than leaving it blank or erroring.

## Step 3 — Done

Do not modify any files. Do not update state. The command is complete.
