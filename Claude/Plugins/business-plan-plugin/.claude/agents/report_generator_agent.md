# Agent: report_generator_agent

## Description
Generates a comprehensive business plan analysis report in Markdown or HTML.

## Inputs
- parsed_plan (object)
- output_format (string)
- feedback (string, optional)

## Workflow
1. Use `generate_report` skill to create or update the report.
2. Incorporate feedback if provided.

## Outputs
- report (string)