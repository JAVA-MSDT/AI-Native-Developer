You are executing the `/approve-step` workflow. Follow every step in order.

## Step 1 — Read State

Read the active state file. First read `<codebase_path>/.dev-workflow/active_state.json` to find the current state file
path. If `active_state.json` does not exist, look for any `*_state.json` file in `.dev-workflow/` and use the most
recently modified one.

If no state file is found, stop and tell the user: "No active analysis found. Run `/start-ticket-analysis` first."

Extract: `implementation_plan`, `current_step`, `completed_steps`, `codebase_path`, `phase`, `test_command`,
`state_path`.

**Validate required fields.** If any of these are missing or null, stop and tell the user:
> "The state file is incomplete (missing: `<field list>`). This usually means `/start-ticket-analysis` did not finish
> successfully. Re-run it to create a fresh analysis."

Required: `codebase_path`, `state_path`, `implementation_plan`, `current_step`, `file_prefix`.

## Step 2 — Determine Which Step to Execute

- If a step number was passed as an argument, use it.
- Otherwise, use `current_step + 1`. If `current_step` is 0, this is step 1.

Look up the step in `implementation_plan` by its `step` number.

If the step number is beyond the last step in the plan, tell the user: "All N implementation steps are complete. Review
your changes with `git log`."

**Check for pending commits.** Scan `completed_steps` for any entries where `"commit": "later"`. If found, display a
reminder banner before proceeding:

```
⚠ Uncommitted steps: <list step numbers and titles marked "later">
  These changes are implemented but not yet committed.
  You can commit them now before continuing, or continue and commit later.
```

## Step 3 — Show Step Details and Confirm

Display:

```
Step N: <title>
Description: <description>
Files to modify: <files list>
Test command: <test_command>
Commit message: <commit_message>
```

Then ask two questions in one prompt:

> "Proceed with this step? And how should tests run?
>
> Test execution:
> - **auto** — I run `<test_command>` and show you the output *(costs tokens — full output enters context)*
> - **manual** — I skip running tests; you run `<test_command>` yourself and paste the result *(saves tokens)*
>
> Reply with both answers together, for example:
> - `yes auto` — proceed and run tests automatically
> - `yes manual` — proceed and I'll ask you to run tests yourself
> - `no` — don't proceed with this step
> - `show me the files first` — show relevant file sections before deciding"

If the user says "show me the files first", read each file and show the relevant sections, then ask both questions again.

Record the test mode choice (`auto` or `manual`) for use in Step 5. If the user answers only the proceed question and omits test mode, default to `auto`.

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

**If test mode is `auto` (chosen in Step 3):**

Run the resolved test command from `codebase_path`.

**If tests fail:**

- Show the full failure output
- Diagnose the root cause
- Propose a fix
- Ask: "Tests failed. Options: (1) Fix and retry, (2) Skip tests, (3) Rollback this step. What would you like to do?"
- Do NOT proceed without user input

**If tests pass:**

- Show a summary of passing results

---

**If test mode is `manual` (chosen in Step 3):**

Tell the user:

> "Run this command yourself to verify the step:
>
> ```
> <test_command>
> ```
>
> Reply with:
> - **passed** — tests green, ready to continue
> - **failed: \<summary\>** — paste the key failure lines and I'll diagnose
> - **skip** — proceed without running tests *(not recommended)*"

Wait for the user's reply.

- **passed** → proceed to Step 6.
- **failed: \<summary\>** → diagnose from the pasted output, propose a fix, re-implement if needed, then ask the user to re-run tests manually and report again.
- **skip** → proceed to Step 6 with a note: `test_output: "skipped by developer"`.

Do NOT proceed without a response.

## Step 6 — Show Implementation Summary and Ask for Review

Run `git diff` from `codebase_path` to show the actual changes made:

```bash
git diff
```

Display the full diff output so the review is based on real changes, not a description of them.

Then summarize:

- Which files were created or modified
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

Then ask:
> "Have you committed step N?
> - **yes** — committed, ready to continue
> - **later** — skip commit for now, I'll batch it with upcoming steps
> - **not yet** — haven't run the command yet"

Wait for a response.

**If the user says "not yet":** repeat the git commands and wait again.

**If the user says "yes":** Before updating state, verify the commit actually landed:

```bash
git log --oneline -1
```

Run this from `codebase_path`. Check that the output contains the expected commit message (`<commit_message from plan>`).

- **If it matches** — proceed to Step 8 with commit status `"committed"`.
- **If it does not match** — do NOT update state. Show the actual latest commit and tell the user:
  > "The latest commit doesn't match the expected message for step N. Expected: `<commit_message>`  
  > Found: `<actual commit line>`  
  > Please run the git commands above and reply **yes** again once the commit is in place."
  
  Repeat the git commands and wait.

**If the user says "later":** proceed to Step 8 with commit status `"later"`. Tell the user:
> "Noted — step N's changes are implemented but not committed. Remember to commit before pushing or opening a PR.
> Run `/rollback-step N` if you need to undo these changes (note: rollback requires a committed hash — uncommitted
> changes must be reverted manually with `git checkout -- <files>`)."

**If the user provides corrections or comments:**

Make the requested changes to the relevant files, then go back to Step 5 (re-run tests). Repeat Step 6 after the fix.

## Step 8 — Update State

After the user responds in Step 7, update the state file at `state_path`:

- Set `phase` to `"implementation"`
- Set `current_step` to the step number just completed
- Append to `completed_steps` using the commit status from Step 7:
  ```json
  { "step": N, "title": "<title>", "commit": "committed" }   ← if user said "yes"
  { "step": N, "title": "<title>", "commit": "later" }       ← if user said "later"
  ```

When the user later runs `/approve-step` for the next step, the state will correctly show which step was last worked on.

> **Note on rollback:** `/rollback-step` uses `git log` to find the relevant commit. Steps marked `"commit": "later"`
> have no standalone commit — they must be reverted manually with `git checkout -- <files>` if needed.

## Step 9 — Check for Completion

After updating state, check if the just-completed step is the last step in `implementation_plan` (i.e.,
`current_step == implementation_plan.length`).

**If this was the last step:**

**Check for uncommitted steps.** Scan `completed_steps` for any entries with `"commit": "later"`. If found, show a
warning before the PR description:

```
⚠ Uncommitted steps detected — commit these before pushing:
  <list each step number, title, and suggested git command>
  git add <files for that step>
  git commit -m "<commit_message from plan>"
```

Update `active_state.json` to signal no active workflow:

```json
{ "state_path": null }
```

Generate a PR description from the state file and display it ready to copy:

```
## <ticket_id or source type>: <ticket title>

### Summary
<2–3 sentence summary of what was implemented, derived from the ticket requirements>

### Changes
<one bullet per completed step: "- <step title> (`<commit_message>`)" >

### Files Changed
<deduplicated list of all files across all completed steps>

### Test Coverage
<list of test commands that were run, one per step>
```

Then tell the user:

> "All N implementation steps are complete.
>
> - Copy the PR description above into your pull request.
> - State preserved at `.dev-workflow/<prefix>_state.json` — use `/rollback-step` if you need to revert any step.
> - Review the full change history with `git log --oneline`.
> - Run `/start-ticket-analysis` when you're ready for the next ticket."

**If there are more steps remaining:**

Tell the user:
> "Step N committed and complete.
>
> - Run `/approve-step` to continue with step N+1: <next step title>
> - Run `/rollback-step` if you need to undo this step before moving forward."
