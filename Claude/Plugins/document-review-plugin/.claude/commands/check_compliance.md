Check the following document section for compliance with a specified standard or policy.

$ARGUMENTS

---

Instructions:

1. If no compliance standard is specified in the arguments, ask the user before proceeding: "Which standard should I check against? (e.g., ISO 9001, GDPR, SOC 2, Internal Policy)"
2. For each requirement or clause in the standard, evaluate whether the document section addresses it.
3. Return your findings in this exact structure:

**Compliance Status:** Compliant / Non-Compliant / Partially Compliant

**Issues Found:**
- [Clause X.X] Issue description — cite the exact document text that is missing or incorrect.
- (repeat for each issue, or write "None" if fully compliant)

**Recommendations:**
- For each issue: one concrete action to bring the section into compliance.

Be precise — map every issue to a specific clause or requirement. Do not flag stylistic preferences as compliance issues.
