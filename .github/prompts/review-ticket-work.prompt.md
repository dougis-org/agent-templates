---
mode: code-review
description: 'Review local changes against a GitHub issue before pushing to remote.'
---

# Review Ticket Work

Review the current local changes against the requirements and acceptance criteria from a GitHub issue or Jira ticket before pushing to remote.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

## Required Input

**Ticket Identifier:** {{TICKET_ID}} (GitHub issue number or Jira ticket key)

> ⚠️ This prompt accepts either a GitHub issue number (numeric) or Jira ticket key (alphanumeric). The ticket will be used to validate that all acceptance criteria are met and the implementation aligns with requirements.

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

1. **Fetch ticket details** using appropriate platform API (GitHub or Jira) for the provided identifier
2. **Extract and document:**
   - Summary and description
   - Acceptance Criteria (AC)
   - Any linked requirements or specifications
   - Business context and stakeholder expectations
3. **Identify local changes** by examining unstaged/staged git changes in the repository

---

### Step 2: Acceptance Criteria Validation
Follow `.github/prompts/includes/ac-validation-workflow.md` and use the table from `.github/prompts/includes/ac-validation-template.md`.

---

### Step 3: Code Quality Analysis

Run quality analysis on all changed files:

1. **(Optional)** If CI/Codacy central scans are not available, run Codacy CLI analysis on each modified file to reproduce or triage; otherwise rely on CI/Codacy central results
2. **Review for:**
   - Linting violations
   - Security vulnerabilities
   - Code smell patterns
   - Test coverage gaps
3. **Verify all tests pass** locally (unit + integration)
4. **Confirm no quality gate regressions** compared to baseline

---

### Step 4: Duplication Assessment
Follow the duplication assessment guidance in `.github/prompts/includes/review-quality-core.md`.

---

### Step 5: Complexity Evaluation
Follow the complexity evaluation guidance in `.github/prompts/includes/review-quality-core.md`.

---

### Step 6: Business Logic Clarity
Follow the business-logic clarity guidance in `.github/prompts/includes/review-quality-core.md`.

---

### Step 7: Pre-Push Checklist

Before approving for push, confirm:

**Quality Gates:**
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Linting/formatting passes
- [ ] Codacy (CI) analysis shows no new issues; if CI results are not available, run local analysis as needed
- [ ] No unresolved TODO markers without ticket references

**Acceptance Criteria:**
- [ ] All AC verified and mapped to implementation
- [ ] Edge cases from AC are handled
- [ ] No AC gaps identified (or gaps documented with justification)

**Code Health:**
- [ ] No significant duplication introduced
- [ ] Complexity within acceptable thresholds
- [ ] Business logic is clear and documented where needed
- [ ] API changes are documented (if applicable)

**Documentation:**
- [ ] README updated if behavior changes
- [ ] Inline comments for complex logic
- [ ] Changelog entry added (if required by project)

---

## Review Summary Template

```markdown
## Review Summary: {{jiraTicket}}

### Acceptance Criteria Status
- **Total AC:** X
- **Verified:** X
- **Gaps:** X

### Quality Metrics
- **Codacy Issues:** X new / X resolved
- **Test Status:** PASS/FAIL
- **Lint Status:** PASS/FAIL

### Code Health
- **Duplication:** None / Minor / Significant
- **Complexity:** Low / Medium / High
- **Business Logic Clarity:** Clear / Needs Comments / Unclear

### Recommendation
- [ ] ✅ Ready to push
- [ ] 🟡 Ready with minor improvements suggested
- [ ] 🔴 Blocked - issues must be resolved

### Action Items (if any)
1. ...
```

---

## Escalation Triggers
Follow `.github/prompts/includes/review-escalation-triggers.md`.
