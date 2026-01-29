---
mode: code-review
description: 'Review a pull request against GitHub issue acceptance criteria with focus on code quality.'
---

# Review Pull Request

Review a pull request against the requirements and acceptance criteria from a GitHub issue or Jira ticket, acting as a senior code reviewer focused on quality, maintainability, and correctness.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

## Required Inputs

**Ticket Identifier:** {{TICKET_ID}}
**Pull Request:** {{pullRequest}}

> ⚠️ This prompt requires:
> - A valid GitHub issue number or Jira ticket key containing acceptance criteria
> - A pull request reference in one of these formats:
>   - PR URL: `https://github.com/owner/repo/pull/123`
>   - Owner/repo + PR number: `owner/repo#123`

---

## Review Workflow

### Step 0: Ticket Detection & Platform Resolution

**Refer to `.github/prompts/includes/ticket-detection.md` for shared ticket detection logic.**

Apply the auto-detection steps in the include and establish PLATFORM + TICKET_ID context.

**Outputs from this step:**
- `PLATFORM` = "github" | "jira"
- `TICKET_ID` = normalized ticket identifier
- Cached ticket data: title, description, AC, labels


### Step 1: Gather Context

1. **Fetch ticket details** using the appropriate API (GitHub or Jira) for the provided identifier
2. **Extract and document:**
   - Summary and description
   - Acceptance Criteria (AC)
   - Linked requirements or specifications
   - Business context and stakeholder expectations
3. **Fetch PR details** using GitHub tools:
   - PR title, description, and linked issues
   - List of changed files
   - Diff content for each file
   - Existing review comments (if any)
   - CI/CD status checks

---

### Step 2: PR Metadata Validation

Verify PR hygiene before diving into code:

1. **Title and description:**
   - Title follows project conventions
   - Description explains the "what" and "why"
   - Jira ticket is linked or referenced
2. **Scope appropriateness:**
   - Changes are focused on the ticket scope
   - No unrelated changes bundled in
   - PR size is reviewable (flag if > 500 lines changed)
3. **Branch and target:**
   - Source branch naming follows conventions
   - Target branch is appropriate

---

### Step 3: Acceptance Criteria Validation
Follow `.github/prompts/includes/ac-validation-workflow.md` and use the table from `.github/prompts/includes/ac-validation-template.md`.

---

### Step 4: Code Quality Review

Review each changed file for quality standards:

#### 4.1 Readability & Maintainability
- [ ] Code is self-documenting with clear naming
- [ ] Complex logic has explanatory comments
- [ ] Consistent style with existing codebase
- [ ] No dead code or commented-out blocks

#### 4.2 Design & Architecture
- [ ] Single responsibility principle followed
- [ ] Appropriate abstraction level
- [ ] Dependencies are minimal and explicit
- [ ] Changes align with existing patterns

#### 4.3 Error Handling
- [ ] Errors are handled appropriately
- [ ] Error messages are informative
- [ ] Failure modes are considered
- [ ] No swallowed exceptions

#### 4.4 Security Considerations
- [ ] No hardcoded secrets or credentials
- [ ] Input validation where applicable
- [ ] No obvious injection vulnerabilities
- [ ] Sensitive data handled appropriately

---

### Step 5: Duplication Analysis
Follow the duplication assessment guidance in `.github/prompts/includes/review-quality-core.md`.

---

### Step 6: Complexity Assessment
Follow the complexity evaluation guidance in `.github/prompts/includes/review-quality-core.md`.

---

### Step 7: Test Coverage Review

Validate testing in the PR:

1. **Test presence:**
   - [ ] New functionality has corresponding tests
   - [ ] Bug fixes include regression tests
   - [ ] Edge cases are tested

2. **Test quality:**
   - [ ] Tests are meaningful (not just coverage padding)
   - [ ] Test names describe behavior
   - [ ] Assertions are specific and complete
   - [ ] No flaky test patterns

3. **Coverage impact:**
   - Note any decrease in coverage
   - Identify untested paths in new code

---

### Step 8: Business Logic Validation
Follow the business-logic clarity guidance in `.github/prompts/includes/review-quality-core.md`.

---

### Step 9: Generate Review Comments

For each issue found, prepare a review comment:

**Comment structure:**
```
**[Severity]** Category

Description of the issue or suggestion.

**Why it matters:** Brief explanation of impact.

**Suggestion:** Specific recommendation or code example.
```

**Severity levels:**
- 🔴 **Blocking:** Must be resolved before merge
- 🟡 **Warning:** Should be addressed, may defer with justification  
- 🟢 **Suggestion:** Nice-to-have improvement
- 💡 **Praise:** Highlight good patterns (don't skip this!)

---

## Review Summary Template

```markdown
## PR Review Summary

**PR:** {{pullRequest}}
**Ticket:** {{jiraTicket}}
**Reviewer:** AI Code Review Agent
**Date:** [current date]

### Acceptance Criteria Verification
| AC # | Summary | Status |
|------|---------|--------|
| 1 | ... | ✅/⚠️/❌ |

**AC Coverage:** X of Y criteria verified

### Code Quality Assessment

| Category | Rating | Notes |
|----------|--------|-------|
| Readability | ⭐⭐⭐⭐⭐ | |
| Design | ⭐⭐⭐⭐⭐ | |
| Error Handling | ⭐⭐⭐⭐⭐ | |
| Security | ⭐⭐⭐⭐⭐ | |
| Test Coverage | ⭐⭐⭐⭐⭐ | |

### Issue Summary
- 🔴 Blocking: X
- 🟡 Warnings: X
- 🟢 Suggestions: X
- 💡 Praise: X

### Duplication & Complexity
- **Duplication:** None / Minor / Significant
- **Complexity:** Low / Medium / High
- **Risk Level:** Low / Medium / High

### Recommendation
- [ ] ✅ **Approve** - Ready to merge
- [ ] 🟡 **Request Changes** - Issues must be addressed
- [ ] 💬 **Comment** - Questions or suggestions only

### Detailed Findings
[List each comment with file, line, and content]
```

---

## Escalation Triggers
Follow `.github/prompts/includes/review-escalation-triggers.md`.

---

## Post-Review Actions

After completing the review:

1. **If using GitHub review tools:** Submit review with appropriate status
2. **Document findings:** Ensure all comments are posted to the PR
3. **Update Jira (if applicable):** Add comment with review summary link
4. **Communicate blockers:** Ensure blocking issues are clearly highlighted
