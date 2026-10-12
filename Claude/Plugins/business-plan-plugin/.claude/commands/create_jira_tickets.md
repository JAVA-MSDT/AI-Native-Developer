# Command: /create_jira_tickets

## Description
Creates JIRA tickets based on the implementation plan.

## Inputs
- jira_info (object)
- parsed_plan (object)

## Workflow
1. Calls `jira_ticket_agent` to create tickets.

## Outputs
- tickets (array)