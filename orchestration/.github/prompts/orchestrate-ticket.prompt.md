---
name: "Orchestrate Ticket Workflow"
description: "Execute complete ticket workflow across three automated segments with two human checkpoints"
mode: "ticket-orchestrator"
---

# Orchestrate Ticket Workflow

**Goal:** Drive a ticket from discovery through merge by executing three automated segments,
pausing at two human checkpoints for plan approval and PR approval.

---

## Inputs

**Required:**

- `TICKET_ID`: GitHub issue number (e.g., `#7`) or Jira ticket key (e.g., `PROJ-123`)

**Optional:**

- `MAX_RETRIES`: Max retries per phase before escalation (default: 2)
- `RESUME_FROM_PHASE`: Resume workflow from a specific phase (e.g., `IMPLEMENTATION`)

---

## Shared References

- **State Schema:** `orchestration/.github/prompts/includes/state-schema.md`
- **Phase Transitions:** `orchestration/.github/prompts/includes/phase-transitions.md`
- **Quality Gates:** `orchestration/.github/prompts/includes/quality-gates.md`
- **Segment Handoff:** `orchestration/.github/prompts/includes/segment-handoff.md`
- **Orchestration Rules:** `orchestration/.github/instructions/orchestration-rules.instructions.md`
- **Ticket Detection:** `.github/prompts/includes/ticket-detection.md`

---

## Phase 0: Initialization

### 0.1 Ticket Detection

Apply ticket detection logic from `.github/prompts/includes/ticket-detection.md`:

1. Parse input: `#\d+` → GitHub, `[A-Z]+-\d+` → Jira
2. Fetch ticket from detected platform
3. If fetch fails, try fallback platform
4. If both fail, ask user for clarification
5. Cache: `PLATFORM`, `TICKET_ID`, `TICKET_URL`, ticket title/description/ACs

### 0.2 State Management

Check for existing state file at `docs/plan/tickets/{TICKET_ID}-orchestrator-state.json`:

- **Resume requested:** Load state, display current phase, ask user to confirm continuation
- **State exists, no resume:** Ask user: "Existing state found at phase [PHASE]. Resume or start fresh?"
- **No state file:** Initialize new state per `includes/state-schema.md`

### 0.3 Confirm Workflow Start

```text
=== ORCHESTRATOR WORKFLOW ===

Ticket:   [TICKET_ID] — [TITLE]
URL:      [TICKET_URL]
Platform: [PLATFORM]
Retries:  [MAX_RETRIES] per phase

Workflow:
  Segment 1 (auto): Discovery → Planning → Analysis
  Gate 1 (human):   Review the Plan
  Segment 2 (auto): Implementation → Self-Review → Cut PR
  Gate 2 (human):   Review the PR
  Segment 3 (auto): Code Review → Merge → Done

Ready to begin? (yes / details / abort)
```

On `yes`: proceed to Segment 1.
On `details`: show full TICKET_FLOW.md summary.
On `abort`: save state and exit.

---

## Segment 1: Discovery → Planning → Analysis

**Dispatch to:** `orchestration/.github/prompts/segment-1-plan.prompt.md`

**Context to provide:**

- TICKET_ID, PLATFORM, ticket data
- State file path
- MAX_RETRIES

**Expected output from segment:**

- Plan file created at `docs/plan/tickets/{TICKET_ID}-plan.md`
- Analysis complete with severity assessment
- State updated to `currentPhase = "PLAN_CHECKPOINT"`
- Artifacts recorded in state

**Quality gate validation (per includes/quality-gates.md):**

- Plan file exists with all 11 sections
- No CRITICAL analysis findings
- Decomposition evaluated

**On success:** Advance to Gate 1.
**On failure:** Retry segment (up to maxRetries), then escalate to human.

---

## Gate 1: Plan Review (Human Checkpoint)

**Dispatch to:** `orchestration/.github/prompts/gate-1-plan-review.prompt.md`

**Present to human:**

- Plan summary (Sections 1-4 from plan file)
- Risk assessment (Section 6)
- Decomposition recommendation
- Analysis findings summary

**Human response handling:**

| Response | Action |
| -------- | ------ |
| Approve (`yes`, `approve`, `proceed`, `lgtm`) | Set `checkpoints.planApproved = true`, advance to Segment 2 |
| Reject with feedback | Route feedback to appropriate sub-agent, re-run affected phases, return to Gate 1 |
| Request details | Answer questions, show full plan, return to gate prompt |

**Enforcement:** Cannot proceed to Segment 2 without `checkpoints.planApproved == true`.

---

## Segment 2: Implementation → Self-Review → PR Creation

**Dispatch to:** `orchestration/.github/prompts/segment-2-implement.prompt.md`

**Prerequisites:**

- `checkpoints.planApproved == true`
- Plan file path available in state

**Context to provide:**

- TICKET_ID, approved plan file path
- Branch name (from planning phase or create new)
- State file path
- MAX_RETRIES

**Expected output from segment:**

- Implementation complete with all tests passing
- Self-review complete with no blocking issues
- PR created and open
- State updated to `currentPhase = "PR_CHECKPOINT"`
- PR URL and details recorded in state artifacts

**Quality gate validation:**

- All tests pass locally
- All linters pass
- PR exists with description and ticket link
- AC coverage documented

**On success:** Advance to Gate 2.
**On failure:** Retry failed phase (up to maxRetries), then escalate to human.

---

## Gate 2: PR Review (Human Checkpoint)

**Dispatch to:** `orchestration/.github/prompts/gate-2-pr-review.prompt.md`

**Present to human:**

- PR title, URL, file count, commit count
- AC coverage table (each AC mapped to test and status)
- Test results summary (unit, integration, coverage)
- Quality gate status (build, lint, duplication, complexity)
- Link to full PR diff

**Human response handling:**

| Response | Action |
| -------- | ------ |
| Approve (`yes`, `approve`, `merge`, `lgtm`) | Set `checkpoints.prApproved = true`, advance to Segment 3 |
| Reject with feedback | Route to work-ticket (code) or cut-pr (PR format), re-run, return to Gate 2 |
| Request details | Answer questions, show diffs/tests, return to gate prompt |

**Enforcement:** Cannot proceed to Segment 3 without `checkpoints.prApproved == true`.

---

## Segment 3: Code Review → Merge → Done

**Dispatch to:** `orchestration/.github/prompts/segment-3-finalize.prompt.md`

**Prerequisites:**

- `checkpoints.prApproved == true`
- PR URL available in state

**Context to provide:**

- TICKET_ID, PR URL, PR number
- State file path

**Expected output from segment:**

- Code review complete
- PR merged to default branch
- Post-merge CI verified green
- Ticket closed (or status updated)
- Final summary generated

**On success:** Transition to DONE, generate final summary.

---

## Phase DONE: Final Summary

```text
=== ORCHESTRATOR WORKFLOW COMPLETE ===

Ticket: [TICKET_ID] — [TITLE]  ✅ DONE

Timeline:
  Started:   [START_TIMESTAMP]
  Completed: [END_TIMESTAMP]
  Duration:  [DURATION]

Segments:
  1. ✅ Discovery → Planning → Analysis     [TIME]
  2. ✅ Plan Checkpoint — Approved           [TIMESTAMP]
  3. ✅ Implementation → Review → PR         [TIME]
  4. ✅ PR Checkpoint — Approved             [TIMESTAMP]
  5. ✅ Code Review → Merge                  [TIME]

Deliverables:
  Plan:          docs/plan/tickets/[TICKET_ID]-plan.md
  PR:            [PR_URL] (merged)
  Files Changed: [COUNT]
  Tests Written: [COUNT]
  Coverage:      [PERCENT]%

Quality:
  All gates passed
  All ACs verified

State: docs/plan/tickets/[TICKET_ID]-orchestrator-state.json
```

Update state to `currentPhase = "DONE"`, persist final summary.
Post completion comment to GitHub issue if applicable.

---

## Error Handling

### Sub-Agent Failure

1. Log error with phase context in phase history
2. Increment `retryCount[phase]`
3. If under `maxRetries`: re-invoke sub-agent with error context
4. If at limit: present failure to human with options:
   - Reset and retry
   - Adjust input/plan and retry
   - Abort and preserve state

### State Corruption

1. Detect invalid JSON on load
2. Create backup: `{TICKET_ID}-orchestrator-state.json.backup`
3. Offer: reset state / specify recovery phase / abort

### Invalid Ticket

1. Report format error with examples (`#7` for GitHub, `PROJ-123` for Jira)
2. Ask for corrected ID
3. Restart initialization

---

## State Visibility

User can request state inspection at any time during the workflow:

- `show state` → Display current phase, segment, checkpoints, retry counts
- `show history` → Display full phase history with timestamps
- `show artifacts` → Display all recorded artifacts
- `show plan` → Display plan file contents
