You are executing the `/rollback-step` workflow. Follow every step in order.

## Step 1 — Read State

Read `.claude/state/workflow_state.json`.

If the file does not exist, stop and tell the user: "No active workflow found. Nothing to rollback."

Extract: `completed_steps`, `git_commits`, `current_step`.

If `completed_steps` is empty, stop and tell the user: "No completed steps to rollback."

## Step 2 — Determine Which Step to Rollback

- If a step number was passed as an argument, find that step in `completed_steps`.
- Otherwise, use the last entry in `completed_steps`.

If the requested step is not found in `completed_steps`, tell the user which steps are available to rollback.

## Step 3 — Confirm

Display:
```
Rollback Step N: <title>
Commit to revert: <commit hash>
Warning: This creates a revert commit. The original commit stays in git history.
```

Ask: "Confirm rollback of step N? (yes / no)"

Wait for explicit confirmation. Do not proceed without it.

## Step 4 — Revert the Commit

```bash
git revert <commit_hash> --no-edit
```

If git revert produces conflicts:
- Show the conflicted files
- Ask: "There are merge conflicts. Would you like me to resolve them, or do you want to handle this manually?"
- Do not auto-resolve without user input

Capture the revert commit hash from the output.

## Step 5 — Update State

Update `.claude/state/workflow_state.json`:
- Remove the rolled-back step from `completed_steps`
- Set `current_step` to the step number before the rolled-back step (or 0 if it was step 1)
- Remove the original commit hash from `git_commits`
- Add a `rollbacks` array (or append to it) with:
  ```json
  { "step": N, "original_commit": "<hash>", "revert_commit": "<revert_hash>" }
  ```

## Step 6 — Present to User

Show:
1. Step N reverted successfully
2. Revert commit hash
3. Current state: which steps are still completed

Then say: "Use `/approve-step` to re-implement step N with a revised approach, or `/submit-review-feedback` if the implementation plan itself needs revision."
