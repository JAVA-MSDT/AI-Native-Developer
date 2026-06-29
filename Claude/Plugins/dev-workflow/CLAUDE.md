# Dev Workflow Plugin

Automates ticket analysis, iterative review, and step-by-step implementation with Human-In-The-Loop (HITL) approval.

## Features

- Accepts requirements from any source: JIRA ticket ID, URL, or pasted text
- Optional `scope` parameter limits analysis to specific subdirectories — reduces tokens on large repos
- Analyzes the codebase and generates a full HTML or Markdown report named after the ticket (e.g.,
  `PROJ-123_add-token-refresh.html`)
- Writes a `codebase_context.md` snapshot — reused by review iterations to avoid redundant file reads; always
  regenerated fresh on each new ticket
- `/refresh-snapshot` re-explores the codebase and rewrites the snapshot without touching the state file, report, or
  implementation plan — use after a significant `git pull` mid-ticket
- Builds a structured, self-contained implementation plan as part of the report
- **Token-efficient plan generation** — consolidates steps that touch the same file into one, and eliminates trivially
  small steps (single annotations, variable declarations, import lines) by absorbing them into adjacent substantive
  steps; every step in the final plan represents real implementation work
- Validates state file integrity on every command — clear error if a previous run failed mid-way
- **Collision guard on `/start-ticket-analysis`** — before doing any work, detects if an active session already exists
  for the same ticket and prompts to continue, start fresh (by deleting `.dev-workflow/`), or cancel; prevents
  accidentally discarding in-progress analysis mid-implementation
- Supports unlimited review/feedback iterations — report updates in place
- Shows `git diff` after each implementation step so review is based on real changes
- HITL confirmation before each step; developer runs git commands manually — after each step choose **yes** (committed now), **later** (batch with upcoming steps), or **not yet** (re-prompt); uncommitted steps are tracked and surfaced at MR description time
- **Mandatory test coverage check on every implementation step** — after each code change the agent independently
  assesses whether tests are needed, searches for existing test files first, and either updates them or creates new
  ones; no step completes without an explicit test verdict
- Generates a ready-to-paste Merge Request description when all steps are complete — title formatted as
  `[Story Number] Story title`, with sections for what the task was about, what was done, and what test approaches
  were used
- Rollback any step with `git revert` — git history is always preserved
- Writes all state and reports to `<your-project>/.dev-workflow/` — never inside the plugin itself

## Usage

```
/start-ticket-analysis   — Fetch ticket, analyze codebase, generate report
/submit-review-feedback  — Re-analyze based on your findings, update report
/approve-step            — Implement the next step and get the commit command
/rollback-step           — Revert the last committed step
/refresh-snapshot        — Re-explore codebase and update snapshot after a git pull
/status                  — Show current workflow state
```

## How to Use This Plugin

Open Claude Code with this `dev-workflow/` directory as the working directory. When running `/start-ticket-analysis`,
point `codebase_path` at the project you want to analyze. The plugin works on that project without touching its
`.claude/` configuration.

```
dev-workflow/     ← open Claude Code here
your-project/     ← passed as codebase_path
  .dev-workflow/  ← state and reports written here (add to .gitignore)
```

## Setup

For JIRA integration, set these environment variables:

```
JIRA_URL=https://yourcompany.atlassian.net
JIRA_USERNAME=your.email@company.com
JIRA_API_KEY=your_api_token
```

If JIRA env vars are not set, paste requirements directly when prompted.
