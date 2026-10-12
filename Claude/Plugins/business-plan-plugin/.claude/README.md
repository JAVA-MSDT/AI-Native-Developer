# Business Plan Plugin

## About
 
## Features

- HITL at every stage: User must approve before proceeding.
- Modular agents and skills: Easy to extend for new document types or ticketing systems.
- Supports local and remote documentation creation.
- Clear, actionable reports and documentation for BA/PO and teams.

## Tech Stack
 
## Project Structure
 
```txt
business-plan-plugin/
│
├── CLAUDE.md
└── .claude/
    ├── settings.json
    ├── README.md
    ├── agents/
    │   ├── orchestrator_agent.md
    │   ├── business_plan_agent.md
    │   ├── report_generator_agent.md
    │   ├── documentation_agent.md
    │   └── jira_ticket_agent.md
    ├── commands/
    │   ├── start_business_plan_analysis.md
    │   ├── submit_report_feedback.md
    │   ├── approve_report.md
    │   ├── create_documentation.md
    │   ├── approve_document.md
    │   └── create_jira_tickets.md
    └── skills/
        ├── parse_business_plan.md
        ├── generate_report.md
        ├── generate_documentation.md
        ├── create_jira_ticket.md
```

### Workflow Diagram (Textual)

```txt
User provides business plan (file path or text)
        |
        v
/start_business_plan_analysis
        |
        v
orchestrator_agent
        |
        v
business_plan_agent --> parse_business_plan skill
        |
        v
report_generator_agent --> generate_report skill
        |
        v
HITL: User reviews report
        |      \
        |       \
approve_report   submit_report_feedback (for changes)
        |       /
        |      /
        v
If approved, create documentation (documentation_agent)
        |
        v
HITL: User reviews each document (approve_document or feedback)
        |
        v
If approved, create JIRA tickets (jira_ticket_agent)
        |
        v
HITL: User reviews tickets
        |
        v
Workflow complete
```

## Getting Started

### How to Use

1. Start analysis:
/start_business_plan_analysis business_plan="path/to/plan.txt" output_format="md" documentation_path="docs/"

2. Review the report:
If you want changes, run:
/submit_report_feedback feedback="Clarify business goals" report="current_report.md"

3. Approve the report:
/approve_report approval=true report="final_report.md"

4. Create documentation:
/create_documentation documentation_path="docs/" output_format="md"

5. Approve each document:
/approve_document approval=true document="architecture.md"

6. Create JIRA tickets:
/create_jira_tickets jira_info="{...}" parsed_plan="{...}"
 
## Configuration

 
## License
 
## Author