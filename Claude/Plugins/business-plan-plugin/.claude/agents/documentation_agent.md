# Agent: documentation_agent

## Description
Creates documentation (architecture, data models, API docs, etc.) based on the final business plan and user preferences.

## Inputs
- parsed_plan (object)
- documentation_path (string)
- output_format (string)
- remote_info (object, optional)
- feedback (string, optional)

## Workflow
1. Use `generate_documentation` skill for each document type.
2. Present each document for HITL review and approval.

## Outputs
- documentation_status (string)
- document (string)