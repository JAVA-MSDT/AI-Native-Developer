# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a pure-Markdown content repository with two distinct parts:

1. **`Prompts template/`** — A curated library of AI prompt engineering templates organized by audience (General, Developers, Testers, Social media).
2. **`Claude/Plugins/document-review-plugin/`** — A reference implementation of a Claude Code plugin demonstrating multi-agent orchestration, HITL patterns, slash commands, and hooks.

There are no build tools, package managers, or test runners — this is a content-only project.

## Prompt Template Structure

Every template file follows this internal structure:

- `## Solution Summary` — explains the prompting technique and why it works
- `## Prompt` — the actual template with `[bracketed placeholders]` for user substitution
- `## Example LLM Response` — illustrative output
- `## Is it worth it?` — rationale for when to use this approach

When adding a new template, place it in the appropriate audience folder and follow this structure exactly. Techniques used across the library include: Chain of Thought, Few-Shot, Zero-Shot, Role Prompting, Recipe Pattern, and R.O.S.E.S. structured roles.

## Document Review Plugin Architecture

The plugin lives at `Claude/Plugins/document-review-plugin/`. Claude Code must be opened with that directory as the working directory — the plugin's `.claude/` config is not discovered from the repo root.

**Agent structure:**
- `main_agent.md` — orchestrator; uses `Read`, `Grep`, `Task` tools; runs the full review workflow, delegates to the subagent, runs compliance checks, then aggregates a structured Markdown report
- `tech_review_subagent.md` — uses `Read`, `Grep`, `WebSearch`; validates technical accuracy; implements HITL by pausing to ask the user when content is ambiguous; returns structured JSON findings

**Slash commands** (in `.claude/commands/`):
- `/check_compliance` — maps document sections to standards (ISO 9001, GDPR, SOC 2, etc.)
- `/summarize_section` — extracts Topic, Key Points, Decisions/Action Items, Open Questions in ≤200 words

**Hook:** A `PostToolUse` hook in `.claude/settings.json` fires after every `Task` tool call and echoes a timestamped log line to the terminal.

**Permissions:** `Read(*)` and `Bash(echo:*)` — intentionally minimal scope.

## Repository Notes

- The `.claude/settings.local.json` at the repo root contains stale one-off permissions from a past file-rename refactor — safe to ignore.
- The `README.md` tables are slightly out of date; the `Social media/` category and several newer General templates (HTML refactoring, README Generation, Recommendation System) are not yet listed there.
- Past migration: plugin commands were previously in a `Skills/` directory and hooks in a `Hooks/` directory; both were consolidated per current Claude Code conventions (`commands/` folder, inline `settings.json` hook definitions).
