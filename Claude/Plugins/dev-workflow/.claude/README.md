# Dev Workflow Plugin

---

## About

Automates ticket analysis, iterative review, and step-by-step implementation with Human-In-The-Loop (HITL) approval using Claude agents, commands, and skills.

This plugin mirrors exactly how a senior developer handles a ticket: understand it deeply, validate the analysis, then implement in small verified steps. Every decision that matters requires your explicit approval.

```
PHASE 1 — ANALYSIS
─────────────────────────────────────────────────────────────────────
You           → Run /start-ticket-analysis
              → Provide: ticket (JIRA ID / URL / pasted text),
                         codebase path, output format (html/md)

Claude        → Fetches or reads the requirements
              → Explores the codebase (Glob, Grep, Read)
              → Maps affected files, risks, edge cases
              → Builds a numbered implementation plan
              → Writes report to .claude/state/analysis_report.html
              → Saves state to .claude/state/workflow_state.json

You           → Open and read the report
              → Identify gaps, unclear areas, missing scenarios

PHASE 2 — REVIEW LOOP  (repeat until satisfied)
─────────────────────────────────────────────────────────────────────
You           → Run /submit-review-feedback
              → Paste your findings:
                "The auth edge case for expired tokens is missing.
                 Section 3 doesn't cover the background job impact."

Claude        → Re-analyzes the specific areas you flagged
              → Updates the report in place (adds "Review Iteration N")
              → Revises the implementation plan if needed

You           → Review the updated report
              → Repeat as many times as needed
              → Stop when you are satisfied with the analysis

PHASE 3 — IMPLEMENTATION  (one step at a time)
─────────────────────────────────────────────────────────────────────
You           → Run /approve-step

Claude        → Shows you exactly what step N will do:
                title, files to touch, test command, commit message
              → Asks for confirmation before touching any code

You           → Confirm (or ask to show files first)

Claude        → Implements only that step
              → Runs tests
              → If tests pass: commits with the planned commit message
              → Reports commit hash and test results
              → Waits — does NOT auto-proceed to step N+1

You           → Review the diff, run your own checks if needed
              → Run /approve-step again for the next step
              → OR run /rollback-step if something is wrong

ROLLBACK  (any time during implementation)
─────────────────────────────────────────────────────────────────────
You           → Run /rollback-step (or /rollback-step 2 for a specific step)

Claude        → Shows which commit will be reverted
              → Asks for confirmation

You           → Confirm

Claude        → Runs git revert — original commit stays in history
              → Updates state so /approve-step resumes from the right place
```

---

## Features

- Accepts requirements from any source: JIRA ticket ID, URL, or pasted text
- Analyzes the codebase against requirements and generates a full HTML or Markdown report
- Builds a structured, self-contained implementation plan as part of the report
- Supports unlimited review/feedback iterations — report updates in place
- Implements code step-by-step, one commit per step, with HITL confirmation before each
- Rollback any step with `git revert` — git history is always preserved
- Persists workflow state in `.claude/state/workflow_state.json` — survives session restarts

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| AI runtime | [Claude Code](https://claude.ai/code) — Claude Sonnet / Opus |
| Agent orchestration | Claude Code subagents (`.claude/agents/`) |
| Commands | Claude Code slash commands (`.claude/commands/`) |
| Ticket source | JIRA REST API v3, any URL, or plain text |
| Version control | Git (`git add`, `git commit`, `git revert`) |
| Report output | Self-contained HTML (inline CSS) or Markdown |
| State persistence | JSON file (`.claude/state/workflow_state.json`) |

No external packages or build tools required — this is a prompt-only plugin.

---

## Project Structure

```
dev-workflow/
├── CLAUDE.md                        ← Plugin entry point (read by Claude Code)
└── .claude/
    ├── README.md                    ← This file
    ├── settings.json                ← Permissions and hooks
    ├── agents/
    │   ├── orchestrator_agent.md    ← Coordinates all phases and state
    │   ├── ticket_analysis_agent.md ← Fetches/parses requirements from any source
    │   ├── report_generator_agent.md← Analyzes codebase, builds report + impl plan
    │   ├── reanalysis_agent.md      ← Targeted re-analysis for reviewer feedback
    │   └── implementation_agent.md  ← Executes one step, runs tests, commits
    ├── commands/
    │   ├── start_ticket_analysis.md ← /start-ticket-analysis
    │   ├── submit_review_feedback.md← /submit-review-feedback
    │   ├── approve_step.md          ← /approve-step
    │   └── rollback_step.md         ← /rollback-step
    └── skills/
        ├── fetch_requirements.md    ← JIRA API / URL fetch / paste fallback
        ├── analyze_codebase.md      ← Glob + Grep + Read patterns for analysis
        ├── generate_html_report.md  ← HTML report structure and inline CSS spec
        ├── generate_md_report.md    ← Markdown report structure spec
        └── implement_code_change.md ← Edit + test + commit logic and rollback
```

---

## Getting Started

### 1. Open the plugin directory in Claude Code

Claude Code must be opened with `dev-workflow/` as the working directory so it loads the `.claude/` config.

```bash
cd path/to/dev-workflow
claude
```

Or open it as the root in VS Code / JetBrains with the Claude Code extension.

### 2. Verify the plugin loads

Type `/` in the Claude Code prompt. You should see all four commands:
- `/start-ticket-analysis`
- `/submit-review-feedback`
- `/approve-step`
- `/rollback-step`

If any are missing, check that `.claude/commands/` contains the matching `.md` files.

### 3. Run your first analysis

```
/start-ticket-analysis
```

Claude will ask for:
- **ticket_source** — a JIRA ID (e.g., `PROJ-123`), a URL, or paste your requirements directly
- **codebase_path** — path to the repo you want analyzed
- **output_format** — `html` (default) or `md`
- **test_command** *(optional)* — e.g., `npm test`, `pytest`, `go test ./...`

### Using This Plugin in Another Project

The plugin is project-agnostic. It analyzes whatever codebase you point it at.

**Option A — Copy the plugin alongside your project**

```
your-project/
├── src/
├── tests/
├── package.json
└── dev-workflow/        ← copy the plugin here
    ├── CLAUDE.md
    └── .claude/
```

Open Claude Code with `dev-workflow/` as the working directory, then pass your project root as `codebase_path`:
```
/start-ticket-analysis codebase_path=../  test_command="npm test"
```

**Option B — Copy `.claude/` directly into your project**

If you want the commands available while working inside your project:

```bash
cp -r dev-workflow/.claude your-project/.claude
```

Then open your project in Claude Code normally. The four commands will appear as slash commands.

> **Note:** Add `.claude/state/` to your `.gitignore`. State files are session-specific and should not be committed.

### What you need to provide

| Input | Description | Required |
|-------|-------------|----------|
| `ticket_source` | JIRA ID, URL, or pasted text | Yes |
| `codebase_path` | Path to the repo root you want analyzed | Yes |
| `output_format` | `html` (default) or `md` | No |
| `test_command` | e.g. `npm test`, `pytest`, `go test ./...` | No (asked per step if missing) |

### Testing the Plugin

**End-to-end smoke test (no real project needed)**

Use the plugin against its own directory as a self-referential test:

```
/start-ticket-analysis
  ticket_source = "Add a /status command that shows the current workflow state"
  codebase_path = .
  output_format = html
  test_command  = echo "no tests"
```

Expected results:
- Claude asks for any missing inputs
- Claude reads files in `.claude/`
- A report is created at `.claude/state/analysis_report.html`
- The report includes affected files, an implementation plan with at least 1 step, and section headings
- `.claude/state/workflow_state.json` exists and contains `"phase": "review"`

**Test the review loop**

After the initial analysis:
```
/submit-review-feedback
  findings = "The report doesn't mention what happens if workflow_state.json is corrupted"
```

Expected:
- Claude revisits the relevant command/agent files
- The report gains a "Review Iteration 1" section
- The open questions or risk sections are updated
- `workflow_state.json` shows `"review_iterations": 1`

**Test implementation gating**

```
/approve-step
```

Expected:
- Claude shows step details and asks for confirmation
- Claude does NOT implement anything until you say yes
- After implementation, `workflow_state.json` shows `"current_step": 1` and a commit hash in `completed_steps`

**Test rollback**

After at least one step is committed:
```
/rollback-step
```

Expected:
- Claude shows which commit will be reverted and asks for confirmation
- `git log` shows a new revert commit
- `workflow_state.json` shows `completed_steps` with the rolled-back step removed

**Verify state persistence**

After any command, close and reopen Claude Code in the same directory. Run `/approve-step`. Claude should read `workflow_state.json` and pick up exactly where you left off, without needing to re-run the analysis.

---

## Configuration

### JIRA credentials

Only needed if you use JIRA ticket IDs. Set these environment variables before starting:

```bash
export JIRA_URL=https://yourcompany.atlassian.net
export JIRA_USERNAME=your.email@company.com
export JIRA_API_KEY=your_api_token    # from id.atlassian.com → Security → API tokens
```

If you don't have JIRA, just paste the requirements text when prompted — the plugin works the same way.

### Test runner

You can tell the plugin your project's test command upfront:
```
/start-ticket-analysis test_command="npm test"
```
Or leave it out — Claude will ask per step during implementation.

### Permissions and hooks (`settings.json`)

The plugin's `settings.json` grants these permissions automatically:

| Permission | Purpose |
|------------|---------|
| `Read(*)`, `Write(*)`, `Edit(*)` | Codebase analysis and report generation |
| `Bash(curl:*)` | Fetching JIRA tickets |
| `Bash(git add/commit/revert/log/status/diff/stash:*)` | Committing and rolling back steps |
| `Bash(mkdir/cat/echo:*)` | Creating the state directory and logging |

A `PostToolUse` hook logs a timestamped line to the terminal after every subagent `Task` call.

### State management

Workflow state is written to `.claude/state/workflow_state.json`. Key fields:

| Field | Description |
|-------|-------------|
| `phase` | `review` or `implementation` |
| `ticket_source` | The original ticket input |
| `codebase_path` | The analyzed repo path |
| `implementation_plan` | Array of step objects (title, files, test_command, commit_message) |
| `current_step` | Last completed step number |
| `completed_steps` | Array of completed steps with commit hashes |
| `review_iterations` | Number of feedback cycles completed |

**To reset:** delete `.claude/state/workflow_state.json`. Back up the report files first if you want to keep them.

### User advice

**Before you start**

- **Commit a clean state first.** The plugin creates commits during implementation. Starting from a dirty working tree makes rollback much harder. Run `git status` and commit or stash anything in progress.
- **Be specific with `codebase_path`.** Point it at the actual repository root — not a parent directory with many unrelated projects.

**During analysis**

- **The more specific your findings, the better the re-analysis.** Instead of "the auth section needs more detail", say "the report doesn't cover what happens when the refresh token is expired and the user has an active request in flight."
- **You can iterate as many times as you need.** There is no limit on `/submit-review-feedback` cycles. Stop only when you trust the implementation plan.
- **Trust the open questions section.** If the report flags something as an open question, answer it before approving implementation. Open questions during analysis become bugs during implementation.

**During implementation**

- **Read each step before confirming.** Claude will show you the step details and ask for confirmation. If something looks wrong, say no and use `/submit-review-feedback` to revise the plan.
- **Each step is one commit.** This is intentional. Small commits make it easy to `git bisect`, rollback individual steps, and understand the change history.
- **If tests fail, don't skip them lightly.** Claude will ask whether to fix, skip, or rollback. Skipping tests defeats the purpose of the gating.
- **`/rollback-step` is safe.** It uses `git revert` — it never rewrites history. You can rollback, revise the plan, and re-implement.

**State and sessions**

- **State survives session restarts.** If you close Claude Code mid-workflow, re-open in the same directory and continue.
- **Don't commit `.claude/state/` to your repo.** State files are session-specific and contain paths and plans that only make sense in your local context.

---

## License

MIT

---

## Author

Ahmed Samy
