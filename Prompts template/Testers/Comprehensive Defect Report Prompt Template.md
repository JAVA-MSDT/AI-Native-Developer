# Comprehensive Defect Report Prompt Template (with Prompt Components & Few-Shot Example)

## Template Category

**Category:**  
- Defect Reporting & Bug Documentation  
- Few-Shot Prompting  
- QA & Bug Triage  
- Developer Communication

---

## Solution Summary

This template uses explicit **Prompt Components** (Instruction, Input Context, Constraints, Output Specification) and a **few-shot example** to guide the LLM in generating structured, developer-ready defect reports. The approach ensures technical accuracy, logical consistency, and professional communication, making it ideal for QA, bug triage, and cross-team collaboration. Few-shot prompting is used to reinforce schema and tone; chain-of-thought can be added for deeper analysis if needed.

---

## Prompt

### Instruction
Write a detailed defect report for an edge case or bug in [feature/module] of [application/system], clearly documenting the issue and supporting developer resolution.

### Input Context
- **Feature Under Test:** [Describe the feature/module]
- **Defect Scenario:** [Describe the specific defect or edge case]
- **Steps to Reproduce:**
    1. [Step 1]
    2. [Step 2]
    3. [Step 3]
    4. [Step 4]
    5. [Step 5]
    6. [Step 6]
    7. [Step 7]
- **Cart/State/Test Data:** [Describe relevant state or test data]
- **Expected Behavior:** [Describe what should happen]
- **Environment:** [Browser, OS, device, staging/production, etc.]
- **Affected Version/Build:** [Version/build info]
- **Timestamp:** [Date/time of defect]

### Constraints
- Ensure the report is technically and logically consistent.
- Follow the standard defect report template.
- Maintain a professional and clear tone for effective communication.
- Do not invent logs or data; if unavailable, mark as N/A.
- Use consistent formatting (e.g., currency, timestamps).
- Include reproducibility rate, affected build/version, and timestamp if known.
- If any field is unknown, mark as N/A.

### Output Specification
The defect report must include the following sections:
- Defect ID
- Summary
- Steps to Reproduce
- Expected Result
- Actual Result
- Severity (with rationale)
- Priority (optional)
- Reproducibility (e.g., 5/5)
- Environment
- Affected Version/Build
- Impact/Business Risk (optional)
- Suspected Root Cause/Notes (optional)
- Workarounds (optional)
- Attachments/Logs (optional)

---

## Example Output

**Defect ID:** [Unique identifier]

**Summary:**  
[Short description of the defect]

**Steps to Reproduce:**  
1. [Step 1]
2. [Step 2]
3. [Step 3]
4. [Step 4]
5. [Step 5]
6. [Step 6]
7. [Step 7]

**Expected Result:**  
[Describe expected outcome]

**Actual Result:**  
[Describe actual outcome]

**Severity:** [e.g., Medium]  
**Rationale:** [Justify severity]

**Priority:** [e.g., High]

**Reproducibility:** [e.g., 5/5]

**Environment:**  
- [Browser/OS/Device]
- [Staging/Production]

**Affected Version/Build:** [Version/build info]  
**Timestamp:** [Date/time]

**Impact/Business Risk:**  
[Describe business/user impact]

**Suspected Root Cause/Notes:**  
[Technical notes or hypotheses]

**Workarounds:**  
[Describe any temporary fixes]

**Attachments/Logs:**  
- [Screenshots, logs, traces, if available]

---

**Template Name:** Comprehensive Defect Report Prompt Template (with Prompt Components & Few-Shot Example)

---

**Is it worth it?**  
Yes! This template:
- Ensures consistent, high-quality, and actionable defect reports.
- Supports best practices in QA, bug triage, and developer communication.
- Is easily adaptable for any application, feature, or defect scenario.
- Saves time and reduces ambiguity in defect documentation.
