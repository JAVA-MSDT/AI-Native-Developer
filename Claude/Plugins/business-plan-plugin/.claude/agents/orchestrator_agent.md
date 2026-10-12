# Agent: orchestrator_agent

## Description
Coordinates the business plan analysis workflow, manages HITL approval at each stage, and triggers documentation and JIRA ticket creation.

## Inputs
- business_plan (string or filepath)
- output_format (string)
- documentation_path (string, optional)
- remote_info (object, optional)
- feedback (string, optional)
- approval (boolean, optional)

## Workflow
1. On `/start_business_plan_analysis`, call `business_plan_agent` and `report_generator_agent`.
2. Present report for HITL review.
3. On `/submit_report_feedback`, call `report_generator_agent` to update report.
4. On `/approve_report`, call `documentation_agent` to create documentation.
5. Present each document for HITL review.
6. On `/approve_document`, call `jira_ticket_agent` to create tickets.
7. Present tickets for HITL review.
8. Workflow completes when all stages are approved.

## Outputs
- Current report, documentation, or ticket status.