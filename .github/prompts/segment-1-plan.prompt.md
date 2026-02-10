---
name: "Segment 1: Discovery → Planning → Analysis"
description: "Discovers ticket, creates implementation plan, and validates it"
mode: "ticket-orchestrator"
---

# Segment 1: Discovery → Planning → Analysis

**Goal:** Execute the first three phases of the ticket workflow automatically,
producing a validated implementation plan ready for human review.

---

## Inputs

**From orchestrator state:**

- `TICKET_ID`: Ticket identifier (GitHub `#N` or Jira `KEY-N`)
- `PLATFORM`: `"github"` or `"jira"`
- `TICKET_URL`: Full ticket URL
- Cached ticket data: title, description, ACs, labels
- `STATE_FILE_PATH`: Path to orchestrator state JSON
- `MAX_RETRIES`: Retry limit per phase

---

## Phase 1: DISCOVERY

**Purpose:** Load and validate the ticket, prepare context for planning.

### Steps

1. **Fetch full ticket context** using platform API:
   - GitHub: `mcp_gh-issues_issue_read` with method `"get"`
   - Jira: fetch via API (if available)

2. **Extract and cache:**
   - Title and description
   - Acceptance criteria (numbered list)
   - Labels, milestone, priority
   - Linked issues and dependencies
   - Comments with additional context

3. **Identify blocking questions:**
   - Missing ACs or vague requirements
   - Unresolved dependencies
   - If CRITICAL gaps: note them for plan to address (do not stop workflow)

4. **Update state:**

   ```json
   {
     "currentPhase": "DISCOVERY",
     "phaseHistory": [{ "phase": "DISCOVERY", "status": "completed", "summary": "..." }]
   }
   ```

### Quality Gate

| Check | Validation |
| ----- | --------- |
| Ticket loaded | Title and description non-empty |
| Platform confirmed | GitHub or Jira API responded successfully |

**On failure:** Retry fetch once. If persistent, escalate with:
"Ticket [ID] could not be loaded from [PLATFORM].
Verify the ticket exists and retry."

---

## Phase 2: PLANNING

**Purpose:** Create an execution-ready implementation plan using TDD principles.

### Steps — PLANNING

1. **Invoke plan-ticket sub-agent** (or apply plan-ticket prompt logic):

   Provide context:

   - Ticket ID, platform, full ticket data
   - Target plan path: `docs/plan/tickets/{TICKET_ID}-plan.md`
   - Reference: `.github/prompts/plan-ticket.prompt.md` for full planning logic
   - Reference: `.github/agents/plan-ticket.agent.md` for mode guardrails

2. **Plan-ticket responsibilities:**
   - Fetch ticket details (already cached — pass through)
   - Scan codebase for reusable patterns
   - Evaluate decomposition needs
   - Create plan file with all 11 required sections:
     1. Summary
     2. Assumptions and Open Questions
     3. Acceptance Criteria (normalized)
     4. Approach and Design Brief
     5. Step-by-Step Implementation Plan (TDD)
     6. Effort, Risks, Mitigations
     7. File-Level Change List
     8. Test Plan
     9. Rollout and Monitoring Plan
     10. Handoff Package
     11. Traceability Map

3. **Validate plan output:**
   - File exists at expected path
   - All 11 sections present (search for section headers)
   - No ALL-CAPS placeholder text remaining
   - ACs in Section 3 are numbered and testable
   - Decomposition in Section 6 has explicit decision

4. **Update state:**

   ```json
   {
     "currentPhase": "PLANNING",
     "planFilePath": "docs/plan/tickets/{TICKET_ID}-plan.md",
     "phaseHistory": [..., { "phase": "PLANNING", "status": "completed", "artifacts": ["plan file path"] }]
   }
   ```

### Quality Gate — PLANNING

| Check | Validation |
| ----- | --------- |
| Plan file exists | File at expected path |
| Plan completeness | All 11 section headers found |
| No placeholders | No `{{...}}` or ALL-CAPS placeholder patterns |
| Decomposition evaluated | Section 6 contains explicit decision |

**On failure:** Retry plan-ticket with error context (up to `maxRetries`). Common issues:

- Missing sections → re-invoke with explicit section checklist
- Placeholder text → re-invoke with "replace all placeholders" instruction

---

## Phase 3: ANALYSIS

**Purpose:** Validate the plan against ticket requirements and identify gaps.

### Steps — ANALYSIS

1. **Invoke analyze-ticket sub-agent** (or apply analyze-ticket prompt logic):

   Provide context:

   - Ticket ID and full ticket data
   - Plan file path
   - Reference: `.github/prompts/analyze-ticket.prompt.md` for full analysis logic

2. **Analyze-ticket responsibilities:**
   - Load plan file and ticket data
   - Validate AC coverage (every AC has test + implementation step)
   - Check edge-case coverage
   - Verify TDD ordering (tests before implementation)
   - Assess decomposition recommendation
   - Check utility reuse citations
   - Verify parameterized test strategy
   - Generate severity-rated analysis report

3. **Evaluate analysis results:**
   - **CRITICAL findings:** Flag for remediation before advancing
   - **HIGH findings:** Document; may require plan updates
   - **MEDIUM/LOW findings:** Document; proceed with awareness

4. **If CRITICAL findings exist:**
   - Re-invoke plan-ticket with specific remediation instructions
   - Re-run analysis after plan update
   - Repeat up to `maxRetries`

5. **Record analysis artifacts:**

   ```json
   {
     "segment1_analysisSummary": "2 HIGH, 3 MEDIUM findings; no CRITICAL",
     "segment1_riskLevel": "MEDIUM",
     "segment1_decomposition": "single-ticket",
     "segment1_acCount": 8
   }
   ```

6. **Update state:**

   ```json
   {
     "currentPhase": "PLAN_CHECKPOINT",
     "phaseHistory": [..., { "phase": "ANALYSIS", "status": "completed", "summary": "..." }]
   }
   ```

### Quality Gate — ANALYSIS

| Check | Validation |
| ----- | --------- |
| Analysis complete | Analysis report generated |
| No CRITICAL issues | Zero CRITICAL severity findings |

**On failure (CRITICAL issues persist after retries):** Escalate to human at the Plan Checkpoint with analysis details included.

---

## Segment Output

On successful completion of all three phases, the segment produces:

```text
=== SEGMENT 1 COMPLETE ===

Phases completed:
  ✅ DISCOVERY — Ticket loaded from [PLATFORM]
  ✅ PLANNING  — Plan created ([SECTION_COUNT] sections)
  ✅ ANALYSIS  — [FINDING_COUNT] findings ([SEVERITY_SUMMARY])

Plan file: docs/plan/tickets/[TICKET_ID]-plan.md
Risk level: [LOW/MEDIUM/HIGH]
Decomposition: [single-ticket / split-N]
AC count: [N]

Ready for human review at PLAN_CHECKPOINT.
```

State transitions to `currentPhase = "PLAN_CHECKPOINT"` and control returns to the orchestrator for Gate 1 presentation.

---

## Error Recovery

| Error | Action | Retry? |
| ----- | ------ | ------ |
| Ticket fetch fails | Retry once, then escalate | Yes (1x) |
| Plan-ticket produces incomplete plan | Re-invoke with missing section list | Yes (maxRetries) |
| Analysis finds CRITICAL issues | Re-invoke plan-ticket with remediation | Yes (maxRetries) |
| All retries exhausted | Escalate to human with full error context | No |
