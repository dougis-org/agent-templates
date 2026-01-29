---
description: Review implementation plan for feasibility, dependency correctness, and timeline realism with gap-fix workflow (delivery-manager mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `delivery-manager` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

**Gap Discovery Workflow:**
Refer to `.github/prompts/includes/gap-discovery-workflow.md` for review workflow.

---

**Goal:** Validate implementation plan feasibility before milestone evaluation.

> Output: Review report with gap list and suggested fixes. Modify plan only with user approval.

## Inputs

Required:
- Initiative ID: {{INIT_ID}}
- Implementation Plan: `docs/plan/{{INIT_ID}}/implementation.md`
- LLD Document: `docs/design/lld/{{INIT_ID}}-lld.md`

Optional:
- Team capacity information: {{TEAM_CAPACITY}}

---

## Execution Steps

### 1. Load Documents
- Read implementation plan and LLD
- Extract milestones, dependencies, effort estimates
- Note parallel worker count and derived timeline

### 2. Feasibility Assessment
- Validate timeline vs. capacity (velocity realistic?)
- Check if worker count assumption is achievable
- Assess if milestone sizing is reasonable (1-6 weeks each)

### 3. LLD Coverage Check
For each LLD component:
- [ ] Assigned to a milestone
- Flag unmapped components as BLOCKING

For each milestone:
- [ ] Contains LLD components
- Flag milestones with vague component references

### 4. Dependency Analysis
For each milestone dependency:
- [ ] Dependency is justified (why needed?)
- [ ] Dependency is transitive closure correct
- [ ] No circular dependencies (DAG validation)

Build dependency graph and validate:
- Critical path correctly identified
- Parallel opportunities not missed

### 5. Effort Estimation Validation
For each milestone:
- [ ] Story points justified (complexity factors cited)
- [ ] Effort aligns with component count
- [ ] Risk buffer included for unknowns

Check for:
- Overly optimistic estimates (too small for complexity)
- Overly pessimistic estimates (inflated without justification)

### 6. Milestone Independence Check
For each milestone:
- [ ] Independently valuable (shippable alone)
- [ ] Can be developed in parallel (if no dependencies)
- [ ] Success metrics measurable

### 7. Rollout & Deployment Validation
For each milestone:
- [ ] Rollout plan defined (phased approach)
- [ ] Rollback strategy clear
- [ ] Monitoring requirements specified
- [ ] Feature flags used where appropriate

### 8. Risk Identification
- External dependencies (third-party systems)
- Team skill gaps (unfamiliar tech)
- Timeline pressure (derived timeline too long)
- Resource constraints (worker count insufficient)

### 9. Resource Allocation Assessment
- Worker count realistic for organization?
- Skills aligned with milestone needs?
- Dependencies on specific individuals flagged?

### 10. Create Review Report
**Output:** `docs/plan/{{INIT_ID}}/implementation-review.md`

Structure:
```markdown
# Implementation Plan Review: {{INIT_ID}}

## Review Summary
- **Reviewer:** [Agent/Name]
- **Date:** {{DATE}}
- **Plan Version:** [Ref or date]
- **Decision:** [APPROVED / APPROVED_WITH_ADJUSTMENTS / REMAND / ESCALATE]

## Feasibility Assessment
- **Timeline Realistic:** [✅/⚠️/❌]
- **Worker Count Achievable:** [✅/⚠️/❌]
- **Milestone Sizing Reasonable:** [✅/⚠️/❌]
- **Velocity Assumption:** [X SP per worker per week - realistic?]

## LLD Coverage Check
| LLD Component | Assigned Milestone | Status |
|---------------|-------------------|--------|
| Component A   | M1                | ✅     |
| Component B   | UNMAPPED          | ❌     |

**Unmapped Components:** [List with BLOCKING severity]

## Dependency Analysis
| Milestone | Dependencies | Justified | DAG Valid | Status |
|-----------|-------------|-----------|-----------|--------|
| M1        | None        | N/A       | ✅        | ✅     |
| M2        | M1          | ✅        | ✅        | ✅     |
| M3        | M1, M2      | ⚠️        | ✅        | ⚠️     |

**Circular Dependencies:** [List any - BLOCKING]
**Critical Path Validation:** [Correct: ✅/❌]
**Missed Parallel Opportunities:** [List any]

## Effort Estimation Validation
| Milestone | Estimate | Justification | Realistic | Status |
|-----------|----------|---------------|-----------|--------|
| M1        | 21 SP    | ✅ (10 APIs)  | ✅        | ✅     |
| M2        | 5 SP     | ❌ (vague)    | ⚠️        | ⚠️     |

## Milestone Independence Check
| Milestone | Independently Valuable | Parallelizable | Metrics | Status |
|-----------|------------------------|----------------|---------|--------|
| M1        | ✅                     | ✅             | ✅      | ✅     |
| M2        | ⚠️ (partial value)     | ✅             | ✅      | ⚠️     |

## Rollout & Deployment Review
| Milestone | Rollout Plan | Rollback | Monitoring | Feature Flags | Status |
|-----------|--------------|----------|------------|---------------|--------|
| M1        | ✅           | ✅       | ✅         | ✅            | ✅     |
| M2        | ⚠️ (vague)   | ✅       | ⚠️         | ✅            | ⚠️     |

## Gap Analysis

### BLOCKING Gaps
1. **Gap:** [Description]
   - **Location:** [Milestone/Section]
   - **Severity:** BLOCKING
   - **Impact:** [Why blocking]
   - **Suggested Fix:** [Milestone adjustment, dependency fix, estimate revision]

### HIGH Priority Gaps
[Same structure]

### MEDIUM Priority Gaps
[Same structure]

## Risk Identification
- [Risk 1: External dependency on System X] - Mitigation: [...]
- [Risk 2: Team skill gap on Tech Y] - Mitigation: [...]

## Resource Allocation Assessment
- [✅/⚠️] Worker count realistic for org
- [✅/⚠️] Skills aligned with milestone needs
- [✅/⚠️] No single points of failure (specific individuals)

## Decision
**[APPROVED / APPROVED_WITH_ADJUSTMENTS / REMAND / ESCALATE]**

Rationale: [Why this decision]

### If ESCALATE:
Reason: [Derived timeline significantly exceeds stakeholder expectations]
Options:
1. Re-evaluate milestone scope (reduce features)
2. Increase parallel workers
3. Negotiate timeline with stakeholders
```

### 11. Gap Discovery Workflow (if gaps found)
Follow `.github/prompts/includes/gap-discovery-workflow.md`.

### 12. Quality Gate Check
- [ ] All BLOCKING gaps identified (unmapped LLD components, circular dependencies)
- [ ] Circular dependencies are BLOCKING
- [ ] Missing LLD coverage is BLOCKING
- [ ] Decision is justified

### 13. Present Decision
- Show review summary and coverage matrix
- If APPROVED: "Ready to proceed to milestone evaluation"
- If APPROVED_WITH_ADJUSTMENTS: "Minor adjustments noted"
- If REMAND: "Plan issues must be addressed"
- If ESCALATE: "Scope/timeline negotiation required - present options to stakeholders"

---

## Out of Scope

- ❌ No milestone decomposition (evaluation phase)
- ❌ No ticket creation
- ❌ No automatic plan updates without user approval
- ❌ No stakeholder communication (user responsibility)

---

**Next Phase:** `evaluate-milestone.prompt.md` (Phase 5, Step 1) - if APPROVED
