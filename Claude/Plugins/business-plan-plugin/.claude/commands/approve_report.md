# Command: /approve_report

## Description
Approves the business plan analysis report and triggers documentation creation.

## Inputs
- approval (boolean)
- report (string)

## Workflow
1. Calls `orchestrator_agent` to proceed to documentation creation.

## Outputs
- documentation_status (string)