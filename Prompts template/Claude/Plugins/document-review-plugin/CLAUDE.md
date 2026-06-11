# Document Review Plugin

A Claude Code plugin for orchestrated document review, technical accuracy checking, and compliance validation.

## Agents

| Agent                  | Invocation                      | Purpose                                                                                       |
| ---------------------- | ------------------------------- | --------------------------------------------------------------------------------------------- |
| `main-agent`           | `@main-agent` or via Agent tool | Orchestrates the full review workflow — delegates to subagents and aggregates results         |
| `tech-review-subagent` | Invoked by `main-agent`         | Validates technical accuracy; pauses for human clarification (HITL) when content is ambiguous |

## Commands (Slash Commands)

| Command              | Purpose                                                                       |
| -------------------- | ----------------------------------------------------------------------------- |
| `/check-compliance`  | Check a document section against a compliance standard (e.g., ISO 9001, GDPR) |
| `/summarize-section` | Produce a structured summary of a document section                            |

## Typical Usage

1. Paste a document or provide a file path in your Claude Code conversation.
2. For a full review: ask `@main-agent` to review the document.
3. For targeted checks: use `/check-compliance` or `/summarize-section` directly.
4. If the technical review finds ambiguous content, the agent will pause and ask you for clarification before continuing.

## Hook Behavior

After each review Task completes, a timestamped log line is echoed to the terminal via the `PostToolUse` hook defined in `settings.json`.

## Testing the Plugin

### Prerequisites
Open Claude Code with `document-review-plugin/` as the working directory — `.claude/` is only discovered from the project root:
```bash
cd "path/to/document-review-plugin"
claude
```

### 1. Test slash commands
```
/check-compliance ISO 9001 — Section 4.1: The organization determines external and internal issues relevant to its purpose. Issues are reviewed annually.
```
**Expected:** Returns Compliance Status, Issues Found, and Recommendations mapped to ISO 9001 clauses.

```
/summarize-section Section 1: This document describes vendor onboarding. Vendors must submit Form A within 30 days. Finance approves within 5 business days. Open question: who handles late submissions?
```
**Expected:** Returns Topic, Key Points, Decisions/Action Items, and Open Questions.

### 2. Test the main agent (full workflow)
```
@main-agent Review this document: "API Timeout Policy v1.2 — All REST endpoints must respond within 200ms. Compliant with RFC 7231. Data retention: 6 months."
```
**Expected:** Agent asks for the compliance standard if not provided, delegates to `tech-review-subagent`, then returns a full report with Technical Findings, Compliance Findings, and Recommendations.

### 3. Test HITL (human-in-the-loop pause)
```
@main-agent Review this: "System shall process requests fast enough for user satisfaction."
```
**Expected:** `tech-review-subagent` flags "fast enough" as ambiguous and pauses to ask for the actual SLA target before continuing.

### 4. Verify the hook fires
After any review that invokes the Task tool, check the terminal for:
```
[document-review] Review task completed at <timestamp>
```
If absent, confirm `settings.json` includes `"Bash(echo:*)"` in the `permissions.allow` list.

### Quick smoke test checklist
- [ ] `CLAUDE.md` content appears in the first system message
- [ ] `/check-compliance` returns a structured compliance result
- [ ] `/summarize-section` returns Topic + Key Points + Action Items
- [ ] `@main-agent` produces a full review report
- [ ] HITL pause triggers on ambiguous content
- [ ] Hook log line appears in terminal after Task completes

## Plugin Structure

```
document-review-plugin/
├── CLAUDE.md                        ← this file (loaded automatically)
└── .claude/
    ├── settings.json                ← permissions + hook definitions
    ├── README.md                    ← internal config notes
    ├── agents/
    │   ├── main_agent.md            ← orchestrator agent
    │   └── tech_review_subagent.md  ← technical reviewer with HITL
    └── commands/
        ├── check_compliance.md      ← /check-compliance slash command
        └── summarize_section.md     ← /summarize-section slash command
```
