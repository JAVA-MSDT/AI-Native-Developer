# Dev Workflow Plugin

Automates ticket analysis, iterative review, and step-by-step implementation with Human-In-The-Loop (HITL) approval.

## Features

- Accepts requirements from any source: JIRA ticket ID, URL, or pasted text
- Analyzes the codebase against requirements and generates a full HTML or Markdown report
- Builds a structured, self-contained implementation plan as part of the report
- Supports unlimited review/feedback iterations — report updates in place
- Implements code step-by-step, one commit per step, with HITL confirmation before each
- Rollback any step with `git revert` — git history is always preserved
- Persists workflow state in `.claude/state/workflow_state.json` — survives session restarts

## Usage

```
/start-ticket-analysis   — Fetch ticket, analyze codebase, generate report
/submit-review-feedback  — Re-analyze based on your findings, update report
/approve-step            — Implement and commit the next step (after review)
/rollback-step           — Revert the last committed step
```

## Setup

For JIRA integration, set these environment variables before starting:
```
JIRA_URL=https://yourcompany.atlassian.net
JIRA_USERNAME=your.email@company.com
JIRA_API_KEY=your_api_token
```

If JIRA env vars are not set, you can paste requirements directly when prompted.

For the test runner, either pass `test_command` to `/start-ticket-analysis` or you will be asked per-step.

## State

Workflow state is written to `.claude/state/workflow_state.json`. Delete this file to reset and start a new analysis.
