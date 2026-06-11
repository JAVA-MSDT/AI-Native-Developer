# Explicit Errors Identification and Correction Prompt Template

## Solution Summary

This template operationalizes the **Explicit Errors Prompting Pattern** in a stepwise, Chain-of-Thought manner: for each sentence, the LLM (1) identifies errors, (2) highlights the error span using [brackets], (3) explains the issue, and (4) provides a corrected version. The template includes cues to check for grammar, spelling, punctuation, logic, and subtle issues (e.g., tense, modifiers, homophones). This structured approach ensures clarity, actionable feedback, and preservation of the original meaning, making it ideal for proofreading, editing, and educational use.

---

## Prompt

You are an expert proofreader. Review the following text for grammatical, spelling, punctuation, and logical errors. For each error you find:

1. Quote the original sentence, highlighting the error span in [brackets].
2. Briefly explain the issue.
3. Provide a corrected version of the sentence.

**Instructions and Constraints:**
- Highlight only the error span in [brackets] within the original sentence.
- Preserve the original meaning and intent.
- Maintain logical and tense consistency throughout the text.
- Avoid stylistic rewrites unless necessary for clarity or correctness.
- Do not introduce new errors.
- Check for subtle issues, including punctuation, misplaced modifiers, and homophones.

**Text for Review:**  
> [Insert your paragraph or text here.]

---

## Output

### List of Identified Errors and Corrections

1. **Error:** "[Original sentence with error span in brackets]"  
   - **Explanation:** [Brief explanation of the issue.]  
   - **Correction:** [Corrected version of the sentence.]

[Repeat for each error found.]

---

## Final Corrected Text

[Paste the fully corrected version of the text here.]
