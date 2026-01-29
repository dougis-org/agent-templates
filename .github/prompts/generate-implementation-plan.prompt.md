---
description: Break LLD into executable milestones with derived timeline from parallel work capacity (delivery-manager mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `delivery-manager` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

---

**Goal:** Create implementation plan with milestones, dependencies, and derived timeline.

> Output: Implementation plan with milestone breakdown. Timeline is DERIVED from work, not constrained.

## Inputs

Required:
- Initiative ID: {{INIT_ID}}
- LLD Document: `docs/design/lld/{{INIT_ID}}-lld.md`
- **Number of parallel workers:** {{WORKER_COUNT}} (engineers/teams working simultaneously)

Optional:
- LLD Review: `docs/design/lld/{{INIT_ID}}-lld-review.md`
- Risk factors: {{RISK_FACTORS}}

---

## Execution Steps

### 1. Load LLD Context
- Read approved LLD
- Extract components, APIs, database schema, integrations
- Identify feature flags and deployment requirements

### 2. Ask for Parallel Capacity
If not provided, ask user:
- "How many engineers/teams will work in parallel on this initiative?"
- This determines timeline calculation

### 3. Identify Work Units
Group LLD components into logical milestones:
- Related components (same feature area)
- Shared dependencies (database, auth, etc.)
- Natural boundaries (API, data layer, UI)

### 4. Define Milestone Objectives
For each milestone:
- Clear objective (what value delivered)
- Acceptance criteria (how to verify)
- Components/features included
- Independently valuable (can ship alone)

### 5. Map Dependencies
For each milestone:
- Prerequisites (what must complete first)
- Blocking relationships (explicit only, no assumptions)
- Shared resources (database schema, APIs)
- Validate DAG (no circular dependencies)

### 6. Estimate Effort
For each milestone:
- Count API endpoints, tables, integrations
- Assess complexity (simple CRUD vs. complex logic)
- Consider team familiarity
- Estimate story points (Fibonacci: 1, 2, 3, 5, 8, 13, 21)
- Milestone size: 1-6 weeks worth of work

### 7. Calculate Timeline (DERIVED)
- Total work = Sum of all milestone story points
- Per-worker capacity = Assume velocity (e.g., 10 SP per week)
- Identify critical path (longest dependency chain)
- Identify parallel work opportunities
- **Timeline = Critical path duration + parallel work slots / worker count**

### 8. Plan Rollout Strategy
For each milestone:
- Deployment approach (canary, blue-green, feature flag)
- Rollback plan
- Monitoring requirements
- Success metrics

### 9. Create Implementation Plan
**Output:** `docs/plan/{{INIT_ID}}/implementation.md`

Structure:
```markdown
# Implementation Plan: {{INIT_ID}}

## Overview
- **Initiative:** [Name]
- **Parallel Workers:** {{WORKER_COUNT}}
- **Total Effort:** [X story points]
- **Derived Timeline:** [Y weeks]
- **Critical Path:** [Z weeks]

## Timeline Calculation
- Total Story Points: [Sum]
- Assumed Velocity: [SP per worker per week]
- Critical Path: [Milestones in sequence]
- Parallel Opportunities: [Milestones that can run simultaneously]
- **Derived Duration:** [Weeks calculated from work breakdown]

## Milestone Breakdown

### M1: [Milestone Name] (Priority: CRITICAL_PATH)
- **Objectives:** [What value delivered]
- **Acceptance Criteria:**
  - [ ] AC1
  - [ ] AC2
- **Components Included:** [List from LLD]
- **Estimated Effort:** [Story points]
- **Duration:** [Weeks for this milestone]
- **Dependencies:** [None OR M#, M#]
- **Success Metrics:** [How to measure success]
- **Rollout Plan:**
  - Phase 1: Internal testing
  - Phase 2: Canary (5%)
  - Phase 3: GA (100%)
- **Rollback:** [How to rollback if issues]

[Repeat for M2, M3, ...]

## Parallel Work Schedule
| Week | Worker 1 | Worker 2 | Worker 3 |
|------|----------|----------|----------|
| 1-2  | M1       | M2       | M3       |
| 3-4  | M4       | M2       | M3       |
| 5-6  | M4       | M5       | M6       |

## Critical Path
M1 → M4 → M7 (Total: X weeks)

## Dependency Graph
```
M1 (no deps)
├─ M2 (depends on M1)
├─ M3 (depends on M1)
└─ M4 (depends on M2, M3)
   └─ M5 (depends on M4)
```

## Risk Register
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| [Risk 1] | MED | HIGH | [How to mitigate] |
| [Risk 2] | LOW | MED | [How to mitigate] |

## Timeline Validation Note
If derived timeline ({{DERIVED_WEEKS}} weeks) does not meet stakeholder expectations:
- **Do NOT compress artificially**
- **Option 1:** Re-evaluate milestone scope (reduce features)
- **Option 2:** Increase parallel workers
- **Option 3:** Negotiate timeline with stakeholders
```

### 10. Quality Gate Check
- [ ] All LLD components assigned to a milestone
- [ ] Milestones are independently valuable
- [ ] Dependencies form a DAG (no circular)
- [ ] Effort estimates justified
- [ ] Critical path identified
- [ ] Parallel opportunities maximized
- [ ] Timeline DERIVED from work (not artificially constrained)
- [ ] Each milestone has measurable success criteria
- [ ] Rollout strategy is low-risk

### 11. Present to User
- Show plan summary, timeline calculation, and dependency graph
- Highlight critical path and parallel opportunities
- If timeline is longer than expected: Present options (scope reduction, more workers, negotiate timeline)
- Ask: "Ready for implementation plan review phase?"

---

## Out of Scope

- ❌ No timeline compression without scope reduction or resource increase
- ❌ No ticket creation (later phase)
- ❌ No developer assignments (review phase may address)
- ❌ No sprint planning (milestone evaluation phase)

---

**Next Phase:** `review-implementation-plan.prompt.md` (Phase 4, Step 2)
