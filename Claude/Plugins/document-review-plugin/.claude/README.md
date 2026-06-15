# Document Review Plugin — .claude Configuration

This directory contains all Claude Code configuration for the document-review plugin.

## What happened to Skills/ and Hooks/?

| Old (non-standard) | New (Claude Code convention) | Why                                                              |
| ------------------ | ---------------------------- | ---------------------------------------------------------------- |
| `Skills/`          | `commands/`                  | Claude Code loads slash commands from `commands/`, not `Skills/` |
| `Hooks/`           | removed                      | Hooks are defined in `settings.json`, not as separate files      |

## Structure

```
.claude/
├── settings.json                    ← permissions + PostToolUse hook
├── README.md                        ← this file
├── agents/
│   ├── main_agent.md                ← orchestrator agent (@main-agent)
│   └── tech_review_subagent.md      ← technical reviewer with HITL support
└── commands/
    ├── check_compliance.md          ← /check-compliance slash command
    └── summarize_section.md         ← /summarize-section slash command
```

## Agents

| Name                   | Invocation              | Role                                                             |
| ---------------------- | ----------------------- | ---------------------------------------------------------------- |
| `main-agent`           | `@main-agent` in chat   | Coordinates the full review workflow                             |
| `tech-review-subagent` | Spawned by `main-agent` | Technical accuracy review; pauses for human input when ambiguous |

## Commands

| Command              | File                            | Usage                              |
| -------------------- | ------------------------------- | ---------------------------------- |
| `/check-compliance`  | `commands/check_compliance.md`  | Paste section text as the argument |
| `/summarize-section` | `commands/summarize_section.md` | Paste section text as the argument |

## Permissions

`settings.json` grants `Read(*)` (agents can read files) and `Bash(echo:*)` (hook logging). Add more entries to the `allow` array as needed.

## Hook

A `PostToolUse` hook fires after every `Task` tool call and logs a completion timestamp. Defined in `settings.json` under `hooks.PostToolUse`.

## Author

- **Ahmed Samy** — collected from real-world AI experience and shared for the community.
- **LinkedIn** [Catch Me There](https://www.linkedin.com/in/java-msdt/)