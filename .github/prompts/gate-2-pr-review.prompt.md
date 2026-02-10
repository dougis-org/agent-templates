---
name: "Gate 2: PR Review"
description: "Review and approve the pull request before merge"
mode: "ticket-orchestrator"
---

# Gate 2: PR Review (Human Checkpoint)

**Goal:** Present the pull request details for human approval.
The workflow cannot proceed to code review and merge until the human explicitly approves.

---

## Prerequisites

Before presenting this gate, verify:

- `currentPhase == "PR_CHECKPOINT"`
- `checkpoints.planApproved == true` (Gate 1 was passed)
- PR exists at `state.prUrl` (GitHub API confirms)
- All local tests pass
- All local linters pass

---

## Presentation

Load PR details and implementation artifacts from state, then present:

```text
=== PR CHECKPOINT ===

Ready for human review of implementation pull request.

Ticket: [TICKET_ID] — [TITLE]
Plan:   [PLAN_FILE_PATH]

───────────────────────────────────────
PULL REQUEST
───────────────────────────────────────

Title:   [PR_TITLE]
URL:     [PR_URL]
Branch:  [BRANCH_NAME] → [DEFAULT_BRANCH]
Commits: [COMMIT_COUNT]
Files:   [FILE_COUNT] changed

───────────────────────────────────────
ACCEPTANCE CRITERIA COVERAGE
───────────────────────────────────────

| AC # | Criterion | Status | Test Evidence |
|------|-----------|--------|---------------|
| 1    | [AC_1]    | ✅ PASS | [test file/method] |
| 2    | [AC_2]    | ✅ PASS | [test file/method] |
| ...  | ...       | ...    | ...           |

Coverage: [N]/[TOTAL] ACs verified

───────────────────────────────────────
TEST RESULTS
───────────────────────────────────────

Unit Tests:        [COUNT] — PASS
Integration Tests: [COUNT] — PASS
Total Coverage:    [PERCENT]%

───────────────────────────────────────
QUALITY GATE STATUS
───────────────────────────────────────

Build:        ✅ PASS
Linting:      ✅ PASS
Duplication:  ✅ OK
Complexity:   ✅ OK
Coverage:     ✅ No regression
Documentation: ✅ Updated

───────────────────────────────────────
IMPLEMENTATION SUMMARY
───────────────────────────────────────

[Brief description of what was implemented, key design decisions,
 any deviations from the plan with justification]

───────────────────────────────────────
AVAILABLE ACTIONS
───────────────────────────────────────

Review PR diff:     [PR_URL]/files
Review test files:  [list key test files]
Review plan:        [PLAN_FILE_PATH]
Ask questions about any aspect

───────────────────────────────────────
DECISION REQUIRED
───────────────────────────────────────

Approve PR and proceed to code review and merge?

  approve  — Proceed to final code review and merge (Segment 3)
  reject   — Provide feedback for implementation changes
  details  — Ask questions or review specific files
```

---

## Response Handling

### On Approval

Trigger words: `approve`, `yes`, `proceed`, `merge`, `lgtm`, `go`, `looks good`

Actions:

1. Set `checkpoints.prApproved = true`
2. Set `checkpoints.prApprovedAt = [ISO_TIMESTAMP]`
3. Update `currentPhase = "CODE_REVIEW"`
4. Append to phase history:

   ```json
   {
     "phase": "PR_CHECKPOINT",
     "status": "completed",
     "summary": "PR approved by human reviewer"
   }
   ```

5. Post approval comment to PR:

   ```text
   PR approved for merge. Proceeding to final code review (Segment 3).
   ```

6. Return control to orchestrator to dispatch Segment 3

### On Rejection

Trigger: any response without an approval keyword, or explicit `reject`, `no`, `changes needed`

#### With Feedback

If the user provides specific feedback text:

1. Parse feedback to determine routing:

   | Feedback about | Route to | Phase reset |
   | -------------- | -------- | ----------- |
   | Code logic, bugs, tests | work-ticket | IMPLEMENTATION |
   | Missing test cases | work-ticket | IMPLEMENTATION |
   | PR title, description, format | cut-pr | PR_CREATION |
   | Code quality, duplication | work-ticket | IMPLEMENTATION |
   | Missing documentation | work-ticket | IMPLEMENTATION |

2. Store feedback in phase history:

   ```json
   {
     "phase": "PR_CHECKPOINT",
     "status": "failed",
     "summary": "Rejected by reviewer",
     "error": "[USER_FEEDBACK]"
   }
   ```

3. Re-invoke the appropriate sub-agent with context:

   ```text
   User Feedback on PR:
   "[USER_FEEDBACK]"

   Please address these concerns. After changes, the implementation
   will go through self-review and a new/updated PR will be presented
   for re-approval.
   ```

4. After sub-agent completes:
   - If code changed → re-run LOCAL_REVIEW → update PR or create new → return to Gate 2
   - If only PR metadata changed → return to Gate 2 directly

#### Without Feedback

If the user just says "no" or "reject":

1. Ask: "What needs to change? (code / tests / PR description / documentation / other)"
2. Wait for response
3. Process as rejection with feedback (above)

### On Details Request

Trigger: `details`, `more info`, `questions`, `show me`, `diff`, `what about`

1. Answer the user's specific question
2. Offer to show:
   - Full PR diff
   - Individual file changes
   - Test output
   - Plan sections for comparison
3. After answering, return to the decision prompt:

   ```text
   Ready to decide? (approve / reject / more questions)
   ```

---

## Safety Rules

- **Never auto-approve:** The gate requires explicit human approval text
- **Never skip:** Even if all quality gates pass, human review is mandatory
- **Never auto-merge:** Merge only happens in Segment 3 after approval
- **Preserve context:** All feedback and decisions recorded in state
- **Unlimited iterations:** The gate cycles until approval (no timeout)
- **No code changes at this gate:** Changes happen via sub-agent re-invocation only
