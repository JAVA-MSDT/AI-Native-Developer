You are executing the `/start-ticket-analysis` workflow. Follow every step in order.

## Step 1 — Collect Inputs

You need these inputs. If they were not passed as arguments, ask the user for all missing ones in a single message:

- **ticket_source** *(required)*: One of:
  - A JIRA ticket ID (e.g., `PROJ-123`) — will be fetched via API
  - A URL to a ticket or requirements page — will be fetched via HTTP
  - Plain text / pasted requirements — use directly

- **codebase_path** *(required)*: Path to the codebase root. Do not default to `.` — ask explicitly if not provided.

- **output_format**: `html` or `md` (default: `html`)

- **test_command** *(optional)*: The command to run tests for this project (e.g., `npm test`, `pytest`, `go test ./...`). If not provided now, you will ask per-step during implementation.

## Step 2 — Fetch or Parse the Requirements

Determine the ticket source type:

**JIRA ticket ID** (matches pattern like `PROJ-123`, `ABC-456`):
Use environment variables `JIRA_URL`, `JIRA_USERNAME`, `JIRA_API_KEY`:
```bash
curl -s -u "$JIRA_USERNAME:$JIRA_API_KEY" \
  "$JIRA_URL/rest/api/3/issue/$TICKET_SOURCE" \
  -H "Accept: application/json"
```
Parse the response to extract: summary, description, acceptance criteria, linked issues.
If env vars are missing or the call fails, fall through to the "pasted text" path below.

**URL** (starts with `http://` or `https://`):
Fetch the page content. Extract the main requirements text — look for headings like "Description", "Acceptance Criteria", "Requirements", "Definition of Done".

**Pasted text / free-form requirements**:
Use the provided text directly as the requirements. Parse it for acceptance criteria markers like "AC:", "- [ ]", "Must", "Should".

In all cases, if the content is ambiguous or acceptance criteria are not clearly identifiable, note this as an open question in the report.

## Step 3 — Explore the Codebase

Using Read, Glob, and Grep on `codebase_path`:

1. Map the top-level project structure — understand what kind of project this is, what language/framework, and where the key entry points are.
2. For each requirement or acceptance criterion from the ticket:
   - Use Glob to find relevant files by name patterns
   - Use Grep to find relevant functions, classes, or keywords
   - Read the most relevant files to understand the implementation
   - Trace callsites and dependencies
3. Document: affected files, functions, modules, and any third-party dependencies touched.

## Step 4 — Build the Implementation Plan

Create a numbered, ordered list of self-contained implementation steps. Each step must:
- Have one clear, single purpose
- Only touch the files listed for that step
- Be independently committable and verifiable
- Include a test command or validation criteria

Represent each step as a JSON object:
```json
{
  "step": 1,
  "title": "Short imperative title",
  "description": "What changes and why",
  "files": ["path/to/file1.ts", "path/to/file2.ts"],
  "test_command": "npm test -- --testPathPattern=auth",
  "commit_message": "feat: add token refresh logic to auth service"
}
```

## Step 5 — Generate the Report

Ensure `.claude/state/` directory exists:
```bash
mkdir -p .claude/state
```

Write the report to:
- `.claude/state/analysis_report.html` if output_format is `html`
- `.claude/state/analysis_report.md` if output_format is `md`

### HTML Report Structure

A complete, self-contained HTML file with inline CSS. Include:

- **Header**: Ticket ID, title, date generated
- **Section 1 — Ticket Summary**: Requirements and acceptance criteria listed clearly
- **Section 2 — Codebase Analysis**: Table of affected files with their role and how they're impacted
- **Section 3 — Risk Assessment**: Risks color-coded by severity (green = low, yellow = medium, red = high/breaking)
- **Section 4 — Implementation Plan**: Numbered steps with title, description, files, test command, and commit message
- **Section 5 — Open Questions**: Anything ambiguous that needs user clarification before implementation

Use clean inline CSS: dark header, white body, code blocks with light gray background, clear section borders.

### Markdown Report Structure

Use `##` headings, tables for affected files, fenced code blocks for file paths and snippets.

## Step 6 — Persist State

Write `.claude/state/workflow_state.json`:
```json
{
  "phase": "review",
  "ticket_source": "<jira_id | url | 'pasted'>",
  "ticket_id": "<id if jira, else null>",
  "codebase_path": "<path>",
  "output_format": "<html|md>",
  "test_command": "<project test command, or null if not provided>",
  "report_path": ".claude/state/analysis_report.<ext>",
  "implementation_plan": [<array of step objects from Step 4>],
  "current_step": 0,
  "completed_steps": [],
  "git_commits": [],
  "review_iterations": 0
}
```

## Step 7 — Present to User

Show:
1. A 3–5 bullet summary of the most important findings
2. The report file path
3. The number of implementation steps identified

Then say: "Review the report at `<report_path>`. Use `/submit-review-feedback` with your notes to refine the analysis, or `/approve-step` when you're ready to begin implementing."
