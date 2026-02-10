# Phase Transition Map

## Valid Transitions

```text
DISCOVERY       → PLANNING
PLANNING        → ANALYSIS
ANALYSIS        → PLAN_CHECKPOINT
PLAN_CHECKPOINT → IMPLEMENTATION        (on approval)
PLAN_CHECKPOINT → PLANNING              (on rejection → re-plan)
PLAN_CHECKPOINT → ANALYSIS              (on rejection → re-analyze)
IMPLEMENTATION  → LOCAL_REVIEW
LOCAL_REVIEW    → PR_CREATION
PR_CREATION     → PR_CHECKPOINT
PR_CHECKPOINT   → CODE_REVIEW           (on approval)
PR_CHECKPOINT   → IMPLEMENTATION        (on rejection → re-implement)
PR_CHECKPOINT   → PR_CREATION           (on rejection → update PR only)
CODE_REVIEW     → DONE
CODE_REVIEW     → IMPLEMENTATION        (on review changes requested)
```

---

## Segment Mapping

| Segment | Phases | Automated? | Gate After? |
| ------- | ------ | ---------- | ----------- |
| 1 — Plan | DISCOVERY → PLANNING → ANALYSIS | Yes | PLAN_CHECKPOINT |
| Gate 1 | PLAN_CHECKPOINT | **Human** | — |
| 2 — Implement | IMPLEMENTATION → LOCAL_REVIEW → PR_CREATION | Yes | PR_CHECKPOINT |
| Gate 2 | PR_CHECKPOINT | **Human** | — |
| 3 — Finalize | CODE_REVIEW → DONE | Yes | — |

---

## Gate Enforcement Rules

### PLAN_CHECKPOINT (Gate 1)

**Prerequisites (ALL must be true):**

- Plan file exists at `docs/plan/tickets/{TICKET_ID}-plan.md`
- Plan file contains all 11 required sections
- Analysis completed with no CRITICAL severity issues
- State shows `currentPhase == "PLAN_CHECKPOINT"`

**Cannot proceed to IMPLEMENTATION unless** `checkpoints.planApproved == true`

### PR_CHECKPOINT (Gate 2)

**Prerequisites (ALL must be true):**

- PR exists and is open (GitHub API confirms)
- All local tests pass
- All local linters pass
- PR has description and links to ticket
- State shows `currentPhase == "PR_CHECKPOINT"`

**Cannot proceed to CODE_REVIEW unless** `checkpoints.prApproved == true`

---

## Retry Transitions

On phase failure:

1. Increment `retryCount[phase]`
2. If `retryCount[phase] < maxRetries`: re-execute same phase
3. If `retryCount[phase] >= maxRetries`: escalate to human with options:
   - Reset retries and try again
   - Skip to next phase (with documented justification)
   - Abort workflow and preserve state

---

## Rejection Routing

### Gate 1 Rejection (Plan)

| Feedback Category | Route To | Phase Reset |
| ----------------- | -------- | ----------- |
| Scope / Requirements | plan-ticket sub-agent | PLANNING |
| Risk / Mitigation | analyze-ticket sub-agent | ANALYSIS |
| Decomposition | plan-ticket sub-agent | PLANNING |
| Ambiguous | Ask user to clarify target | — |

### Gate 2 Rejection (PR)

| Feedback Category | Route To | Phase Reset |
| ----------------- | -------- | ----------- |
| Code / Tests | work-ticket sub-agent | IMPLEMENTATION |
| PR format / description | cut-pr sub-agent | PR_CREATION |
| Quality / coverage | work-ticket sub-agent | IMPLEMENTATION |
| Ambiguous | Ask user to clarify target | — |
