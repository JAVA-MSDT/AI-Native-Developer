You are executing the `/approve-step` workflow. Follow every step in order.

## Step 1 — Read State

Read the active state file. First read `<codebase_path>/.dev-workflow/active_state.json` to find the current state file path. If `active_state.json` does not exist, look for any `*_state.json` file in `.dev-workflow/` and use the most recently modified one.

If no state file is found, stop and tell the user: "No active analysis found. Run `/start-ticket-analysis` first."

Extract: `implementation_plan`, `current_step`, `completed_steps`, `codebase_path`, `phase`, `test_command`, `state_path`.

## Step 2 — Determine Which Step to Execute

- If a step number was passed as an argument, use it.
- Otherwise, use `current_step + 1`. If `current_step` is 0, this is step 1.

Look up the step in `implementation_plan` by its `step` number.

If the step number is beyond the last step in the plan, tell the user: "All N implementation steps are complete. Review your changes with `git log`."

## Step 3 — Show Step Details and Confirm

Display:
```
Step N: <title>
Description: <description>
Files to modify: <files list>
Test command: <test_command>
Commit message: <commit_message>
```

Ask: "Proceed with this step? (yes / no / show me the files first)"

If the user says "show me the files first", read each file and show the relevant sections, then ask again.

Wait for explicit confirmation before proceeding. Do not assume yes.

## Step 4 — Implement the Step

Using the `files` list from the step plan:

1. Read each file first (always read before editing)
2. Make only the changes described in the step `description`
3. Use Edit to make targeted changes — do not rewrite whole files unless necessary
4. **Do NOT** alter, refactor, or clean up code outside the declared scope
5. **Do NOT** implement any part of the next step

## Step 5 — Run Tests

Determine the test command (in priority order):
1. The step's own `test_command` field (if defined and non-empty)
2. The project-level `test_command` from `workflow_state.json` (if set)
3. Ask the user: "What command should I run to verify this step?"

Run the resolved test command from `codebase_path`.

**If tests fail:**
- Show the full failure output
- Diagnose the root cause
- Propose a fix
- Ask: "Tests failed. Options: (1) Fix and retry, (2) Skip tests, (3) Rollback this step. What would you like to do?"
- Do NOT proceed without user input

**If tests pass:**
- Show a summary of passing results

## Step 6 — Show Implementation Summary and Ask for Review

Display a summary of what was changed:
- Which files were created or modified
- A brief description of the changes made
- Test results (passed / skipped)

Then ask:
> "Does the implementation look correct to you? Any changes needed before committing?
> - Reply **'looks good'** and I'll give you the commit command to run.
> - Reply with **comments or corrections** and I'll update the code before you commit."

Wait for the user's response. Do not commit automatically.

## Step 7 — Handle Review Response

**If the user says it looks good (or equivalent):**

Show the exact git commands to commit this step:

```bash
git add <list each file explicitly — one per line>
git commit -m "<commit_message from plan>"
```

Tell the user:
> "Run the commands above to commit step N. Once committed, run `/approve-step` to continue with step N+1, or `/rollback-step` if you need to undo this step."

**If the user provides corrections or comments:**

Make the requested changes to the relevant files, then go back to Step 5 (re-run tests). Repeat Step 6 after the fix.

## Step 8 — Update State

After the user confirms the implementation looks good (at Step 6), update the state file at `state_path` immediately — do not wait for the commit:
- Set `phase` to `"implementation"`
- Set `current_step` to the step number just completed
- Append to `completed_steps`:
  ```json
  { "step": N, "title": "<title>", "commit": "pending" }
  ```

When the user later runs `/approve-step` for the next step, the state will correctly show which step was last worked on.

> **Note on rollback:** `/rollback-step` uses `git log` to find the relevant commit automatically — you do not need to provide the commit hash manually.

## Step 9 — Check for Completion

After updating state, check if the just-completed step is the last step in `implementation_plan` (i.e., `current_step == implementation_plan.length`).

**If this was the last step:**

Update `active_state.json` to signal no active workflow:
```json
{ "state_path": null }
```

Then tell the user:

> "All N implementation steps are complete.
>
> - State preserved at `.dev-workflow/<prefix>_state.json` — use `/rollback-step` if you need to revert any step.
> - Review the full change history with `git log --oneline`.
> - Run `/start-ticket-analysis` when you're ready for the next ticket."

**If there are more steps remaining:**

Tell the user:
> "Step N complete. Run `/approve-step` to continue with step N+1."
