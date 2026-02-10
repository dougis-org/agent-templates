---
applyTo: ".github/**"
description: "Global rules for the ticket orchestration system"
---

# Orchestration Rules

These rules govern the behavior of the ticket orchestration system.
They apply to all orchestration agents and prompts under `.github/`.

---

## Core Principles

### 1. Two Human Checkpoints — Non-Negotiable

The orchestration workflow has exactly two mandatory human checkpoints:

1. **Gate 1 (PLAN_CHECKPOINT):** After planning and analysis, before implementation
2. **Gate 2 (PR_CHECKPOINT):** After implementation and PR creation, before merge

No automated process may bypass these gates.
The workflow pauses indefinitely at each gate until the human provides an explicit approval or rejection.

### 2. Sequential Execution

Phases within each segment execute sequentially. No parallel execution of phases. No skipping phases within a segment.

Segment order is fixed:

```text
Segment 1 → Gate 1 → Segment 2 → Gate 2 → Segment 3
```

### 3. State Persistence

All workflow state is persisted to `docs/plan/tickets/{TICKET_ID}-orchestrator-state.json`.
Every phase transition updates this file. The workflow can be resumed from any saved phase.

### 4. Sub-Agent Delegation

The orchestrator delegates work to specialized sub-agents defined in `.github/agents/` and `.github/prompts/`.
The orchestrator does not directly write code, create plans, or perform reviews — it coordinates.

### 5. Quality Gate Enforcement

Every phase boundary has a quality gate (defined in `includes/quality-gates.md`).
The orchestrator validates these gates before advancing.
Failed gates trigger retries or human escalation.

---

## Safety Rules

### Gate Bypass Prevention

- The orchestrator MUST check `checkpoints.planApproved` before dispatching Segment 2
- The orchestrator MUST check `checkpoints.prApproved` before dispatching Segment 3
- If either checkpoint is `false`, the orchestrator MUST halt and present the gate
- No configuration, flag, or override can bypass these checks

### Retry Limits

- Each phase has a maximum retry count (default: 2)
- After exhausting retries, the orchestrator escalates to the human
- The human can choose to: reset retries, adjust inputs, or abort
- Retry counts are tracked per-phase in the state file

### No Scope Expansion

- The orchestrator follows the ticket scope and approved plan
- If a sub-agent discovers out-of-scope work, it documents it but does not execute it
- New issues or features should be raised as separate tickets

### No Automatic Merging

- PR merge only occurs in Segment 3, after Gate 2 approval
- The orchestrator never merges without both checkpoints passing

---

## Integration with Existing System

The orchestration system is an overlay on the existing agent-templates:

| Orchestration Component | Delegates To |
| --- | --- |
| Segment 1, Phase: DISCOVERY | `.github/agents/find-next-ticket.agent.md` |
| Segment 1, Phase: PLANNING | `.github/agents/plan-ticket.agent.md` |
| Segment 1, Phase: ANALYSIS | `.github/prompts/analyze-ticket.prompt.md` |
| Segment 2, Phase: IMPLEMENTATION | `.github/agents/work-ticket.agent.md` |
| Segment 2, Phase: LOCAL_REVIEW | `.github/prompts/review-ticket-work.prompt.md` |
| Segment 2, Phase: PR_CREATION | `.github/prompts/cut-pr.prompt.md` |
| Segment 3, Phase: CODE_REVIEW | `.github/agents/code-review.agent.md` |

The orchestration prompts provide coordination context; the underlying agents/prompts provide implementation logic.

---

## Error Handling Hierarchy

1. **Transient tool failure:** Retry once automatically
2. **Sub-agent failure:** Retry phase up to `maxRetries`
3. **Quality gate failure:** Fix root cause and retry, or escalate
4. **Gate rejection:** Route feedback to appropriate sub-agent, re-present gate
5. **State corruption:** Backup state, offer reset or manual recovery
6. **Unrecoverable error:** Preserve state, escalate to human with full context

---

## Observability

The orchestrator provides visibility through:

- **State file:** Full audit trail of all phases, timestamps, artifacts, errors
- **Phase history:** Append-only log of every phase execution
- **On-demand inspection:** User can request `show state`, `show history`, `show artifacts` at any time
- **Ticket comments:** Progress milestones posted to GitHub issue at segment boundaries
