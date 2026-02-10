---
name: "Segment 2: Implementation → Self-Review → PR Creation"
description: "Implements approved plan with TDD, self-reviews, and creates a PR"
mode: "ticket-orchestrator"
---

# Segment 2: Implementation → Self-Review → PR Creation

**Goal:** Execute the approved plan using strict TDD, self-review the implementation,
and create a pull request ready for human review.

**Prerequisite:** `checkpoints.planApproved == true`

---

## Inputs

**From orchestrator state:**

- `TICKET_ID`: Ticket identifier
- `PLATFORM`: `"github"` or `"jira"`
- `planFilePath`: Path to the approved plan file
- Cached ticket data: title, description, ACs
- `STATE_FILE_PATH`: Path to orchestrator state JSON
- `MAX_RETRIES`: Retry limit per phase
- `branchName`: Working branch (from planning, or to be created)

---

## Phase 1: IMPLEMENTATION

**Purpose:** Execute the plan with strict TDD (RED → GREEN → REFACTOR) and quality gates.

### Steps

1. **Verify prerequisites:**
   - Plan file exists and is readable
   - `checkpoints.planApproved == true` (refuse to proceed otherwise)
   - Working branch exists or create one

2. **Invoke work-ticket sub-agent** (or apply work-ticket prompt logic):

   Provide context:

   - Ticket ID, platform, full ticket data
   - Plan file path (approved)
   - Branch name
   - Reference: `.github/prompts/work-ticket.prompt.md` for full implementation logic
   - Reference: `.github/agents/work-ticket.agent.md` for mode guardrails

3. **Work-ticket responsibilities (per approved plan):**

   **Phase 2 — TDD RED:**

   - Write unit tests (nominal, boundary, error cases)
   - Write integration tests
   - Write contract/API tests (if applicable)
   - Write regression tests
   - Verify all new tests FAIL (prove validity)

   **Phase 3 — TDD GREEN:**

   - Implement domain/DTOs
   - Implement service interfaces and implementations
   - Implement data layer
   - Implement controllers/API
   - Implement config/env vars
   - Implement migrations (backward compatible)
   - Wire feature flags (default OFF)
   - Iterate until all tests pass

   **Phase 4 — Docs and Artifacts:**

   - Update README/module docs
   - Update CHANGELOG
   - Update runbooks/dashboards/alerts

   **Phase 5 — Quality Gates:**

   - All tests pass (unit + integration)
   - All linters pass
   - Schema drift check (if applicable)
   - Security/input validation review
   - Coverage maintained or improved
   - Pre-commit duplication and complexity scan

4. **Validate implementation output:**
   - All tests pass (verify via test runner)
   - Build succeeds
   - Linters clean
   - No unresolved TODO without ticket reference

5. **Update state:**

   ```json
   {
     "currentPhase": "LOCAL_REVIEW",
     "branchName": "[BRANCH_NAME]",
     "artifacts": {
       "segment2_testCount": N,
       "segment2_coveragePercent": N
     }
   }
   ```

### Quality Gate

| Check | Validation |
| ----- | --------- |
| Tests pass | Test runner returns exit code 0 |
| Build succeeds | No compile/build errors |
| Linters pass | All applicable linters clean |
| Coverage maintained | No unjustified regression |

**On failure:** Retry work-ticket with error context (up to `maxRetries`).
Common remediation:

- Test failure → fix code, not tests (never dilute tests)
- Lint failure → apply formatters, fix violations
- Coverage drop → add missing test cases

---

## Phase 2: LOCAL_REVIEW

**Purpose:** Self-review implementation before creating a PR.

### Steps — LOCAL_REVIEW

1. **Invoke review-ticket-work sub-agent** (or apply review-ticket-work prompt logic):

   Provide context:

   - Ticket ID and ticket data
   - Branch with changes
   - Reference: `.github/prompts/review-ticket-work.prompt.md` for review logic

2. **Review-ticket-work responsibilities:**
   - Map each AC to implementation files
   - Verify completeness of each AC
   - Check for code duplication
   - Assess complexity (method length, nesting, cyclomatic)
   - Verify business logic clarity
   - Produce review report

3. **Evaluate review results:**
   - **RED (blocking):** Issues that must be fixed before PR
     - Route back to work-ticket for remediation
     - Re-run local review after fixes
   - **YELLOW (minor):** Suggestions to improve quality
     - Apply improvements if feasible
     - Document accepted risks
   - **GREEN (clean):** No issues found

4. **If blocking issues found:**
   - Re-invoke work-ticket with specific fix instructions
   - Re-run local review
   - Repeat up to `maxRetries`

5. **Update state:**

   ```json
   {
     "currentPhase": "PR_CREATION",
     "phaseHistory": [..., { "phase": "LOCAL_REVIEW", "status": "completed", "summary": "..." }]
   }
   ```

### Quality Gate — LOCAL_REVIEW

| Check | Validation |
| ----- | --------- |
| Review complete | Review report generated |
| No blocking issues | Zero RED-status items |

---

## Phase 3: PR_CREATION

**Purpose:** Create a pull request from the working branch.

### Steps — PR_CREATION

1. **Pre-flight checks (per cut-pr prompt):**
   - No uncommitted changes (`git status --porcelain` is empty)
   - All commits pushed to remote
   - Branch is up to date with default branch
   - No merge conflicts (dry-run merge check)
   - Current branch is not the default branch

2. **If pre-flight fails:**
   - Uncommitted changes → commit with conventional message
   - Unpushed commits → push to remote
   - Behind default branch → pull and rebase
   - Merge conflicts → escalate to human (cannot auto-resolve)

3. **Invoke cut-pr sub-agent** (or apply cut-pr prompt logic):

   Provide context:

   - Branch name, ticket ID
   - Reference: `.github/prompts/cut-pr.prompt.md` for PR creation logic

4. **Cut-pr responsibilities:**
   - Generate semantic PR title: `<type>(<scope>): #<TICKET_ID> <summary>`
   - Load PR template (if exists in repo)
   - Generate PR description with:
     - Change summary
     - AC coverage
     - Test results
     - Quality gate status
     - Link to ticket
   - Create PR via GitHub API

5. **Validate PR output:**
   - PR exists (GitHub API confirms)
   - PR has title and description
   - PR references ticket
   - PR targets default branch

6. **Record PR artifacts:**

   ```json
   {
     "segment2_prUrl": "https://github.com/org/repo/pull/42",
     "segment2_prNumber": 42,
     "segment2_filesChanged": N,
     "segment2_qualityGateStatus": "ALL_PASS",
     "segment2_acCoverage": { "AC1": "PASS", "AC2": "PASS" }
   }
   ```

7. **Update state:**

   ```json
   {
     "currentPhase": "PR_CHECKPOINT",
     "prUrl": "[PR_URL]",
     "prNumber": N
   }
   ```

### Quality Gate — PR_CREATION

| Check | Validation |
| ----- | --------- |
| PR exists | GitHub API returns PR data |
| PR has description | Body is non-empty |
| Ticket linked | PR body or title references TICKET_ID |
| Branch clean | No uncommitted changes remain |

---

## Segment Output

On successful completion of all three phases:

```text
=== SEGMENT 2 COMPLETE ===

Phases completed:
  ✅ IMPLEMENTATION — TDD complete, all tests pass
  ✅ LOCAL_REVIEW   — Self-review clean
  ✅ PR_CREATION    — PR created

PR: [PR_URL]
Branch: [BRANCH_NAME]
Files Changed: [COUNT]
Tests Written: [COUNT]
Coverage: [PERCENT]%
Quality Gates: ALL PASS

AC Coverage:
  ✅ AC#1: [summary]
  ✅ AC#2: [summary]
  ...

Ready for human review at PR_CHECKPOINT.
```

State transitions to `currentPhase = "PR_CHECKPOINT"` and control returns to the orchestrator for Gate 2 presentation.

---

## Error Recovery

| Error | Action | Retry? |
| ----- | ------ | ------ |
| Test failures persist | Re-invoke work-ticket with failure details | Yes (maxRetries) |
| Build failure | Re-invoke work-ticket with compiler output | Yes (maxRetries) |
| Lint failures | Apply formatters, re-invoke if needed | Yes (maxRetries) |
| Local review RED items | Re-invoke work-ticket with fix list | Yes (maxRetries) |
| Merge conflicts on PR | Escalate to human (manual resolution) | No |
| PR creation fails | Retry API call, then escalate | Yes (1x) |
| All retries exhausted | Escalate to human with full context | No |
