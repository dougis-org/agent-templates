# Implementation Plan: Issue #7

## 1) Summary

- **Ticket:** [#7](https://github.com/dougis-org/agent-templates/issues/7)
- **One-liner:** Create an orchestrator agent that coordinates the complete ticket workflow (TICKET_FLOW.md) by delegating to sub-agents, enforcing quality gates, and pausing at human review checkpoints for approval
- **Related milestone(s):** N/A
- **Out of scope:**
  - Automatic ticket creation (orchestrator coordinates existing tickets only)
  - Modification of existing sub-agent behaviors beyond invocation
  - Parallel execution of multiple tickets simultaneously
  - Integration with external notification systems (Slack, email)
  - Automatic merging of PRs without human approval

---

## 2) Assumptions & Open Questions

### Assumptions

1. The `runSubagent` tool is available and supports delegation to agents with specific mode contexts
2. Sub-agents (`plan-ticket`, `work-ticket`, etc.) already function correctly in isolation
3. Human review checkpoints occur at exactly two stages: post-plan (Step 3.2) and post-PR (Step 5.3) per TICKET_FLOW.md
4. User feedback during rejection is provided as free-form text that can be parsed for routing
5. The orchestrator operates within a single conversation session (state is transient, not persisted across sessions)

### Open Questions (non-blocking)

1. Should the orchestrator support resumption from a mid-workflow state if the session is interrupted?
   - **Assumed:** No, orchestrator runs to completion or user explicitly restarts from a phase
2. Should there be a maximum retry count for sub-agent failures before escalating to human?
   - **Assumed:** Yes, 2 retries per phase before escalation

---

## 3) Acceptance Criteria (normalized)

1. **Agent file exists** at `.github/agents/orchestrator.agent.md` with proper YAML frontmatter, tool declarations, and behavioral guardrails
2. **Prompt file exists** at `.github/prompts/orchestrator.prompt.md` with workflow instructions
3. **Ticket platform agnostic:** Orchestrator accepts both GitHub issue numbers and Jira ticket keys, using ticket detection logic from `.github/prompts/includes/ticket-detection.md`
4. **Quality enforcement:** Orchestrator refuses to advance workflow if sub-agent reports quality gate failures; requires remediation before proceeding
5. **Sequential workflow execution:** Orchestrator follows TICKET_FLOW.md phases in order:
   - find-next-ticket (optional, if no ticket ID provided)
   - plan-ticket
   - analyze-ticket
   - [CHECKPOINT: Human Review - Plan]
   - work-ticket
   - review-ticket-work
   - cut-pr
   - [CHECKPOINT: Human Review - PR]
   - review-pr
6. **Sub-agent delegation:** Each phase invokes the correct sub-agent using `runSubagent` with appropriate mode context:
   - `find-next-ticket` → find-next-ticket.agent.md
   - `plan-ticket` → plan-ticket.agent.md
   - `analyze-ticket` → analyze-ticket.prompt.md
   - `work-ticket` → work-ticket.agent.md
   - `review-ticket-work` → review-ticket-work.prompt.md
   - `cut-pr` → cut-pr.prompt.md
   - `review-pr` → review-pr.prompt.md
7. **Human checkpoint (Plan):** After plan-ticket and analyze-ticket complete, orchestrator presents:
   - Plan summary (sections 1-4 from plan file)
   - Risk assessment
   - Decomposition recommendation (if any)
   - Explicit approval request
8. **Human checkpoint (PR):** After cut-pr completes, orchestrator presents:
   - PR URL and title
   - AC coverage summary
   - Test results summary
   - Quality gate status
   - Explicit approval request
9. **Approval flow:** On user approval ("approve", "yes", "proceed"), orchestrator advances to next phase
10. **Rejection flow:** On user rejection with feedback:
    - Orchestrator determines responsible sub-agent (planning issue → plan-ticket, code issue → work-ticket, etc.)
    - Invokes sub-agent with custom prompt incorporating user feedback
    - After remediation, returns to the checkpoint for re-approval
11. **Sub-agent completion:** Sub-agents complete their assigned task fully; only blocking questions (undefined requirements, ambiguous ACs) should pause for clarification at task start
12. **State tracking:** Orchestrator maintains internal state of:
    - Current workflow phase
    - Ticket ID and platform
    - Plan file path
    - PR URL (once created)
    - Checkpoint approval status

---

## 4) Approach & Design Brief

### Current State

- Individual agents exist for each workflow phase (`plan-ticket.agent.md`, `work-ticket.agent.md`, etc.)
- Agents are invoked manually by the user for each phase
- No automated coordination or quality enforcement across phases
- Human must manually track workflow state and transition between phases
- TICKET_FLOW.md documents the workflow but is not machine-enforced

### Proposed Changes

Create a new orchestrator agent that automates the TICKET_FLOW.md workflow by:

1. **Centralized Coordination:** Single entry point that manages the complete ticket lifecycle
2. **Delegated Execution:** Uses `runSubagent` to invoke specialized agents for each phase
3. **Quality Gate Enforcement:** Validates sub-agent outputs before advancing phases
4. **Human Checkpoints:** Pauses at defined stages to present work for approval
5. **Feedback Routing:** Intelligently routes user feedback to the appropriate sub-agent

**High-Level Architecture:**

```text
User → Orchestrator Agent
          ↓
    Workflow State Machine
          ↓
    ┌─────────────────────────────────────────────────────────────┐
    │ Phase Dispatch                                               │
    │  ├─ find-next-ticket.agent.md (discovery)                   │
    │  ├─ plan-ticket.agent.md (planning)                         │
    │  ├─ analyze-ticket.prompt.md (analysis)                     │
    │  ├─ [CHECKPOINT: Human Review]                               │
    │  ├─ work-ticket.agent.md (implementation)                   │
    │  ├─ review-ticket-work.prompt.md (self-review)              │
    │  ├─ cut-pr.prompt.md (PR creation)                          │
    │  ├─ [CHECKPOINT: Human Review]                               │
    │  └─ review-pr.prompt.md (code review)                       │
    └─────────────────────────────────────────────────────────────┘
          ↓
    Workflow Complete → Done
```

### Data Model / Schema

No database or schema changes. In-memory state tracking only.

**Orchestrator State Structure (conceptual):**

```typescript
interface OrchestratorState {
  ticketId: string;              // "#7" or "PROJ-123"
  platform: "github" | "jira";
  currentPhase: WorkflowPhase;
  planFilePath?: string;         // docs/plan/tickets/7-plan.md
  prUrl?: string;                // https://github.com/owner/repo/pull/123
  checkpoints: {
    planApproved: boolean;
    prApproved: boolean;
  };
  retryCount: Record<WorkflowPhase, number>;
}

type WorkflowPhase = 
  | "DISCOVERY"
  | "PLANNING" 
  | "ANALYSIS"
  | "PLAN_CHECKPOINT"
  | "IMPLEMENTATION"
  | "LOCAL_REVIEW"
  | "PR_CREATION"
  | "PR_CHECKPOINT"
  | "CODE_REVIEW"
  | "DONE";
```

### APIs & Contracts

No new APIs. Uses existing MCP tools and `runSubagent` for delegation.

**Sub-agent invocation contract:**

```text
runSubagent({
  description: "<3-5 word phase description>",
  prompt: "<detailed task description with context from orchestrator state>"
})
```

**Expected sub-agent return:** Text summary of phase completion with:

- Status (success/failure)
- Key artifacts (file paths, URLs)
- Any blocking issues or quality failures

### Feature Flags

None required. Orchestrator execution is user-initiated and explicit.

### Config

No new environment variables. Inherits config from sub-agents.

### External Dependencies

- Existing sub-agents (all listed in AC #6)
- `runSubagent` tool capability
- GitHub/Jira MCP tools for ticket context
- File system tools for plan file access

### Backward Compatibility Strategy

N/A - New capability. Existing individual agent usage remains unchanged.

### Observability

- Progress updates at each phase transition
- Clear phase boundaries in output: `=== PHASE: PLANNING ===`
- Checkpoint summaries with structured information
- Error messages with phase context and remediation guidance
- Final workflow summary on completion

### Security & Privacy

- No sensitive data handling beyond existing sub-agents
- Orchestrator inherits security constraints from delegated agents
- Human checkpoints prevent automated actions without approval
- No credential storage or transmission

### Alternatives Considered

1. **Shell script orchestration:** Rejected - violates MCP tooling mandate, poor error handling
2. **Single monolithic agent:** Rejected - violates separation of concerns, harder to maintain
3. **Event-driven pipeline:** Rejected - over-engineered for single-session workflow
4. **GitHub Actions workflow:** Rejected - requires CI integration, not suitable for local development

---

## 5) Step-by-Step Implementation Plan (TDD)

### Phase 1: RED (Test First)

#### 1.1 Create Test Data Providers

**(New)** `.github/prompts/test-data/orchestrator-scenarios.json`:

- Test scenarios for each workflow phase
- Mock sub-agent responses (success/failure)
- Human checkpoint approval/rejection scenarios
- Feedback routing test cases

#### 1.2 Create Test Suite

**(New)** `.github/prompts/__tests__/orchestrator.test.md`:

- **Test 1:** Orchestrator accepts GitHub issue number and initializes state
- **Test 2:** Orchestrator accepts Jira ticket key and initializes state
- **Test 3:** Phase transitions occur in correct order per TICKET_FLOW.md
- **Test 4:** Sub-agent failure triggers retry (up to 2 attempts)
- **Test 5:** Human checkpoint (Plan) presents required information
- **Test 6:** Human checkpoint (PR) presents required information
- **Test 7:** Approval advances workflow to next phase
- **Test 8:** Rejection with feedback routes to correct sub-agent
- **Test 9:** Quality gate failure blocks phase advancement
- **Test 10:** Workflow completes with success summary

*Data source:* `.github/prompts/test-data/orchestrator-scenarios.json`
*Justification for parameterized:* Multiple scenarios per test (approval/rejection, GitHub/Jira, success/failure paths)

### Phase 2: GREEN (Minimal Implementation)

#### 2.1 Create Include Files

**(New)** `.github/agents/includes/orchestrator-state-management.md`:

- State initialization logic
- Phase transition rules
- Retry count tracking

**(New)** `.github/agents/includes/human-checkpoint-protocol.md`:

- Checkpoint presentation format
- Approval/rejection parsing
- Feedback extraction for routing

#### 2.2 Create Agent File

**(New)** `.github/agents/orchestrator.agent.md`:

- YAML frontmatter with description and tools array
- Purpose and Role sections
- Tool Declarations & Access (superset of all sub-agent tools + `agent` + `todo`)
- Behavioral Guardrails:
  1. Quality Gate Enforcement
  2. Human Checkpoint Protocol
  3. Sub-Agent Delegation Rules
  4. State Tracking Requirements
  5. Feedback Routing Logic
- Non-Goals section

#### 2.3 Create Prompt File

**(New)** `.github/prompts/orchestrator.prompt.md`:

- Input section (TICKET_ID, optional TARGET_DATE)
- Ticket detection (reference include)
- Workflow phases with sub-agent invocation instructions
- Checkpoint presentation templates
- Approval/rejection handling
- Error handling and retry logic
- Completion summary format

#### 2.4 Update Documentation

**(Updated)** `README.md`:

- Add "Orchestrator Agent" section under Usage
- Link to orchestrator.prompt.md

**(Updated)** `TICKET_FLOW.md`:

- Add note about orchestrator agent for automated workflow execution
- Reference orchestrator.agent.md as alternative to manual phase transitions

### Phase 3: REFACTOR

- Ensure consistent formatting across new files
- Verify all include references are correct
- Run markdownlint on all new/modified files
- Remove any code duplication between files
- Simplify prompt instructions where possible

### Phase 4: Pre-PR Quality Review (MANDATORY)

Per `.github/prompts/includes/pre-commit-quality-review.md`:

- [ ] Review for duplication within changeset
- [ ] Verify no duplicate patterns with existing agents/prompts
- [ ] Markdownlint clean on all files
- [ ] All include file references valid
- [ ] Documentation complete and accurate
- [ ] No placeholders remaining

---

## 6) Effort, Risks, Mitigations

### Effort

**Size:** Medium (M)

**Rationale:**

- 4-6 files to create (agents, prompts, includes, tests)
- Complex state machine logic but well-defined workflow
- Heavy reuse of existing patterns and sub-agents
- Testing requires multi-scenario coverage

### Risks

| Risk | Severity | Probability | Mitigation | Fallback |
|------|----------|-------------|------------|----------|
| `runSubagent` tool limitations (context loss, timeout) | HIGH | MEDIUM | Test sub-agent delegation early in implementation; document any limitations | Fall back to user-invoked phase transitions with orchestrator providing guidance only |
| Sub-agent output parsing inconsistency | MEDIUM | MEDIUM | Define strict output contract for sub-agents; validate outputs | Add output validation layer with retry on parse failure |
| State loss on session interruption | MEDIUM | LOW | Document limitation; user must restart workflow | Future enhancement: persist state to plan file |
| Human checkpoint fatigue (too many approvals) | LOW | LOW | Keep checkpoints to exactly 2 per TICKET_FLOW.md | Allow user to "fast-track" with reduced checkpoints (quality tradeoff) |
| Feedback routing misclassification | MEDIUM | MEDIUM | Use keyword matching and context analysis | Present routing options to user for confirmation |

---

## 7) File-Level Change List

### New Files

| Path | Purpose |
|------|--------|
| `.github/agents/orchestrator.agent.md` | Chatmode definition for orchestrator with tools, guardrails, non-goals |
| `.github/prompts/orchestrator.prompt.md` | Workflow instructions, checkpoint templates, error handling |
| `.github/agents/includes/orchestrator-state-management.md` | State tracking patterns, phase transitions, retry logic |
| `.github/agents/includes/human-checkpoint-protocol.md` | Checkpoint presentation, approval parsing, feedback extraction |
| `.github/prompts/test-data/orchestrator-scenarios.json` | Test data provider with workflow scenarios |
| `.github/prompts/__tests__/orchestrator.test.md` | Test suite with 10 test cases |

### Modified Files

| Path | Changes |
|------|--------|
| `README.md` | Add "Orchestrator Agent" section with usage instructions |
| `TICKET_FLOW.md` | Add orchestrator reference as automation option |

---

## 8) Test Plan

### Parameterized Test Strategy

**Data Provider:** `.github/prompts/test-data/orchestrator-scenarios.json`

**Scenarios covered:**

- GitHub issue initialization (issue #, expected platform, expected state)
- Jira ticket initialization (ticket key, expected platform, expected state)
- Phase transition sequences (current phase, action, expected next phase)
- Sub-agent response parsing (response text, expected status, expected artifacts)
- Checkpoint presentation (phase, state, expected output structure)
- Approval parsing (user input, expected decision, expected action)
- Rejection parsing (user input, expected feedback, expected target agent)
- Retry scenarios (failure count, expected behavior)

### Test Coverage by Category

| Category | Approach | Source |
|----------|----------|--------|
| Happy paths | Parameterized - full workflow completion | `orchestrator-scenarios.json#happy-path` |
| Edge cases | Parameterized - retry exhaustion, parse failures | `orchestrator-scenarios.json#edge-cases` |
| Error cases | Parameterized - sub-agent failures, invalid inputs | `orchestrator-scenarios.json#error-cases` |
| Checkpoint validation | Parameterized - approval/rejection variants | `orchestrator-scenarios.json#checkpoints` |
| Regression | Simple - single smoke test for end-to-end flow | Justification: unique integration test |
| Manual QA | Checklist below | N/A |

### Manual QA Checklist

- [ ] Run orchestrator with real GitHub issue; verify plan checkpoint presentation
- [ ] Approve at plan checkpoint; verify transition to work-ticket
- [ ] Reject at plan checkpoint with feedback; verify routing to plan-ticket
- [ ] Complete full workflow to PR merge
- [ ] Test with Jira ticket (if Jira integration available)
- [ ] Verify sub-agent retry on transient failure

---

## 9) Rollout & Monitoring Plan

### Feature Flag(s)

None. User-initiated orchestrator invocation.

### Deployment Steps

1. Merge PR to main
2. Update README documentation
3. Announce in repository discussions/changelog

### Dashboards & Key Metrics

N/A - Prompt/agent files only, no runtime metrics.

### Alerts

N/A - No runtime component.

### Success Metrics / KPIs

- Adoption: Number of orchestrator invocations (user feedback)
- Completion rate: Percentage of workflows reaching DONE state
- Checkpoint efficiency: Average time at checkpoints before approval
- Quality: Reduction in back-and-forth between phases

### Rollback Procedure

1. Revert merge commit: `git revert <commit-sha>`
2. Push to main
3. Users fall back to manual phase invocation (existing behavior)

---

## 10) Handoff Package

- **GitHub Issue:** [#7](https://github.com/dougis-org/agent-templates/issues/7)
- **Branch:** `feature/7-orchestrator-agent`
- **Plan File:** `docs/plan/tickets/7-plan.md`
- **Key Commands:**
  - Lint: `markdownlint .github/agents/orchestrator.agent.md .github/prompts/orchestrator.prompt.md`
  - Test: Run test scenarios from `.github/prompts/__tests__/orchestrator.test.md`
- **Known Gotchas:**
  - `runSubagent` tool context limitations need early validation
  - Sub-agent output parsing relies on consistent formatting
  - Session interruption loses state (document for users)

---

## 11) Traceability Map

| Criterion # | Requirement | Milestone | Task(s) | Flag(s) | Test(s) |
|-------------|-------------|-----------|---------|---------|--------|
| 1 | Agent file at correct path | N/A | Create orchestrator.agent.md | None | Test 1, 2 (file existence implicit) |
| 2 | Prompt file at correct path | N/A | Create orchestrator.prompt.md | None | Test 1, 2 (file existence implicit) |
| 3 | GitHub/Jira ticket support | N/A | Implement ticket detection | None | Test 1 (GitHub), Test 2 (Jira) |
| 4 | Quality enforcement | N/A | Implement quality gate validation | None | Test 9 |
| 5 | Sequential workflow execution | N/A | Implement state machine | None | Test 3 |
| 6 | Sub-agent delegation | N/A | Implement runSubagent calls | None | Test 3, 4 |
| 7 | Human checkpoint (Plan) | N/A | Implement plan checkpoint | None | Test 5 |
| 8 | Human checkpoint (PR) | N/A | Implement PR checkpoint | None | Test 6 |
| 9 | Approval flow | N/A | Implement approval parsing | None | Test 7 |
| 10 | Rejection flow | N/A | Implement feedback routing | None | Test 8 |
| 11 | Sub-agent completion | N/A | Document sub-agent expectations | None | Test 4 (retry validates completion) |
| 12 | State tracking | N/A | Implement state management | None | Test 3, 7, 8 |

---

## Quality Validation

| Criterion | Validation | Status |
|-----------|------------|--------|
| Decomposition Decision | Keep as single ticket - tightly coupled slices | ✅ Documented in Section 2 |
| Reuse Evidence | Agent patterns from plan-ticket.agent.md, work-ticket.agent.md; ticket detection from includes | ✅ Cited |
| Parameterized Tests | orchestrator-scenarios.json for all multi-scenario tests | ✅ Strategy in Section 8 |
| Utility Duplication | No new utilities duplicating existing; reuses includes | ✅ Verified |
| Dependency Graph | Linear workflow with checkpoints; no cycles | ✅ No cycles |
| Feature Flags | None required - user-initiated | ✅ Justified |
| Observability | Progress updates, phase boundaries, error context | ✅ Section 4 |
| Traceability | All 12 ACs mapped to tasks and tests | ✅ Section 11 |
| Rollback Strategy | Git revert to main | ✅ Section 9 |
| Security & Privacy | Inherits from sub-agents; human checkpoints enforce | ✅ Section 4 |
