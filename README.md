# AI Native Developer (Prompts, Experience Library and beyond)



## About

A curated collection of prompt templates and AI techniques gathered from real-world usage — shared openly so others can benefit, adapt, and build on them.

Over time, working with AI tools surfaces patterns: prompting styles that consistently produce better results, templates that save hours of back-and-forth, and techniques worth keeping. This repository captures those learnings in a reusable, well-named format so they don't stay locked in a single person's workflow.

Whether you're new to prompt engineering or a seasoned practitioner, you'll find ready-to-use templates and the reasoning behind each one.



## Features

- Ready-to-use prompt templates with clearly marked placeholders
- Each template documents the prompting technique used and why it works
- Organized by audience — general users, developers, testers, social media creators, and business analysts
- Covers a range of techniques: chain of thought, few-shot, zero-shot, role prompting, recipe pattern, R.O.S.E.S. framework, and more
- Designed to be adapted, not just copied
- Includes Claude Code plugins demonstrating agents, slash commands, and hooks



## Tech Stack

No code, no dependencies — just Markdown.

Compatible with any AI assistant (ChatGPT, Claude, Gemini, Copilot, etc.).



## Project Structure

```
AI-Native-Developer/
├── CLAUDE.md                                  # Project instructions for Claude Code
├── .gitignore
├── Prompts template/                          # Prompt templates by audience
│   ├── General/                               # Broadly applicable prompts
│   │   ├── Explicit Errors Identification and Correction Prompt Template.md
│   │   ├── Personalized Vacation Itinerary Prompt Template.md
│   │   ├── Presentation Generation Prompt Template.md
│   │   ├── Role-Based Career Advisor Prompt Template.md
│   │   ├── Role-Based Perspective Prompt Template.md
│   │   └── Zero-Shot Interview Feedback Generation Template.md
│   ├── Developers/                            # Engineering-focused prompts
│   │   ├── Chain of Thought Bug Analysis & Remediation Prompt Template.md
│   │   ├── Comprehensive Prompt Debugging & Enhancement Template.md
│   │   ├── Comprehensive Requirements Analysis & Incremental Implementation Template.md
│   │   ├── Few-Shot Contextual Debugging Prompt Template.md
│   │   ├── HTML Requirements Refactoring & Implementation Plan Template.md
│   │   ├── Incremental HTML Implementation Plan Template.md
│   │   ├── R.O.S.E.S. System Design & Solution Architecture Prompt Template.md
│   │   ├── README Generation Prompt Template.md
│   │   ├── Recipe Pattern Algorithm Design & Trade-off Analysis Prompt Template.md
│   │   └── Recommendation System Prompt Template.md
│   ├── Testers/                               # QA-focused prompts
│   │   ├── Comprehensive Defect Report Prompt Template.md
│   │   ├── Few-Shot Functional Test Case Generation Prompt Template.md
│   │   ├── Structured Defect Report Prompt Template.md
│   │   ├── Structured Error Description Diversity Analysis Prompt Template.md
│   │   └── java/
│   │       └── Comprehensive Java Test Class Structure & Coverage Template.md
│   ├── Social media/                          # Content creation and channel growth
│   │   └── YouTube Video Optimization Prompt Template.md
│   └── Business/                              # Business analysis prompts
│       └── Business Plan Analysis & Feature Implementation Prompt Template.md
└── Claude/                                    # Claude Code plugins and configurations
    └── Plugins/
        ├── document-review-plugin/            # Orchestrated document review plugin
        │   ├── CLAUDE.md                      # Plugin entry point loaded by Claude Code
        │   └── .claude/
        │       ├── settings.json              # Permissions and PostToolUse hook
        │       ├── agents/
        │       │   ├── main_agent.md          # Orchestrator agent (@main-agent)
        │       │   └── tech_review_subagent.md  # Technical reviewer with HITL support
        │       └── commands/
        │           ├── check_compliance.md    # /check-compliance slash command
        │           └── summarize_section.md   # /summarize-section slash command
        └── dev-workflow/                      # JIRA-driven development workflow plugin
            ├── CLAUDE.md                      # Plugin entry point loaded by Claude Code
            └── .claude/
                ├── settings.json              # Permissions and PostToolUse hook
                ├── agents/
                │   ├── orchestrator_agent.md        # Routes work, maintains state
                │   ├── ticket_analysis_agent.md     # Fetches and parses JIRA tickets
                │   ├── report_generator_agent.md    # Builds HTML/Markdown reports
                │   ├── reanalysis_agent.md          # Processes reviewer feedback
                │   └── implementation_agent.md      # Executes one step with git commit
                ├── commands/
                │   ├── start_ticket_analysis.md     # /start-ticket-analysis
                │   ├── submit_review_feedback.md    # /submit-review-feedback
                │   ├── approve_step.md              # /approve-step (HITL gate)
                │   ├── rollback_step.md             # /rollback-step
                │   ├── refresh_snapshot.md          # /refresh-snapshot
                │   └── status.md                    # /status
                └── skills/
                    ├── fetch_jira_ticket.md         # JIRA fetch patterns
                    ├── analyze_codebase.md          # Codebase exploration patterns
                    ├── generate_html_report.md      # HTML report template spec
                    ├── generate_md_report.md        # Markdown report spec
                    └── implement_code_change.md     # Safe edit + commit patterns
```



## Getting Started

1. Browse the template tables below and pick one that fits your situation.
2. Open the template file and read the **Solution Summary** to understand the technique.
3. Copy the prompt, fill in the `[bracketed placeholders]` with your specifics.
4. Paste into your AI assistant of choice and iterate.

### Templates

#### General
| Template                                                                                                                                                 | Technique         |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- |
| [Explicit Errors Identification and Correction](Prompts%20template/General/Explicit%20Errors%20Identification%20and%20Correction%20Prompt%20Template.md) | Explicit errors   |
| [Personalized Vacation Itinerary](Prompts%20template/General/Personalized%20Vacation%20Itinerary%20Prompt%20Template.md)                                 | Few-shot          |
| [Presentation Generation](Prompts%20template/General/Presentation%20Generation%20Prompt%20Template.md)                                                   | Structured output |
| [Role-Based Career Advisor](Prompts%20template/General/Role-Based%20Career%20Advisor%20Prompt%20Template.md)                                             | Role prompting    |
| [Role-Based Perspective](Prompts%20template/General/Role-Based%20Perspective%20Prompt%20Template.md)                                                     | Role prompting    |
| [Zero-Shot Interview Feedback Generation](Prompts%20template/General/Zero-Shot%20Interview%20Feedback%20Generation%20Template.md)                        | Zero-shot         |

#### Developers
| Template                                                                                                                                                                                     | Technique            |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------- |
| [Chain of Thought Bug Analysis & Remediation](Prompts%20template/Developers/Chain%20of%20Thought%20Bug%20Analysis%20%26%20Remediation%20Prompt%20Template.md)                                | Chain of thought     |
| [Comprehensive Prompt Debugging & Enhancement](Prompts%20template/Developers/Comprehensive%20Prompt%20Debugging%20%26%20Enhancement%20Template.md)                                           | Prompt refinement    |
| [Comprehensive Requirements Analysis & Incremental Implementation](Prompts%20template/Developers/Comprehensive%20Requirements%20Analysis%20%26%20Incremental%20Implementation%20Template.md) | Structured analysis  |
| [Few-Shot Contextual Debugging](Prompts%20template/Developers/Few-Shot%20Contextual%20Debugging%20Prompt%20Template.md)                                                                      | Few-shot             |
| [HTML Requirements Refactoring & Implementation Plan](Prompts%20template/Developers/HTML%20Requirements%20Refactoring%20%26%20Implementation%20Plan%20Template.md)                           | Structured output    |
| [Incremental HTML Implementation Plan](Prompts%20template/Developers/Incremental%20HTML%20Implementation%20Plan%20Template.md)                                                               | Incremental delivery |
| [R.O.S.E.S. System Design & Solution Architecture](Prompts%20template/Developers/R.O.S.E.S.%20System%20Design%20%26%20Solution%20Architecture%20Prompt%20Template.md)                        | R.O.S.E.S. framework |
| [README Generation](Prompts%20template/Developers/README%20Generation%20Prompt%20Template.md)                                                                                                | Zero-shot            |
| [Recipe Pattern Algorithm Design & Trade-off Analysis](Prompts%20template/Developers/Recipe%20Pattern%20Algorithm%20Design%20%26%20Trade-off%20Analysis%20Prompt%20Template.md)              | Recipe pattern       |
| [Recommendation System](Prompts%20template/Developers/Recommendation%20System%20Prompt%20Template.md)                                                                                        | System prompting     |

#### Testers
| Template                                                                                                                                                             | Technique                    |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| [Comprehensive Defect Report](Prompts%20template/Testers/Comprehensive%20Defect%20Report%20Prompt%20Template.md)                                                     | Few-shot                     |
| [Few-Shot Functional Test Case Generation](Prompts%20template/Testers/Few-Shot%20Functional%20Test%20Case%20Generation%20Prompt%20Template.md)                       | Few-shot                     |
| [Structured Defect Report](Prompts%20template/Testers/Structured%20Defect%20Report%20Prompt%20Template.md)                                                           | Generate knowledge           |
| [Structured Error Description Diversity Analysis](Prompts%20template/Testers/Structured%20Error%20Description%20Diversity%20Analysis%20Prompt%20Template.md)         | Zero-shot / Parameter tuning |
| [Comprehensive Java Test Class Structure & Coverage](Prompts%20template/Testers/java/Comprehensive%20Java%20Test%20Class%20Structure%20%26%20Coverage%20Template.md) | Structured output            |

#### Social Media
| Template                                                                                                              | Technique         |
| --------------------------------------------------------------------------------------------------------------------- | ----------------- |
| [YouTube Video Optimization](Prompts%20template/Social%20media/YouTube%20Video%20Optimization%20Prompt%20Template.md) | Structured output |

#### Business
| Template                                                                                                                                                          | Technique           |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------- |
| [Business Plan Analysis & Feature Implementation](Prompts%20template/Business/Business%20Plan%20Analysis%20%26%20Feature%20Implementation%20Prompt%20Template.md) | Structured analysis |



## Claude Plugins

The `Claude/Plugins/` directory contains Claude Code plugins — real-world examples of agents, slash commands, and hooks.

### document-review-plugin

An orchestrated document review workflow demonstrating multi-agent coordination and human-in-the-loop (HITL) patterns.

| Component              | Type          | Description                                                  |
| ---------------------- | ------------- | ------------------------------------------------------------ |
| `@main-agent`          | Agent         | Orchestrates the full review; delegates to the subagent      |
| `tech-review-subagent` | Subagent      | Validates technical accuracy; pauses on ambiguous content    |
| `/check-compliance`    | Slash command | Check a section against a compliance standard (ISO, GDPR, …) |
| `/summarize-section`   | Slash command | Produce a structured summary of a document section           |
| `PostToolUse` hook     | Hook          | Logs a timestamped completion line after each review task    |

To use the plugin, open Claude Code with `Claude/Plugins/document-review-plugin/` as your working directory. See [`CLAUDE.md`](Claude/Plugins/document-review-plugin/CLAUDE.md) for full usage and testing instructions.

### dev-workflow

A JIRA-driven development workflow plugin demonstrating a full 3-phase pipeline — Analysis → Review Loop → Implementation — with multi-agent orchestration, HITL approval gates, state persistence, and rollback support.

**Agents:**

| Component                | Type  | Description                                                               |
| ------------------------ | ----- | ------------------------------------------------------------------------- |
| `orchestrator_agent`     | Agent | Routes work across phases; maintains state in `<codebase>/.dev-workflow/` |
| `ticket_analysis_agent`  | Agent | Fetches and parses JIRA tickets, URLs, or pasted requirements             |
| `report_generator_agent` | Agent | Explores codebase (or reuses snapshot) and builds implementation reports  |
| `reanalysis_agent`       | Agent | Processes reviewer feedback against the snapshot; updates report in-place |
| `implementation_agent`   | Agent | Executes exactly one step per invocation; commits with explicit `git add` |

**Slash commands:**

| Component                 | Type          | Description                                                                      |
| ------------------------- | ------------- | -------------------------------------------------------------------------------- |
| `/start-ticket-analysis`  | Slash command | Kicks off the full workflow: fetch ticket → explore codebase → generate report   |
| `/submit-review-feedback` | Slash command | Feeds reviewer findings back into the analysis; updates report in-place          |
| `/approve-step`           | Slash command | HITL gate: shows next step, implements it, runs tests, then asks for sign-off    |
| `/rollback-step`          | Slash command | Identifies last committed step, provides the `git revert` command, updates state |
| `/refresh-snapshot`       | Slash command | Re-explores codebase without touching state, report, or plan                     |
| `/status`                 | Slash command | Read-only display of current phase, progress, and remaining steps                |
| `PostToolUse` hook        | Hook          | Logs a timestamped completion line after each subagent task                      |

To use the plugin, open Claude Code with `Claude/Plugins/dev-workflow/` as your working directory. Set the `JIRA_BASE_URL`, `JIRA_EMAIL`, and `JIRA_API_TOKEN` environment variables if using JIRA ticket fetching. See [`CLAUDE.md`](Claude/Plugins/dev-workflow/CLAUDE.md) for full usage and testing instructions.



## Configuration

No setup required for prompts — just choose the template that meets your needs.

## License

Open for use and adaptation. Attribution appreciated but not required.



## Author

- **Ahmed Samy** — collected from real-world AI experience and shared for the community.
- **LinkedIn** [Catch Me There](https://www.linkedin.com/in/java-msdt/)
