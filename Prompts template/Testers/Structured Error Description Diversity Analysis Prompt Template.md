# Structured Error Description Diversity Analysis Prompt Template (Penalty Parameter Sweep)

## Template Category

**Category:**  
- LLM Output Evaluation & Benchmarking  
- Zero-Shot Prompting  
- QA Automation & Error Analysis  
- Diversity & Clarity Metrics

---

## Solution Summary

This template uses a **Zero-shot Prompting** approach with explicit, structured instructions and formatting constraints. It is designed for evaluating how different frequency and presence penalty settings affect the diversity, clarity, and usefulness of LLM-generated error descriptions. The template ensures reproducibility by requiring per-run sections, concrete diversity metrics, and a comparative assessment, making it ideal for QA automation, prompt engineering, and LLM evaluation tasks.

---

## Prompt

You are a QA automation specialist. Given the error log below, generate concise, clear, and logically diverse error descriptions for each failed test case. For each run, use the specified frequency and presence penalty settings at the API level:

- **Run 1:** Frequency Penalty = [low value, e.g., 0], Presence Penalty = [low value, e.g., 0]
- **Run 2:** Frequency Penalty = [medium value, e.g., 0.5], Presence Penalty = [medium value, e.g., 0.7]
- **Run 3:** Frequency Penalty = [high value, e.g., 1.0], Presence Penalty = [high value, e.g., 1.2]

**Instructions:**
- For each run, output a section with a heading (e.g., "Run 1: ..."), followed by a bulleted list of error descriptions (one per test case).
- Each description must be 15–25 words.
- Avoid repetitive phrasing, template reuse, and identical sentence structures; vary syntax and vocabulary for logical diversity.
- If information is missing, mark as N/A.
- After each run, report:
  - Repetition count (number of identical or near-identical phrases)
  - Average description length (in words)
  - A brief qualitative diversity note (e.g., "high diversity in structure and vocabulary")
- After all runs, provide a comparative assessment:
  - Summarize the strengths and weaknesses of each penalty setting.
  - Explicitly identify which penalty combination produced the most useful, non-repetitive, and actionable error descriptions for QA purposes.

**Error Log:**
 - TestCase: [testCaseName] Result: FAIL Error: [Error message] ...

---

## Comparative Assessment Table

| Run | Frequency Penalty | Presence Penalty | Repetition Count | Avg. Length | Diversity | Clarity |
|-----|-------------------|------------------|------------------|-------------|----------|---------|
| 1   | [value]           | [value]          | [count]          | [words]     | [note]   | [note]  |
| 2   | [value]           | [value]          | [count]          | [words]     | [note]   | [note]  |
| 3   | [value]           | [value]          | [count]          | [words]     | [note]   | [note]  |

- **Run 1:** [Summary]
- **Run 2:** [Summary]
- **Run 3:** [Summary]

**Optimal Setting:**  
[Explicitly state which penalty combination is optimal and why.]

---

**Template Name:** Structured Error Description Diversity Analysis Prompt Template (Penalty Parameter Sweep)

---

**Is it worth it?**  
Yes! This template:
- Enables systematic, reproducible evaluation of LLM output diversity and clarity under different penalty settings.
- Supports best practices in QA automation, prompt engineering, and LLM benchmarking.
- Is easily adaptable for any error log, test scenario, or domain.
- Facilitates actionable insights for optimizing LLM-driven reporting and analysis.