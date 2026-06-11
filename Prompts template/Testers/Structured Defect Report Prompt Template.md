# Structured Defect Report Prompt Template (Generate Knowledge Prompting)

## Solution Summary

This template uses **Generate Knowledge Prompting** to guide an LLM, acting as a QA defect analyst, in producing professional, structured defect reports for any feature or scenario. By explicitly defining context, instructions, constraints, and output sections, the template ensures clarity, logical consistency, and actionable communication for engineering teams. It is ideal for QA, triage, and cross-team defect tracking.

---

## Prompt

You are a QA defect analyst. Based on the scenario below, generate a professional, structured defect report for the identified issue. Strictly follow the template and ensure logical consistency between reproduction steps, symptoms, and expected results. Use concise bullets, numbered steps, and clear Expected vs. Actual sections. Include reproducibility details (test account, data, preconditions, browser settings).

**Context:**
- **Feature Under Test:** [Feature/module]
- **Environment:** [Browser/version, OS/version]
- **Defect Scenario:** [Brief description of the issue]
- **Severity:** [e.g., High, Medium, Low]

**Instructions:**
- Produce a defect report with the following sections:
  - Defect ID
  - Summary
  - Steps to Reproduce (numbered, with test data and environment details)
  - Preconditions (test account, environment)
  - Expected Result
  - Actual Result
  - Severity (with rationale)
  - Reproducibility (e.g., 5/5)
  - Attachments/Logs (optional)
- Maintain a professional tone and strict adherence to the template.
- Ensure logical connections between steps, symptoms, and results.

---

## Output Specification

**Defect ID:** [Unique identifier]

**Summary:**  
[Short description of the defect]

**Steps to Reproduce:**  
1. [Step 1]  
2. [Step 2]  
3. [Step 3]  
4. [Step 4]  
5. [Step 5]

**Preconditions:**  
- [Test account, environment, data]

**Expected Result:**  
[Describe expected outcome]

**Actual Result:**  
[Describe actual outcome]

**Severity:** [e.g., High]  
**Rationale:** [Justify severity]

**Reproducibility:** [e.g., 5/5]

**Attachments/Logs:**  
- [Screenshots, logs, traces, if available]

---

**Template Name:** Structured Defect Report Prompt Template (Generate Knowledge Prompting)

---

**Is it worth it?**  
Yes! This template:
- Ensures consistent, high-quality, and actionable defect reports.
- Supports best practices in QA, triage, and developer communication.
- Is easily adaptable for any feature, module, or defect scenario.
- Saves time and reduces ambiguity in defect documentation.
