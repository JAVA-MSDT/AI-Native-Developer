You are executing the `/start-ticket-analysis` workflow. Follow every step in order.

## Step 1 — Collect Inputs

You need these inputs. If they were not passed as arguments, ask the user for all missing ones in a single message:

- **ticket_source** *(required)*: One of:
    - A JIRA ticket ID (e.g., `PROJ-123`) — will be fetched via API
    - A URL to a ticket or requirements page — will be fetched via HTTP
    - Plain text / pasted requirements — use directly

- **codebase_path** *(required)*: An absolute or relative directory path (e.g., `C:\Projects\MyApp` or `../myapp`). Do
  not default to `.` — ask explicitly if not provided. If the user provides something that is not a path (e.g., "yes",
  "ok", a sentence), reject it and ask again: "That doesn't look like a path. Please provide the directory path to your
  codebase (e.g., `C:\Projects\MyApp` or `.` for the current directory)."

- **output_format**: `html` or `md` (default: `html`)

- **test_command** *(optional)*: The command to run tests for this project (e.g., `npm test`, `pytest`,
  `go test ./...`). If not provided now, you will ask per-step during implementation.

- **scope** *(optional)*: One or more subdirectory paths relative to `codebase_path`, comma-separated (e.g.,
  `src/auth,src/api`). When provided, the codebase exploration and snapshot will be constrained to those directories
  only.

## Step 2 — Derive File Names

From the ticket title or first sentence of pasted text, generate a short slug:

- Lowercase the title
- Replace spaces and special characters with hyphens
- Strip leading/trailing hyphens
- Truncate to 40 characters max

Build the file prefix:

- JIRA ticket: `<ticket-id>_<slug>` (e.g., `PROJ-123_add-token-refresh`)
- URL source: `url_<slug>` (e.g., `url_add-status-command`)
- Pasted text: `pasted_<slug>` (e.g., `pasted_add-status-command`)

Derive paths:
- `report_path` = `<codebase_path>/.dev-workflow/<prefix>.<output_format>`
- `state_path` = `<codebase_path>/.dev-workflow/<prefix>_state.json`

> **Note on slug generation:** For JIRA tickets and URLs you don't yet know the title — use a temporary placeholder
> slug like `pending` for now. You will replace it once `ticket_analysis_agent` returns the actual title.

## Step 3 — Prepare State Directory

Create the output directory if it does not exist:

```bash
mkdir -p "<codebase_path>/.dev-workflow"
```

Then tell the user:
> "Created `.dev-workflow/` in your project root. Add this to your `.gitignore` to avoid committing session state:
> ```
> .dev-workflow/
> ```"

Check if `<codebase_path>/.gitignore` exists. If it does and `.dev-workflow/` is not already listed, offer:
> "Would you like me to add `.dev-workflow/` to your `.gitignore` automatically? (yes / no)"

If yes, append `.dev-workflow/` to the `.gitignore` file.

Also delete `<codebase_path>/.dev-workflow/codebase_context.md` if it exists — always regenerate fresh for each new ticket.

## Step 4 — Spawn ticket_analysis_agent

Use the Task tool to spawn `ticket_analysis_agent` with this prompt:

```
Analyze the following ticket source and return structured requirements JSON.

ticket_source: <ticket_source>
```

Wait for the agent to complete. It returns:

```json
{
  "source_type": "jira | url | pasted",
  "ticket_id": "PROJ-123 | null",
  "title": "string",
  "description": "string",
  "requirements": ["string"],
  "acceptance_criteria": ["string"],
  "type": "feature | bug | tech_debt | other",
  "notes": ["string"]
}
```

If the agent returns an error or empty content, stop and tell the user what went wrong.

Store the result as `requirements`.

**Now finalize the file prefix** using the real title from `requirements.title`. Rebuild `file_prefix`, `report_path`,
and `state_path` with the actual slug.

## Step 5 — Spawn report_generator_agent

Use the Task tool to spawn `report_generator_agent` with this prompt:

```
Generate a full analysis report and implementation plan.

requirements: <requirements JSON from Step 4>
codebase_path: <codebase_path>
output_format: <output_format>
file_prefix: <file_prefix>
report_path: <report_path>
scope: <scope or null>
```

Wait for the agent to complete. It returns:

```json
{
  "report_path": "string",
  "implementation_plan": [{ "step": 1, "title": "...", "description": "...", "files": [], "test_command": "...", "test_type": "...", "test_guidance": "...", "commit_message": "..." }],
  "open_questions": ["string"],
  "affected_files_count": 0,
  "step_count": 0,
  "test_strategy": {
    "detected_convention": "string | null",
    "test_framework": "string | null",
    "patterns_summary": "string",
    "steps_needing_tests": [{ "step": 1, "test_type": "unit | integration | update", "test_file": "path/to/file.test.ts" }]
  }
}
```

If the agent returns an error, stop and tell the user what went wrong.

Store the result as `report_result`.

## Step 6 — Persist State

Write `<state_path>`:

```json
{
  "phase": "review",
  "ticket_source": "<jira_id | url | 'pasted'>",
  "ticket_id": "<id if jira, else null>",
  "file_prefix": "<file_prefix>",
  "codebase_path": "<codebase_path>",
  "output_format": "<html|md>",
  "test_command": "<project test command, or null if not provided>",
  "scope": "<comma-separated scope dirs, or null>",
  "report_path": "<report_path>",
  "state_path": "<state_path>",
  "implementation_plan": "<report_result.implementation_plan>",
  "test_strategy": "<report_result.test_strategy>",
  "current_step": 0,
  "completed_steps": [],
  "review_iterations": 0
}
```

Write `<codebase_path>/.dev-workflow/active_state.json`:

```json
{ "state_path": "<state_path>" }
```

## Step 7 — Present to User

Show:

1. A 3–5 bullet summary of the most important findings from `report_result`
2. The report file path
3. The number of implementation steps identified
4. Any open questions flagged by the agent

Then say:
> "Review the report at `<report_path>`. Use `/submit-review-feedback` with your notes to refine the analysis,
> or `/approve-step` when you're ready to begin implementing."
