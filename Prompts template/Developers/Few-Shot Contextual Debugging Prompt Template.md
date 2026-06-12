# Few-Shot Contextual Debugging Prompt Template

## Template Category

**Category:**  
- Few-Shot Prompting  
- Contextual Debugging & Troubleshooting  
- Performance Engineering  
- DevOps & SRE

---

## Solution Summary

This template uses a **Few-Shot Prompting** approach for technical troubleshooting and debugging. By providing a clear role, rich contextual details (system configuration, logs, environment), explicit instructions and constraints, and an example response, the template ensures the LLM produces structured, actionable, and best-practice-aligned recommendations. This approach is ideal for engineering, DevOps, SRE, or any scenario where context and output format are critical.

---

## Prompt

You are a [role, e.g., senior performance engineer]. Analyze the following scenario and provide actionable, [domain-specific constraints, e.g., thread-safe, scalable] recommendations for mitigating issues. Do not suggest architectural overhauls.

**Context:**
- **System/Service Details:**  
  [e.g., CPU cores, RAM, container limits, deployment environment]
- **Configuration:**  
  [e.g., JVM arguments, thread pool settings, environment variables]
- **Error Logs (excerpt):**  
    ```txt
    [Paste relevant log lines here]
    ```
- **Observed Behavior:**  
[e.g., Latency spikes, failures, resource exhaustion, etc.]

**Instructions:**
- Identify likely causes of the observed issues based on the provided context and logs.
- Suggest practical, [domain-specific constraints, e.g., thread-safe, scalable] mitigation steps that do not require architectural changes.
- Recommendations must align with [domain] best practices.
- Address edge cases and ensure suggestions are feasible for production environments.

**Constraints:**
- Do not propose architectural overhauls (e.g., switching frameworks, breaking up the service).
- All recommendations must be [domain-specific constraints, e.g., thread-safe, scalable].
- Solutions should be actionable and suitable for immediate engineering consideration.

**Expected Output:**
- Concise analysis of the logs, configuration, and system context.
- Specific, actionable mitigation recommendations.
- Suggestions that respect the "no architectural change" constraint and best practices.

---

## Example LLM Response

**Analysis:**  
[LLM provides a concise analysis of the logs, configuration, and system context, identifying likely root causes.]

**Mitigation Recommendations:**

1. **[Recommendation 1]:**  
 - [Details and rationale]

2. **[Recommendation 2]:**  
 - [Details and rationale]

3. **[Recommendation 3]:**  
 - [Details and rationale]

**Note:**  
All recommendations are [domain-specific constraints, e.g., thread-safe, scalable], do not require architectural changes, and align with best practices for production environments.

---

**Template Name:** Few-Shot Contextual Debugging Prompt Template

---

**Is it worth it?**  
Yes! This template:
- Ensures consistent, high-quality, and actionable debugging guidance.
- Saves time and effort for engineers and technical teams.
- Can be easily adapted for any system, language, or domain.
- Supports best practices in structured, context-aware troubleshooting.
