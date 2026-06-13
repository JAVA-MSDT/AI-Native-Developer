You are executing the `/rollback-step` workflow. Follow every step in order.

## Step 1 — Read State

Read the active state file. First read `<codebase_path>/.dev-workflow/active_state.json` to find the current state file path. If `active_state.json` does not exist, look for any `*_state.json` file in `.dev-workflow/` and use the most recently modified one.

If no state file is found, stop and tell the user: "No active workflow found. Nothing to rollback."

Extract: `completed_steps`, `current_step`, `state_path`.

If `completed_steps` is empty, stop and tell the user: "No completed steps to rollback."

## Step 2 — Determine Which Step to Rollback

- If a step number was passed as an argument, find that step in `completed_steps`.
- Otherwise, use the last entry in `completed_steps`.

If the requested step is not found in `completed_steps`, tell the user which steps are available to rollback.

## Step 3 — Find the Commit Hash

The commit hash may be stored in `completed_steps[N].commit`. If it is `"pending"` or missing (because the developer commits manually), find the commit using git log:

```bash
git log --oneline -10
```

Match the commit message from the step's `commit_message` field to find the right hash. Show the matches to the user and ask them to confirm which commit to revert if there is ambiguity.

## Step 4 — Confirm

Display:
```
Rollback Step N: <title>
Commit to revert: <commit hash> — "<commit message>"
Warning: This creates a revert commit. The original commit stays in git history.
```

Ask: "Confirm rollback of step N? (yes / no)"

Wait for explicit confirmation. Do not proceed without it.

## Step 5 — Provide the Revert Command

Show the user the exact command to run:

```bash
git revert <commit_hash> --no-edit
```

Tell the user:
> "Run the command above. If there are merge conflicts, resolve them then run `git revert --continue`. Come back here when the revert is complete."

Do not run the revert automatically — let the developer execute it.

## Step 6 — Update State After User Confirms Revert is Done

Ask: "Has the revert been completed?"

Once the user confirms:

## Step 7 — Update State

Update the state file at `state_path`:
- Remove the rolled-back step from `completed_steps`
- Set `current_step` to the step number before the rolled-back step (or 0 if it was step 1)
- Add a `rollbacks` array (or append to it) with:
  ```json
  { "step": N, "original_commit": "<hash>", "revert_commit": "<revert_hash>" }
  ```

## Step 8 — Present to User

Show:
1. Step N reverted successfully
2. Revert commit hash
3. Current state: which steps are still completed

Then say: "Use `/approve-step` to re-implement step N with a revised approach, or `/submit-review-feedback` if the implementation plan itself needs revision."
