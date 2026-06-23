You are executing the `/approve-step` workflow. Follow every step in order.

## Step 1 — Read State

Read `<codebase_path>/.dev-workflow/active_state.json` to find the current state file path. If it does not exist,
look for any `*_state.json` file in `.dev-workflow/` and use the most recently modified one.

If no state file is found, stop: "No active analysis found. Run `/start-ticket-analysis` first."

Extract: `implementation_plan`, `current_step`, `completed_steps`, `codebase_path`, `phase`, `state_path`,
`file_prefix`.

**Validate required fields.** Stop if any are missing or null:
> "The state file is incomplete (missing: `<field list>`). Re-run `/start-ticket-analysis` to create a fresh analysis."

Required: `codebase_path`, `state_path`, `implementation_plan`, `current_step`, `file_prefix`.

## Step 2 — Determine Which Step to Execute

- If a step number was passed as an argument, use it.
- Otherwise use `current_step + 1`. If `current_step` is 0, this is step 1.

Look up the step in `implementation_plan` by its `step` number.

If the step number is beyond the last step, tell the user:
> "All N implementation steps are complete. Review your changes with `git log`."

**Check for pending commits.** Scan `completed_steps` for any entries where `"commit": "later"`. If found:

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

Record the test mode choice (`auto` or `manual`). If the user answers only the proceed question and omits test mode,
default to `auto`.

Wait for explicit confirmation before proceeding. Do not assume yes.

## Step 4 — Spawn implementation_agent

Use the Task tool to spawn `implementation_agent` with this prompt:

```
Implement the following step and return the result.

step: <step JSON object>
codebase_path: <codebase_path>
test_mode: <auto | manual>
```

Wait for the agent to complete. It returns:

```json
{
  "step": N,
  "status": "completed | failed",
  "test_output": "string",
  "files_modified": ["string"]
}
```

**If `status` is `"failed"`:**

Show the full `test_output`. Then ask:
> "Tests failed. Options:
> - **fix** — I'll diagnose and fix, then re-run
> - **skip** — proceed without passing tests *(not recommended)*
> - **rollback** — undo all changes to these files"

- **fix**: re-spawn `implementation_agent` with the same step + `corrections: "Tests failed with: <test_output> — diagnose and fix"`. Loop back to this step after agent returns.
- **skip**: proceed to Step 5 with `test_output: "skipped by developer"`.
- **rollback**: run `git checkout -- <files_modified>` from `codebase_path`, update state to remove this step, stop.

**If `status` is `"completed"`:**

Compare `agent_result.files_modified` against `step.files`. If any file from `step.files` is absent from
`files_modified`, re-spawn `implementation_agent` immediately with:

```
corrections: "The following files were expected but not created or modified: <missing files list>.
Implement them now as part of this step."
```

Do not proceed to Step 5 until `files_modified` accounts for every file in `step.files`.

## Step 5 — Handle Manual Test Mode

**If `test_mode` is `"manual"`:**

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
- **failed: \<summary\>** → re-spawn `implementation_agent` with `corrections: "Tests failed with: <summary> — diagnose and fix"`. Return to Step 5 after agent completes.
- **skip** → proceed to Step 6 with a note: `test_output: "skipped by developer"`.

**If `test_mode` is `"auto"`:** skip this step — tests already ran in the agent.

## Step 6 — Show git diff and Ask for Review

Run from `codebase_path`:

```bash
git diff
```

Display the full diff output so the review is based on real changes, not a description of them.

Then summarize:
- Which files were created or modified
- Test results (passed / skipped / manual)

Ask:
> "Does the implementation look correct? Any changes needed before committing?
> - Reply **'looks good'** and I'll give you the commit command to run.
> - Reply with **comments or corrections** and I'll update the code before you commit."

Wait for the user's response.

## Step 7 — Handle Review Response

**If the user provides corrections:**

Re-spawn `implementation_agent` with:

```
Implement corrections on top of the current file state.

step: <step JSON object>
codebase_path: <codebase_path>
test_mode: <test_mode from Step 3>
corrections: <user's correction text>
```

After the agent returns, go back to Step 5 (re-handle manual tests if needed), then repeat Step 6.

**If the user says it looks good (or equivalent):**

Show the exact git commands:

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

**If the user says "yes":** verify the commit actually landed:

```bash
git log --oneline -1
```

Run from `codebase_path`. Check that the output contains the expected commit message.

- **If it matches** — proceed to Step 8 with commit status `"committed"`.
- **If it does not match** — do NOT update state. Tell the user:
  > "The latest commit doesn't match the expected message for step N. Expected: `<commit_message>`
  > Found: `<actual commit line>`
  > Please run the git commands above and reply **yes** again once the commit is in place."
  Repeat the git commands and wait.

**If the user says "later":** proceed to Step 8 with commit status `"later"`. Tell the user:
> "Noted — step N's changes are implemented but not committed. Remember to commit before pushing or opening a PR.
> Run `/rollback-step N` if you need to undo these changes (note: rollback requires a committed hash — uncommitted
> changes must be reverted manually with `git checkout -- <files>`)."

## Step 8 — Update State

Update the state file at `state_path`:

- Set `phase` to `"implementation"`
- Set `current_step` to the step number just completed
- Append to `completed_steps`:
  ```json
  { "step": N, "title": "<title>", "commit": "committed" }   ← if user said "yes"
  { "step": N, "title": "<title>", "commit": "later" }       ← if user said "later"
  ```

## Step 9 — Check for Completion

Check if the just-completed step is the last step in `implementation_plan`.

**If there are more steps remaining:**

> "Step N committed and complete.
>
> - Run `/approve-step` to continue with step N+1: <next step title>
> - Run `/rollback-step` if you need to undo this step before moving forward."

**If this was the last step:**

**Check for uncommitted steps.** Scan `completed_steps` for any `"commit": "later"`. If found:

```
⚠ Uncommitted steps detected — commit these before pushing:
  <list each step number, title, and suggested git command>
  git add <files for that step>
  git commit -m "<commit_message from plan>"
```

Update `active_state.json`:
```json
{ "state_path": null }
```

Generate and display the PR description:

```
## <ticket_id or source type>: <ticket title>

### Summary
<2–3 sentence summary of what was implemented>

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
