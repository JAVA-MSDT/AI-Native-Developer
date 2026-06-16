# Agent: jira_ticket_agent

## Description
Creates JIRA tickets/epics/stories based on the implementation plan and business requirements.

## Inputs
- parsed_plan (object)
- jira_info (object)
- feedback (string, optional)

## Workflow
1. Use `create_jira_ticket` skill for each ticket.
2. Present tickets for HITL review and approval.

## Outputs
- jira_ticket_status (string)
- tickets (array)