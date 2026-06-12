# Few-Shot Functional Test Case Generation Prompt Template (Payment Gateway Example)

## Template Category

**Category:**  
- Test Case Generation  
- Few-Shot Prompting  
- QA & Regression Testing  
- Test Automation & Coverage

---

## Solution Summary

This template uses **Few-Shot Prompting** to guide an LLM in generating detailed, execution-ready functional test cases for any feature or module. By providing comprehensive example test cases with strict formatting and domain context, the template ensures the LLM produces consistent, logically complete, and actionable test cases for unexplored scenarios. Few-shot prompting establishes clear guardrails for structure, depth, and coverage, making it ideal for QA, regression testing, and automation.

---

## Prompt

You are a QA engineer tasked with creating detailed, execution-ready functional test cases for the [feature/module] in [application/system]. The system supports [domain-specific details: e.g., payment methods, validation rules, currencies, retries, authentication, environment assumptions]. Review the examples below and generate [N] new, non-duplicative test cases for unexplored edge scenarios (e.g., invalid input, network failure, exceeding limits, unsupported types, failed authentication, etc.). Each test case must include:

- Test Case ID (incremental, e.g., TC003, TC004, ...)
- Title
- Preconditions (user state, environment, test data, etc.)
- Test Steps (step-by-step, with data values)
- Expected Output (UI messages, error codes, etc.)
- Post-Conditions (order status, logs, receipts, database effects)

**Strict Formatting:**  
- Use the same structure and level of detail as the examples.
- Do not duplicate the provided examples.
- Ensure test cases are clear, logically complete, and ready for execution by QA engineers.

---

## Example Test Cases

**Test Case ID:** TC001  
**Title:** [Successful scenario]  
**Preconditions:**  
- [User state, valid data, environment]  
**Test Steps:**  
1. [Step 1]  
2. [Step 2]  
3. [Step 3]  
**Expected Output:**  
- [Success message, confirmation, etc.]  
**Post-Conditions:**  
- [Status update, logs, receipt, etc.]

---

**Test Case ID:** TC002  
**Title:** [Failure scenario]  
**Preconditions:**  
- [User state, invalid/expired data, environment]  
**Test Steps:**  
1. [Step 1]  
2. [Step 2]  
3. [Step 3]  
**Expected Output:**  
- [Error message, error code, etc.]  
**Post-Conditions:**  
- [Status update, logs, no receipt, etc.]

---

## Generate [N] New Edge Test Cases

[LLM to generate new test cases following the above structure and domain context.]

---

**Template Name:** Few-Shot Functional Test Case Generation Prompt Template

---

**Is it worth it?**  
Yes! This template:
- Ensures consistent, high-quality, and actionable test case generation.
- Supports best practices in QA, regression, and automation.
- Is easily adaptable for any feature, module, or domain.
- Saves time and reduces ambiguity in test design.
