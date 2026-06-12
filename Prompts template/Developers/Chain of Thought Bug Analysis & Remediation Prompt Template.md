# Chain of Thought Bug Analysis & Remediation Prompt Template

## Template Category

**Category:**  
- Chain of Thought Prompting  
- Bug Analysis & Root Cause Investigation  
- Technical Debugging & Remediation  
- Performance & System Analysis

---

## Solution Summary

This template uses the **Chain of Thought Prompting** technique to guide an LLM through structured, step-by-step analysis of complex bugs or performance issues. By explicitly breaking down the investigation into logical stages (e.g., log interpretation, artifact analysis, correlation, and remediation), the template ensures thorough, evidence-based, and actionable outputs. This approach is ideal for debugging, root cause analysis, and non-invasive remediation in any technical domain.

---

## Prompt

You are an expert [domain/technology] engineer. Analyze the following scenario using a step-by-step, chain-of-thought approach to identify the root cause of the issue and provide actionable, non-invasive remediation steps. The environment is [system/version/constraints].

**Context:**  
- [Describe symptoms, e.g., resource usage patterns, error messages, performance degradation, etc.]
- Diagnostic artifacts available:  
    - [List relevant logs, dumps, traces, or metrics]

**Instructions:**  
1. **Interpret the logs/metrics:**  
   - Identify patterns, anomalies, or trends relevant to the issue.
2. **Analyze diagnostic artifacts:**  
   - Look for objects, resources, or processes with abnormal behavior or retention.
3. **Correlate findings:**  
   - Connect evidence across artifacts (e.g., threads holding references, resource contention).
4. **Identify likely suspects:**  
   - List components, classes, or patterns most likely responsible for the issue.
   - Explain how the issue manifests in the artifacts.
5. **Recommend tools and remediation steps:**  
   - Suggest profiling, monitoring, or analysis tools.
   - Propose non-invasive fixes or code review focus areas.

**Constraints:**  
- Recommendations must be non-invasive (no architecture redesign).
- Solutions must be applicable in [system/version/constraints].
- Output should be structured as an analysis report.

---

## Example LLM Response

### Analysis Report

**1. Log/Metric Interpretation:**  
- [Summary of key findings from logs/metrics.]

**2. Artifact Analysis:**  
- [Summary of findings from dumps, traces, or other artifacts.]

**3. Correlation:**  
- [How evidence from different sources connects.]

**4. Likely Suspects:**  
- [List of likely root causes and how they manifest.]

**5. Recommendations:**  
- [Tools, fixes, and code review suggestions.]

**Note:**  
All recommendations are non-invasive and suitable for [system/version/constraints].

---

**Template Name:** Chain of Thought Bug Analysis & Remediation Prompt Template

---

**Is it worth it?**  
Yes! This template:
- Ensures systematic, logical, and thorough bug analysis.
- Produces actionable, evidence-based remediation steps.
- Is easily adaptable for any technology, system, or debugging scenario.
- Supports best practices in prompt engineering and technical troubleshooting.
