You are executing the `/start-ticket-analysis` workflow. Follow every step in order.

## Step 1 — Collect Inputs

You need these inputs. If they were not passed as arguments, ask the user for all missing ones in a single message:

- **ticket_source** *(required)*: One of:
    - A JIRA ticket ID (e.g., `PROJ-123`) — will be fetched via API
    - A URL to a ticket or requirements page — will be fetched via HTTP
    - Plain text / pasted requirements — use directly

- **codebase_path** *(required)*: An absolute or relative directory path (e.g., `C:\Projects\MyApp` or `../myapp`). Do
  not default to `.` — ask explicitly if not provided. If the user provides something that is not a path (e.g., "yes", "
  ok", a sentence), reject it and ask again: "That doesn't look like a path. Please provide the directory path to your
  codebase (e.g., `C:\Projects\MyApp` or `.` for the current directory)."

- **output_format**: `html` or `md` (default: `html`)

- **test_command** *(optional)*: The command to run tests for this project (e.g., `npm test`, `pytest`,
  `go test ./...`). If not provided now, you will ask per-step during implementation.

- **scope** *(optional)*: One or more subdirectory paths relative to `codebase_path`, comma-separated (e.g.,
  `src/auth,src/api`). When provided, the codebase exploration and snapshot will be constrained to those directories
  only. Use this for large repos where the ticket clearly affects a known subsystem. If not provided, the full codebase
  is analyzed.

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
Fetch the page content. Extract the main requirements text — look for headings like "Description", "Acceptance
Criteria", "Requirements", "Definition of Done".

**Pasted text / free-form requirements**:
Use the provided text directly as the requirements. Parse it for acceptance criteria markers like "AC:", "- [ ]", "
Must", "Should".

In all cases, if the content is ambiguous or acceptance criteria are not clearly identifiable, note this as an open
question in the report.

## Step 3 — Derive File Names

From the ticket title (or first sentence of pasted text), generate a short slug:

- Lowercase the title
- Replace spaces and special characters with hyphens
- Strip leading/trailing hyphens
- Truncate to 40 characters max

Build the file prefix:

- JIRA ticket: `<ticket-id>_<slug>` (e.g., `PROJ-123_add-token-refresh`)
- URL source: `url_<slug>` (e.g., `url_add-status-command`)
- Pasted text: `pasted_<slug>` (e.g., `pasted_add-status-command`)

Report file: `<prefix>.<ext>` (e.g., `PROJ-123_add-token-refresh.html`)
State file: `<prefix>_state.json` (e.g., `PROJ-123_add-token-refresh_state.json`)

Both files will live in `<codebase_path>/.dev-workflow/`.

## Step 4 — Explore the Codebase

Determine the exploration root:

- If `scope` was provided, explore only `<codebase_path>/<scope_dir>` for each directory listed. Note the scope
  constraint in the snapshot header.
- If `scope` was not provided, explore from `codebase_path` (full codebase).

Using Read, Glob, and Grep on the exploration root:

1. Map the top-level project structure — understand what kind of project this is, what language/framework, and where the
   key entry points are.
2. For each requirement or acceptance criterion from the ticket:
    - Use Glob to find relevant files by name patterns
    - Use Grep to find relevant functions, classes, or keywords
    - Read the most relevant files to understand the implementation
    - Trace callsites and dependencies
3. Document: affected files, functions, modules, and any third-party dependencies touched. Hold these findings in
   memory — the snapshot will be written in Step 6 once the state directory exists.

## Step 5 — Build the Implementation Plan

Create a numbered, ordered list of self-contained implementation steps. Each step must:

- Have one clear, single purpose
- Only touch the files listed for that step
- Be independently committable and verifiable
- Include a test command or validation criteria
- Include a `test_type` assessment and `test_guidance` where applicable (populated in Step 5.5 below)

**For each step, assess whether it requires tests:**

| Change type                               | Test required?                  | test_type       |
| ----------------------------------------- | ------------------------------- | --------------- |
| New public function, class, or module     | Yes — unit test                 | `"unit"`        |
| New API endpoint or DB query/mutation     | Yes — integration test          | `"integration"` |
| Modification to existing public interface | Yes — update existing test      | `"update"`      |
| New UI component with logic/validation    | Yes — unit test                 | `"unit"`        |
| Config / build / tooling / docs only      | No                              | `"none"`        |
| Pure refactor with no behavior change     | Verify with existing tests only | `"none"`        |

Represent each step as a JSON object:

```json
{
  "step": 1,
  "title": "Short imperative title",
  "description": "What changes and why",
  "files": ["path/to/file1.ts", "path/to/file2.ts"],
  "test_command": "npm test -- --testPathPattern=auth",
  "test_type": "unit | integration | update | none",
  "test_guidance": "1–3 sentences on what to test and which patterns to follow — populated after Step 5.5",
  "commit_message": "feat: add token refresh logic to auth service"
}
```

Draft steps first with `test_type: null`. Step 5.5 (below) will populate `test_type`, `test_guidance`, and add test
file paths to `files[]`.

## Step 5.5 — Discover Test Patterns

After drafting steps, explore the test infrastructure to fill in `test_type`, `test_guidance`, and test file paths.

**Step A — Detect test files.** Run these Glob patterns against `codebase_path` in order; stop at the first that
returns results:

1. `**/*.test.ts` or `**/*.spec.ts`
2. `**/*.test.js` or `**/*.spec.js`
3. `**/*Test.java` or `**/*Spec.java`
4. `**/test_*.py` or `**/*_test.py`
5. `**/*_spec.rb`
6. `**/tests/**/*`, `**/__tests__/**/*`, `**/spec/**/*`

Record the matching pattern as `detected_test_convention`.

**Step B — Classify unit vs integration.** Among discovered files, classify as integration/e2e if the path includes
`integration/`, `e2e/`, or `contract/` or the filename contains those words. Remainder = unit tests.

**Step C — Read representative samples.** Read at most 1–2 unit test files + 1–2 integration test files (smallest
files first). Extract: test framework, assertion library, mock strategy (`jest.mock`, `sinon`, `unittest.mock`, etc.),
setup/teardown patterns (`beforeEach`, fixtures), file naming convention relative to source files.
Record as `test_patterns_summary`.

**Step D — If no test files found:** Set `detected_test_convention = null`, `test_patterns_summary = "No existing
tests found."`, and add to open questions: "No test files were found — what test framework and conventions should be
used?" Do not add test files to `files[]` for any step.

**Step E — Back-fill step objects.** For each step with a non-`"none"` test_type:
- Compute the test file path using `detected_test_convention`. If null, skip (open question already added).
- Add the test file to the step's `files[]`.
- Write `test_guidance` referencing the patterns found: framework name, describe/it structure, mock approach,
  the 2–3 specific scenarios to cover.

## Step 6 — Create the State Directory and Write Snapshot

Create `<codebase_path>/.dev-workflow/` if it does not exist:

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

**Write the codebase snapshot.** Delete `<codebase_path>/.dev-workflow/codebase_context.md` if it already exists (always
regenerate — never reuse a stale snapshot from a previous run). Then write a fresh one using the findings from Step 4:

```markdown
# Codebase Context
Generated: <date> | Ticket: <file_prefix>

## Tech Stack
<detected language, framework, test runner, key dependencies — one line each>

## Relevant File Map
| File         | Purpose      | Key symbols                  |
| ------------ | ------------ | ---------------------------- |
| path/to/file | what it does | exported functions / classes |

## Key Patterns
<architecture patterns, naming conventions, auth approach, DB layer, error handling — bullet points>

## Entry Points
<files that bootstrap the app, register routes, or are the main execution entry>
```

This snapshot is the single source of codebase context for all subsequent agents — they read it instead of re-exploring
the codebase.

## Step 7 — Generate the Report

Write the report to `<codebase_path>/.dev-workflow/<prefix>.<ext>`.

### HTML Report Structure

A complete, self-contained HTML file with inline CSS. Include:

- **Header**: Ticket ID (or source type), title, date generated
- **Section 1 — Ticket Summary**: Requirements and acceptance criteria listed clearly
- **Section 2 — Codebase Analysis**: Table of affected files with their role and how they're impacted
- **Section 3 — Risk Assessment**: Risks color-coded by severity (green = low, yellow = medium, red = high/breaking)
- **Section 4 — Implementation Plan**: Numbered steps with title, description, files, test command, and commit message.
  If a step's `test_type` is not `"none"`: show a colored badge for the type and a test-guidance block.
- **Section 5 — Test Strategy**: Detected test framework and conventions; table mapping each step to test type, test
  file, and approach; summary of patterns from existing tests. Omit if no test files were found and no steps need tests.
- **Section 6 — Open Questions**: Anything ambiguous that needs user clarification before implementation

Use clean inline CSS: dark header, white body, code blocks with light gray background, clear section borders.

### Markdown Report Structure

Use `##` headings, tables for affected files, fenced code blocks for file paths and snippets. Include a
`## Test Strategy` section between `## Implementation Plan` and `## Open Questions`.

## Step 8 — Persist State

Write `<codebase_path>/.dev-workflow/<prefix>_state.json`:

```json
{
  "phase": "review",
  "ticket_source": "<jira_id | url | 'pasted'>",
  "ticket_id": "<id if jira, else null>",
  "file_prefix": "<prefix>",
  "codebase_path": "<path>",
  "output_format": "<html|md>",
  "test_command": "<project test command, or null if not provided>",
  "scope": "<comma-separated scope dirs, or null if full codebase>",
  "report_path": "<codebase_path>/.dev-workflow/<prefix>.<ext>",
  "state_path": "<codebase_path>/.dev-workflow/<prefix>_state.json",
  "implementation_plan": [<array of step objects from Step 5 — each with test_type and test_guidance fields>],
  "test_strategy": {
    "detected_convention": "<string | null>",
    "test_framework": "<string | null>",
    "patterns_summary": "<string>",
    "steps_needing_tests": [{ "step": 1, "test_type": "unit | integration | update", "test_file": "path/to/file.test.ts" }]
  },
  "current_step": 0,
  "completed_steps": [],
  "review_iterations": 0
}
```

Also write a pointer file at `<codebase_path>/.dev-workflow/active_state.json` containing just:

```json
{ "state_path": "<codebase_path>/.dev-workflow/<prefix>_state.json" }
```

This allows other commands to find the current active analysis without knowing the prefix.

## Step 9 — Present to User

Show:

1. A 3–5 bullet summary of the most important findings
2. The report file path
3. The number of implementation steps identified

Then say: "Review the report at `<report_path>`. Use `/submit-review-feedback` with your notes to refine the analysis,
or `/approve-step` when you're ready to begin implementing."
