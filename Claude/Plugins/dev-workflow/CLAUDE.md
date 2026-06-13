# Dev Workflow Plugin

Automates ticket analysis, iterative review, and step-by-step implementation with Human-In-The-Loop (HITL) approval.

## Features

- Accepts requirements from any source: JIRA ticket ID, URL, or pasted text
- Analyzes the codebase against requirements and generates a full HTML or Markdown report
- Names each report after the ticket ID and title slug (e.g., `PROJ-123_add-token-refresh.html`)
- Builds a structured, self-contained implementation plan as part of the report
- Supports unlimited review/feedback iterations — report updates in place
- Implements code step-by-step with HITL confirmation before each step and before committing
- Rollback any step with `git revert` — git history is always preserved
- Writes state to `<your-project>/.dev-workflow/` — never inside the plugin itself

## Usage

```
/start-ticket-analysis   — Fetch ticket, analyze codebase, generate report
/submit-review-feedback  — Re-analyze based on your findings, update report
/approve-step            — Implement the next step and get the commit command
/rollback-step           — Revert the last committed step
```

## How to Use This Plugin

Open Claude Code with this `dev-workflow/` directory as the working directory. When running `/start-ticket-analysis`, point `codebase_path` at the project you want to analyze. The plugin works on that project without touching its `.claude/` configuration.

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
