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
              → Writes codebase snapshot to .dev-workflow/codebase_context.md
              → Writes report to .dev-workflow/<prefix>.html
              → Saves state to .dev-workflow/<prefix>_state.json

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
              → Shows git diff of actual changes
              → Asks: "Does the implementation look correct?"

You           → Review the code
              → Reply "looks good" OR provide corrections

Claude        → If corrections: fixes the code, re-runs tests, asks again
              → If looks good: shows the exact git commit command to run

You           → Run the commit command yourself
              → Run /approve-step for the next step
              → OR run /rollback-step if something needs to be undone
              → On the last step: receive a ready-to-paste PR description

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
- Optional `scope` parameter constrains analysis to specific subdirectories — essential for large repos
- Analyzes the codebase against requirements and generates a full HTML or Markdown report
- Writes a `codebase_context.md` snapshot — reused by review iterations to avoid redundant file reads
- Builds a structured, self-contained implementation plan as part of the report
- Supports unlimited review/feedback iterations — report updates in place
- Shows `git diff` after each implementation step — review real changes, not Claude's description
- Implements code step-by-step with HITL confirmation before each step; developer runs git commands manually
- Generates a ready-to-paste PR description when all steps are complete
- Rollback any step with `git revert` — git history is always preserved
- Validates state file integrity on every command — clear error if a previous run failed mid-way
- Persists workflow state in `<your-project>/.dev-workflow/` — survives session restarts

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
| State persistence | JSON files in `<your-project>/.dev-workflow/` |

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

Type `/` in the Claude Code prompt. You should see all five commands:
- `/start-ticket-analysis`
- `/submit-review-feedback`
- `/approve-step`
- `/rollback-step`
- `/status`

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
- **scope** *(optional)* — comma-separated subdirectories to limit analysis (e.g., `src/auth,src/api`). Omit for full codebase analysis. Use for large repos where the ticket affects a known subsystem.

### Using This Plugin in Another Project

The plugin is a self-contained Claude Code project. You open it in its own directory and point it at whichever project you want to analyze. **It never touches the target project's `.claude/` configuration** — this means it can sit alongside any other Claude Code plugins or project settings without conflict.

```
dev-workflow/          ← open Claude Code here
  .claude/             ← plugin's own commands, agents, settings
  CLAUDE.md

your-project/          ← passed as codebase_path
  src/
  .claude/             ← your project's own config — untouched
  .dev-workflow/       ← state and reports written here by the plugin
```

Open Claude Code in the `dev-workflow/` directory, then provide your project path when prompted:
```
/start-ticket-analysis
  codebase_path = C:\Projects\your-project
  test_command  = npm test
```

The plugin writes all state and reports into `your-project/.dev-workflow/` — not inside itself. Each analysis gets its own named files (`PROJ-123_add-token-refresh.html`, `PROJ-123_add-token-refresh_state.json`), so you can run multiple analyses without overwriting anything.

> **Do not copy `.claude/` into your project.** That would overwrite your project's own Claude Code settings. The plugin is designed to be used as a separate directory.

### What you need to provide

| Input | Description | Required |
|-------|-------------|----------|
| `ticket_source` | JIRA ID, URL, or pasted text | Yes |
| `codebase_path` | Path to the repo root you want analyzed | Yes |
| `output_format` | `html` (default) or `md` | No |
| `test_command` | e.g. `npm test`, `pytest`, `go test ./...` | No (asked per step if missing) |

### Testing the Plugin

All tests below use the plugin against its own directory — no external project needed.

---

#### Test 1 — Initial analysis (smoke test)

```
/start-ticket-analysis
  ticket_source = "Add a /status command that shows the current workflow state"
  codebase_path = .
  output_format = html
  test_command  = echo "no tests"
```

**What to verify:**
- Claude asks for any missing inputs before proceeding
- Claude does NOT accept "yes" or "ok" as a `codebase_path` — rejects and asks again
- `.dev-workflow/` folder is created with a `.gitignore` prompt
- Report created at `.dev-workflow/pasted_add-status-command.html`
- Report contains: ticket summary, affected files table, risk section, numbered implementation plan, open questions
- `.dev-workflow/pasted_add-status-command_state.json` exists and contains `"phase": "review"`, `"current_step": 0`, non-empty `implementation_plan`
- `.dev-workflow/active_state.json` contains `"state_path"` pointing to the state file above
- `.dev-workflow/codebase_context.md` exists and contains tech stack, file map, and key patterns

---

#### Test 2 — Scoped analysis (large repo simulation)

```
/start-ticket-analysis
  ticket_source = "Improve error handling in commands"
  codebase_path = .
  scope         = .claude/commands
  output_format = html
  test_command  = echo "no tests"
```

**What to verify:**
- Claude only explores `.claude/commands/` — does not read agent or skill files
- The snapshot `codebase_context.md` header shows the scope constraint
- Report's affected files list only contains files under `.claude/commands/`

---

#### Test 3 — Review loop

After Test 1, run:
```
/submit-review-feedback
  findings = "The report doesn't mention what happens if workflow_state.json is corrupted or partially written"
```

**What to verify:**
- Claude reads `codebase_context.md` first (not re-exploring from scratch)
- Claude does targeted reads of only the relevant command files
- Report gains a `## Review Iteration 1` section near the top
- Open questions or risk sections are updated
- State file shows `"review_iterations": 1`
- `codebase_context.md` is NOT rewritten (it should only be regenerated by `/start-ticket-analysis`)

Run a second iteration:
```
/submit-review-feedback
  findings = "Also check how rollback handles the case where git revert has merge conflicts"
```

**What to verify:**
- Report gains `## Review Iteration 2`
- State file shows `"review_iterations": 2`

---

#### Test 4 — State validation

Manually corrupt the state file — open `pasted_add-status-command_state.json` and delete the `codebase_path` field. Then run:
```
/approve-step
```

**What to verify:**
- Claude stops immediately with a clear error: "The state file is incomplete (missing: `codebase_path`)"
- Claude does NOT attempt to implement anything
- Claude tells you to re-run `/start-ticket-analysis`

Restore the field before continuing.

---

#### Test 5 — Implementation with HITL and git diff

```
/approve-step
```

**What to verify:**
- Claude shows step details (title, description, files, test command, commit message) and asks "Proceed? (yes / no / show me the files first)"
- Claude does NOT implement anything until you explicitly say yes
- After implementation, Claude runs the test command and shows output
- Claude runs `git diff` and displays the actual diff — not just a description
- Claude asks "Does the implementation look correct?" before giving the commit command
- If you say "no" or provide corrections: Claude fixes the code and asks again before giving commit command
- If you say "looks good": Claude shows explicit `git add <files>` and `git commit -m "..."` commands for you to run — it does NOT commit automatically
- After you reply "looks good", state file shows `"current_step": 1` and `completed_steps` has `{"step": 1, "commit": "pending"}`

---

#### Test 6 — Rollback

After committing step 1 manually (run the git commands Claude provided), run:
```
/rollback-step
```

**What to verify:**
- Claude reads state and identifies the last completed step
- Claude shows: step title, commit to revert, and a confirmation prompt
- Claude does NOT revert automatically — waits for "yes"
- After confirmation: Claude provides `git revert <hash> --no-edit` for you to run
- After you confirm the revert is done: state file removes step 1 from `completed_steps`, sets `current_step` to 0

---

#### Test 7 — Completion and PR description

Implement all steps (run `/approve-step` repeatedly, committing each step manually). On the final step:

**What to verify:**
- After you say "looks good" on the last step: Claude generates a PR description with ticket title, summary, per-step bullets with commit messages, full files-changed list, and test commands
- `active_state.json` is updated to `{"state_path": null}`
- Running `/approve-step` again shows "All N steps are complete" — not an error

---

#### Test 8 — Session persistence

After any command mid-workflow, close Claude Code completely. Reopen it in the `dev-workflow/` directory. Run:
```
/approve-step
```

**What to verify:**
- Claude reads `active_state.json`, finds the state file, and resumes exactly where you left off
- No need to re-run `/start-ticket-analysis`
- `/status` shows the current workflow state correctly

---

#### Test 9 — Snapshot staleness protection

Run `/start-ticket-analysis` twice on the same `codebase_path` with different `ticket_source` values.

**What to verify:**
- Each run deletes and regenerates `codebase_context.md` — never reuses the previous ticket's snapshot
- The snapshot header shows the new ticket's `file_prefix`
- `active_state.json` points to the new ticket's state file

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

Workflow state is written to `<codebase_path>/.dev-workflow/`. Each analysis gets its own named files derived from the ticket ID and title:

```
your-project/.dev-workflow/
  PROJ-123_add-token-refresh.html          ← analysis report
  PROJ-123_add-token-refresh_state.json    ← workflow state
  active_state.json                        ← pointer to current active analysis
  codebase_context.md                      ← codebase snapshot (reused by review iterations to avoid re-reading files)
```

Key fields in the state file:

| Field | Description |
|-------|-------------|
| `phase` | `review` or `implementation` |
| `ticket_source` | The original ticket input |
| `file_prefix` | The slug used for this analysis's filenames |
| `codebase_path` | The analyzed repo path |
| `report_path` | Full path to the HTML/MD report |
| `implementation_plan` | Array of step objects (title, files, test_command, commit_message) |
| `current_step` | Last completed step number |
| `completed_steps` | Array of completed steps with commit status |
| `review_iterations` | Number of feedback cycles completed |

**To start a new analysis:** just run `/start-ticket-analysis` — it creates new files with the new ticket's name. Previous analyses remain in `.dev-workflow/` untouched.

**To reset everything:** delete the `.dev-workflow/` folder from your project root.

**Add to `.gitignore`:** the plugin offers to do this automatically on first run. If you skipped it, add manually:
```
.dev-workflow/
```

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
- **Don't commit `.dev-workflow/` to your repo.** State files, reports, and the codebase snapshot are session-specific and contain local paths. The plugin offers to add it to `.gitignore` automatically on first run.

---

## Roadmap

### Done — Implemented

| # | What | Status |
|---|------|--------|
| 1 | **`git diff` after implementation** | ✅ `approve_step.md` Step 6 now runs `git diff` and displays the full output before asking for review. |
| 2 | **Snapshot regeneration guarantee** | ✅ Snapshot write moved to Step 6 (after `mkdir`); old `codebase_context.md` deleted before each new run. |
| 3 | **PR description on completion** | ✅ `approve_step.md` Step 9 generates a copy-ready PR description when all steps complete. |
| 4 | **State validity check** | ✅ All three commands (`approve_step`, `submit_review_feedback`, `rollback_step`) validate required fields after reading state. |
| 5 | **Analysis scope parameter** | ✅ Optional `scope` input added to `/start-ticket-analysis`; constrains exploration and snapshot to specified subdirectories. |

### Future — Larger Features

These require more work but would meaningfully extend the plugin's value.

| # | What | Why |
|---|------|-----|
| 6 | **`/refresh-snapshot` command** | If you pull significant codebase changes mid-ticket (teammates merging while you're mid-implementation), the snapshot 
becomes stale. This command re-explores the codebase and rewrites `codebase_context.md` without touching the state file, report, or implementation plan. Run it after any significant pull during an active workflow. |
| 7 | **`/edit-plan` command** | Common scenario: the report is good but you want to reorder or split steps before implementing. Currently forces a full `/submit-review-feedback` cycle. A direct plan editor removes that friction. |
| 8 | **JIRA write-back** | After implementation, post a comment to the JIRA ticket with commit hashes and report path. Closes the loop on the ticket lifecycle without leaving the workflow. |
| 9 | **Step `depends_on` field** | The plan is strictly sequential but some steps are genuinely parallel (update tests + update docs). An optional `depends_on` field on each step would surface which steps the developer could run in parallel. |

### Known Limitations

- **Prompt-only execution** — there is no code enforcing that Claude followed every step. Report quality depends on the model's context at the time. Always read the report critically.
- **Large codebases** — for repos over ~500k LOC, the snapshot will inevitably be incomplete. Treat the analysis as directional, not exhaustive.
- **Trivial tickets** — the analysis overhead is not worth it for one-liner changes, config updates, or renames. Use the plugin for medium-to-large features and non-trivial bug fixes.

---

## License

MIT

---

## Author

Ahmed Samy
