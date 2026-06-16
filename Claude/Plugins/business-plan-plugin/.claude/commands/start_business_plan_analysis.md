# Command: /start_business_plan_analysis

## Description
Starts the business plan analysis workflow.

## Inputs
- business_plan (string or filepath)
- output_format (string)
- documentation_path (string, optional)
- remote_info (object, optional)

## Workflow
1. Calls `orchestrator_agent` to begin the workflow.

## Outputs
- report (string)