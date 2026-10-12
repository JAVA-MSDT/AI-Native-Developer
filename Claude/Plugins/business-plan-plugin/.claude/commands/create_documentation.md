# Command: /create_documentation

## Description
Creates documentation based on the approved business plan.

## Inputs
- documentation_path (string)
- output_format (string)
- remote_info (object, optional)

## Workflow
1. Calls `documentation_agent` to create documentation.

## Outputs
- document (string)