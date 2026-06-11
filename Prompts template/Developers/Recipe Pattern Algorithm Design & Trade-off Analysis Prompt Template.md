# Recipe Pattern Algorithm Design & Trade-off Analysis Prompt Template

## Solution Summary

This template uses the **Recipe Prompting Pattern** (with Zero-shot or Chain-of-Thought instructions as needed) to guide an LLM in generating multiple alternative solutions to a technical problem, analyzing their trade-offs, and recommending the best fit for real-world scenarios. It is ideal for algorithm design, optimization, or architectural decision-making where structured comparison and practical recommendations are required.

---

## Prompt

You are a [domain] expert specializing in [relevant systems or applications]. Using the Recipe pattern, generate at least three alternative solutions to the following problem:

**Problem Statement:**  
[Describe the technical challenge or optimization goal here.]

For each approach, provide:
- A brief description of the algorithm/solution (e.g., [insert relevant paradigms, frameworks, or techniques])
- An analysis of trade-offs (e.g., resource usage, complexity, maintainability, latency, scalability)
- Applicability in [target environment, language, or version]
- Recommendations for [key priorities, e.g., throughput, maintainability, reliability]

Present your response in a structured format, and conclude with a comparison table summarizing the strengths and weaknesses of each approach. Ensure your recommendations are practical for real-world [domain] systems.

---

## Example LLM Response

### 1. [Approach Name]

**Description:**  
[Brief explanation of the approach.]

**Trade-offs:**  
- **Resource Usage:** [Details]  
- **Complexity:** [Details]  
- **Maintainability:** [Details]  
- **Latency/Scalability:** [Details]

**Applicability:**  
[Where/when this approach is best used.]

**Recommendation:**  
[When/why to choose this approach.]

---

### 2. [Approach Name]

**Description:**  
[Brief explanation of the approach.]

**Trade-offs:**  
- **Resource Usage:** [Details]  
- **Complexity:** [Details]  
- **Maintainability:** [Details]  
- **Latency/Scalability:** [Details]

**Applicability:**  
[Where/when this approach is best used.]

**Recommendation:**  
[When/why to choose this approach.]

---

### 3. [Approach Name]

**Description:**  
[Brief explanation of the approach.]

**Trade-offs:**  
- **Resource Usage:** [Details]  
- **Complexity:** [Details]  
- **Maintainability:** [Details]  
- **Latency/Scalability:** [Details]

**Applicability:**  
[Where/when this approach is best used.]

**Recommendation:**  
[When/why to choose this approach.]

---

### Comparison Table

| Approach Name      | Resource Usage | Complexity | Maintainability | Latency/Scalability | Applicability         | Best Use Case                  |
|--------------------|---------------|------------|-----------------|---------------------|-----------------------|-------------------------------|
| [Approach 1]       | [Value]       | [Value]    | [Value]         | [Value]             | [Value]               | [Value]                       |
| [Approach 2]       | [Value]       | [Value]    | [Value]         | [Value]             | [Value]               | [Value]                       |
| [Approach 3]       | [Value]       | [Value]    | [Value]         | [Value]             | [Value]               | [Value]                       |

---

**Reflection on Prompting Technique:**  
The Recipe pattern was chosen to ensure the LLM systematically explores multiple alternatives, analyzes their trade-offs, and presents structured, comparative recommendations. This approach accelerates ideation and supports informed decision-making for [domain] challenges.

---

**Template Name:** Recipe Pattern Algorithm Design & Trade-off Analysis Prompt Template

---

**Is it worth it?**  
Yes! This template:
- Ensures systematic, multi-perspective solution generation and comparison.
- Produces actionable, real-world recommendations.
- Is easily adaptable for any technical, engineering, or design scenario.
- Supports best practices in prompt engineering and decision support.