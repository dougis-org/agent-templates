---
name: "Segment 3: Code Review → Merge → Done"
description: "Final code review, PR merge, and workflow closure"
mode: "ticket-orchestrator"
---

# Segment 3: Code Review → Merge → Done

**Goal:** Complete the workflow by performing a final code review,
merging the approved PR, verifying post-merge CI, and closing the ticket.

**Prerequisite:** `checkpoints.prApproved == true`

---

## Inputs

**From orchestrator state:**

- `TICKET_ID`: Ticket identifier
- `PLATFORM`: `"github"` or `"jira"`
- `prUrl`: Approved PR URL
- `prNumber`: PR number
- `planFilePath`: Path to plan file (for AC verification)
- Cached ticket data
- `STATE_FILE_PATH`: Path to orchestrator state JSON

---

## Phase 1: CODE_REVIEW

**Purpose:** Perform final code review against acceptance criteria and quality standards.

### Steps

1. **Verify prerequisites:**
   - `checkpoints.prApproved == true` (refuse to proceed otherwise)
   - PR exists and is open (GitHub API confirms)

2. **Invoke review-pr sub-agent** (or apply review-pr prompt logic):

   Provide context:
   - Ticket ID, ticket data (ACs)
   - PR URL and number
   - Reference: `.github/prompts/review-pr.prompt.md` for review logic
   - Reference: `.github/agents/code-review.agent.md` for review mode guardrails

3. **Review-pr responsibilities:**
   - Fetch PR details and full diff
   - Map each AC to code changes
   - Review code quality (readability, design, error handling, security)
   - Check for duplication
   - Assess complexity
   - Validate test coverage
   - Verify business logic clarity
   - Generate review report with severity ratings

4. **Evaluate review results:**
   - **CRITICAL/Blocking:** Must resolve before merge
     - Route back to work-ticket for fixes
     - After fixes, re-review
   - **HIGH:** Should address; create follow-up if deferred
   - **MEDIUM/LOW:** Note in review; proceed with merge

5. **If blocking issues found:**
   - Re-invoke work-ticket with review feedback
   - Push fixes to PR branch
   - Re-run code review
   - Repeat up to `maxRetries`

6. **Submit review:**
   - If no blocking issues: APPROVE the PR
   - If issues addressed: APPROVE with comments

7. **Update state:**

   ```json
   {
     "currentPhase": "CODE_REVIEW",
     "phaseHistory": [..., { "phase": "CODE_REVIEW", "status": "completed", "summary": "..." }]
   }
   ```

### Quality Gate

| Check | Validation |
| ----- | --------- |
| Review complete | Review report generated |
| No blocking issues | Zero CRITICAL severity findings |
| CI/CD green | All status checks pass on PR |

---

## Phase 2: MERGE

**Purpose:** Merge the PR to the default branch.

### Steps — MERGE

1. **Pre-merge verification:**
   - All CI/CD status checks pass
   - PR is approved (review submitted)
   - No merge conflicts
   - Branch is up to date with base

2. **If pre-merge checks fail:**
   - CI failures → investigate and fix (route to work-ticket)
   - Merge conflicts → rebase branch against default, resolve, push
   - Branch behind → update PR branch

3. **Merge the PR:**
   - Prefer squash merge for single logical changeset
   - Use merge commit if preserving step history is valuable
   - Capture merged commit SHA

4. **Verify post-merge:**
   - Confirm merge succeeded (GitHub API shows PR as merged)
   - Verify CI/CD pipeline runs on merged commit to main
   - If post-merge CI fails: alert immediately (may need revert)

5. **Record merge artifacts:**

   ```json
   {
     "segment3_mergeCommitSha": "abc123def456",
     "segment3_mergeStrategy": "squash",
     "segment3_postMergeCiStatus": "GREEN"
   }
   ```

---

## Phase 3: CLOSE TICKET

**Purpose:** Update and close the ticket on the source platform.

### Steps — CLOSE

1. **Update ticket status:**
   - GitHub: Close the issue with state reason `"completed"`
   - Jira: Transition to Done/Closed (if API available)

2. **Post completion comment to ticket:**

   ```text
   Workflow complete. PR #[PR_NUMBER] merged to [DEFAULT_BRANCH].

   Merge SHA: [MERGE_SHA]
   Plan: [PLAN_FILE_PATH]
   ```

3. **Update state to DONE:**

   ```json
   {
     "currentPhase": "DONE",
     "phaseHistory": [..., { "phase": "DONE", "status": "completed", "summary": "..." }]
   }
   ```

---

## Segment Output

On successful completion:

```text
=== SEGMENT 3 COMPLETE ===

Phases completed:
  ✅ CODE_REVIEW — Final review passed
  ✅ MERGE       — PR merged to [DEFAULT_BRANCH]
  ✅ CLOSE       — Ticket closed

Merge SHA:      [MERGE_SHA]
Post-merge CI:  [GREEN/status]

Workflow complete. Returning to orchestrator for final summary.
```

State transitions to `currentPhase = "DONE"` and control returns to the orchestrator for the final summary output.

---

## Error Recovery

| Error | Action | Retry? |
| ----- | ------ | ------ |
| Code review finds blocking issues | Re-invoke work-ticket with fixes | Yes (maxRetries) |
| CI/CD failures on PR | Investigate, fix, push | Yes (maxRetries) |
| Merge conflicts | Rebase and resolve | Yes (1x) |
| Post-merge CI failure | Alert human, consider revert | No (escalate) |
| Ticket close fails | Log warning, continue (non-blocking) | Yes (1x) |
| All retries exhausted | Escalate to human with full context | No |
