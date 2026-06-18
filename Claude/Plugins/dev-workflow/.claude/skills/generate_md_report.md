# Skill: generate_md_report

Generate a Markdown analysis report.

## Required Inputs

- `ticket`: Structured ticket data (id, title, requirements, acceptance_criteria)
- `analysis`: Codebase analysis output (affected_files, risks, edge_cases, patterns_found)
- `implementation_plan`: Array of step objects
- `open_questions`: Array of unanswered questions
- `review_iteration`: Integer (0 = initial, 1+ = updated)
- `output_path`: Where to write the file
- `test_strategy`: Object with `detected_convention`, `test_framework`, `patterns_summary`,
  `steps_needing_tests[]` (each: step, test_type, test_file, test_guidance)

## Markdown Structure

```markdown
# Analysis: {ticket.id} — {ticket.title}

**Generated**: {date}
**Review Iteration**: {review_iteration}
**Implementation Steps**: {count}
**Affected Files**: {count}

---

## Ticket Summary

{ticket.description}

### Requirements

- {requirement 1}
- {requirement 2}

### Acceptance Criteria

- [ ] {criterion 1}
- [ ] {criterion 2}

---

## Codebase Analysis

| File | Role | Impact |
|------|------|--------|
| `path/to/file.ts` | description | description |

### Relevant Patterns Found

- {pattern 1}

---

## Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| description | 🔴 High / 🟡 Medium / 🟢 Low | mitigation |

### Edge Cases

- {edge case 1}

---

## Implementation Plan

### Step 1: {title}

**Description**: {description}

**Files**:
- `path/to/file.ts`

**Test**: `{test_command}`

**Test Type**: `{test_type}`

**Test Guidance**: {test_guidance — omit this line when test_type is "none" or null}

**Commit**: `{commit_message}`

---

### Step 2: ...

---

## Open Questions

- {question 1}
- {question 2}

---

## Test Strategy

**Framework**: {test_strategy.test_framework or "Unknown"}
**Convention**: {test_strategy.detected_convention or "Not detected"}

### Pattern Summary

{test_strategy.patterns_summary}

### Coverage by Step

| Step | Test Type | Test File | Approach |
|------|-----------|-----------|----------|
| {step} | {test_type} | `{test_file}` | {test_guidance — first sentence} |

{if test_strategy.detected_convention is null}
> **Warning**: No existing tests found. Confirm test framework before implementing test steps.
{/if}

---

## Review History

### Iteration 1 (if applicable)
- Addressed: {finding}
- Updated: {what changed}
```

## Writing the File

Write the complete Markdown to `output_path` using the Write tool.

For updates (review_iteration > 0), edit the existing file in place — add a "Review Iteration N" section at the top of
the Review History block and update the affected sections.
