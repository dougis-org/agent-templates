---
name: "Gate 1: Plan Review"
description: "Review and approve the implementation plan before coding"
mode: "ticket-orchestrator"
---

# Gate 1: Plan Review (Human Checkpoint)

**Goal:** Present the implementation plan for human approval.
The workflow cannot proceed to implementation until the human explicitly approves.

---

## Prerequisites

Before presenting this gate, verify:

- `currentPhase == "PLAN_CHECKPOINT"`
- Plan file exists at `state.planFilePath`
- Analysis completed (in phase history)
- No CRITICAL analysis findings unresolved

---

## Presentation

Load the plan file and present a structured summary:

```text
=== PLAN CHECKPOINT ===

Ready for human review of implementation plan.

Ticket: [TICKET_ID] — [TITLE]
Plan:   [PLAN_FILE_PATH]

───────────────────────────────────────
PLAN SUMMARY (Sections 1-3)
───────────────────────────────────────

Summary:
[Section 1 content — one-liner description, scope, out-of-scope]

Assumptions:
[Section 2 content — key assumptions and open questions]

Acceptance Criteria ([AC_COUNT] total):
[Section 3 content — numbered AC list]

───────────────────────────────────────
APPROACH (Section 4)
───────────────────────────────────────

[Section 4 highlights — architecture, data model, feature flags, key decisions]

───────────────────────────────────────
RISK ASSESSMENT (Section 6)
───────────────────────────────────────

Effort: [S/M/L]
Risk Level: [LOW/MEDIUM/HIGH]

Key Risks:
- [Risk 1]: [severity] — [mitigation]
- [Risk 2]: [severity] — [mitigation]
- [Risk 3]: [severity] — [mitigation]

───────────────────────────────────────
DECOMPOSITION
───────────────────────────────────────

Recommendation: [Keep as single ticket / Split into N slices]
[If split: show decomposition table with slices, dependencies, effort]

───────────────────────────────────────
ANALYSIS FINDINGS
───────────────────────────────────────

[Summary of analyze-ticket results]
- CRITICAL: [count] (must be 0 to reach this gate)
- HIGH:     [count]
- MEDIUM:   [count]
- LOW:      [count]

───────────────────────────────────────
AVAILABLE ACTIONS
───────────────────────────────────────

Review full plan: [PLAN_FILE_PATH]
Ask questions about any section

───────────────────────────────────────
DECISION REQUIRED
───────────────────────────────────────

Approve plan and proceed to implementation?

  approve  — Begin implementation (Segment 2)
  reject   — Provide feedback for plan revision
  details  — Ask questions or see full plan sections
```

---

## Response Handling

### On Approval

Trigger words: `approve`, `yes`, `proceed`, `lgtm`, `go`, `looks good`

Actions:

1. Set `checkpoints.planApproved = true`
2. Set `checkpoints.planApprovedAt = [ISO_TIMESTAMP]`
3. Update `currentPhase = "IMPLEMENTATION"`
4. Append to phase history:

   ```json
   {
     "phase": "PLAN_CHECKPOINT",
     "status": "completed",
     "summary": "Plan approved by human reviewer"
   }
   ```

5. Post approval comment to ticket (if GitHub):

   ```text
   Plan approved for implementation. Proceeding to Segment 2.
   Plan: docs/plan/tickets/[TICKET_ID]-plan.md
   ```

6. Return control to orchestrator to dispatch Segment 2

### On Rejection

Trigger: any response without an approval keyword, or explicit `reject`, `no`, `not ready`

#### With Feedback

If the user provides specific feedback text:

1. Parse feedback to determine routing:

   | Feedback about | Route to | Phase reset |
   | -------------- | -------- | ----------- |
   | Scope, requirements, ACs | plan-ticket | PLANNING |
   | Risks, mitigations | analyze-ticket then plan-ticket | ANALYSIS then PLANNING |
   | Decomposition decision | plan-ticket | PLANNING |
   | Test strategy | plan-ticket | PLANNING |

2. Store feedback in phase history:

   ```json
   {
     "phase": "PLAN_CHECKPOINT",
     "status": "failed",
     "summary": "Rejected by reviewer",
     "error": "[USER_FEEDBACK]"
   }
   ```

3. Re-invoke the appropriate sub-agent with context:

   ```text
   User Feedback on Plan:
   "[USER_FEEDBACK]"

   Please address these concerns and revise the plan accordingly.
   The updated plan will be re-presented for approval.
   ```

4. After sub-agent completes, re-run analysis if plan changed
5. Return to this gate and re-present for review

#### Without Feedback

If the user just says "no" or "reject" without details:

1. Ask: "What specific concerns should be addressed? (scope / risks / decomposition / test strategy / other)"
2. Wait for response
3. Process as rejection with feedback (above)

### On Details Request

Trigger: `details`, `more info`, `questions`, `show me`, `what about`

1. Answer the user's specific question
2. Offer to show any plan section in full
3. After answering, return to the decision prompt:

   ```text
   Ready to decide? (approve / reject / more questions)
   ```

---

## Safety Rules

- **Never auto-approve:** The gate requires explicit human approval text
- **Never skip:** Even if analysis shows zero issues, human review is mandatory
- **Preserve context:** All feedback and decisions are recorded in state
- **Unlimited iterations:** The gate cycles until approval (no timeout)
- **No implementation work:** No code changes occur until after approval
