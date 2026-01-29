---
description: Assess milestone complexity and recommend direct tickets or recursive SDLC (delivery-manager mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `delivery-manager` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

---

**Goal:** Determine if milestone is ready for direct ticket creation or needs recursive mini-SDLC.

> Output: Evaluation report with complexity score and recommendation. NO automatic sub-initiative creation.

## Inputs

Required:
- Initiative ID: {{INIT_ID}}
- Milestone ID: {{MILESTONE_ID}}
- Milestone Specification: (from `docs/plan/{{INIT_ID}}/implementation.md`)
- LLD excerpt: (relevant sections for this milestone)

Optional:
- Team familiarity assessment: {{TEAM_FAMILIARITY}}

---

## Execution Steps

### 1. Load Milestone Context
- Read milestone specification from implementation plan
- Extract components, APIs, database tables, integrations
- Note team familiarity and tech stack novelty

### 2. Score Complexity Factors
Apply objective scoring (Low=1, Medium=2, High=3):

| Factor | Low (1pt) | Medium (2pt) | High (3pt) | Score |
|--------|-----------|--------------|------------|-------|
| **Components** | 1-2 | 3-5 | 6+ | [X] |
| **APIs** | <5 | 5-15 | 15+ | [X] |
| **DB Tables** | <10 | 10-30 | 30+ | [X] |
| **Integration Points** | 0 | 1-2 | 3+ | [X] |
| **Data Processing** | Simple CRUD | Workflow logic | Complex algorithms | [X] |
| **Team Familiarity** | Familiar | Somewhat | Unfamiliar | [X] |
| **Tech Stack Novelty** | Proven | Mostly proven | New/experimental | [X] |

**Total Score:** [Sum] / 21

### 3. Apply Thresholds
- **≤10 points:** Low complexity → DIRECT_TICKETS
- **11-15 points:** Medium complexity → Evaluate (consider risk factors)
- **≥16 points:** High complexity → RECURSIVE_SDLC

### 4. Consider Risk Factors (for medium complexity)
If score is 11-15, evaluate:
- External dependencies (risky integrations)
- Team skill gaps (learning curve)
- Novel tech (unproven in org)
- Critical path (delays impact entire initiative)

Lean toward RECURSIVE_SDLC if high risk factors.

### 5. Formulate Recommendation
**DIRECT_TICKETS:**
- Milestone well-understood
- Team capable of implementing directly
- Estimate ticket count (typically 5-12 tickets for milestone)

**RECURSIVE_SDLC:**
- Milestone too complex for direct tickets
- Needs mini-PRD, mini-HLD, mini-LLD to clarify scope
- Define sub-initiative ID: `{{INIT_ID}}-{{MILESTONE_ID}}`
- Outline reduced scope for mini-SDLC

### 6. Create Evaluation Report
**Output:** `docs/plan/{{INIT_ID}}/{{MILESTONE_ID}}-evaluation.md`

Structure:
```markdown
# Milestone Evaluation: {{INIT_ID}}-{{MILESTONE_ID}}

## Overview
- **Milestone:** {{MILESTONE_ID}}
- **Objectives:** [From implementation plan]
- **Estimated Effort:** [Story points]
- **Evaluation Date:** {{DATE}}

## Complexity Scoring
| Factor | Score | Rationale |
|--------|-------|-----------|
| Components | [X/3] | [Count and reasoning] |
| APIs | [X/3] | [Count and reasoning] |
| DB Tables | [X/3] | [Count and reasoning] |
| Integration Points | [X/3] | [Count and reasoning] |
| Data Processing | [X/3] | [CRUD/Workflow/Complex] |
| Team Familiarity | [X/3] | [Familiar/Somewhat/Unfamiliar] |
| Tech Stack Novelty | [X/3] | [Proven/Mostly/New] |

**Total Score:** [Sum] / 21 → [LOW / MEDIUM / HIGH] Complexity

## Risk Factors (if medium complexity)
- [Risk factor 1]: [Impact]
- [Risk factor 2]: [Impact]

## Recommendation

### [DIRECT_TICKETS / RECURSIVE_SDLC]

**Rationale:**
[Why this recommendation based on complexity score and risk factors]

---

## If DIRECT_TICKETS:

**Estimated Ticket Count:** [5-12 tickets]

**Ready for Phase 6:** `generate-tickets.prompt.md`

---

## If RECURSIVE_SDLC:

**Sub-Initiative ID:** `{{INIT_ID}}-{{MILESTONE_ID}}`

**Reduced Scope for Mini-SDLC:**
- **Focus:** [What this milestone addresses]
- **Boundaries:** [What's included/excluded]
- **Entry Point:** Phase 1 (PRD generation) with reduced scope
- **Expected Output:** Tickets for sub-milestones after full mini-SDLC

**Why Recursive SDLC Needed:**
- [Reason 1: e.g., Custom workflow engine design unclear]
- [Reason 2: e.g., Team unfamiliar with graph databases]
- [Reason 3: e.g., 20+ APIs need detailed specification]

**Mini-SDLC Phases:**
1. PRD: Scoped to {{MILESTONE_ID}} features only
2. HLD: Architecture for {{MILESTONE_ID}} components
3. LLD: Detailed specs for {{MILESTONE_ID}}
4. Implementation Plan: Break {{MILESTONE_ID}} into sub-milestones
5. Milestone Evaluation: Evaluate sub-milestones (may recurse further)
6. Ticket Generation: Create tickets for leaf-level work

---

## Next Steps
- [ ] User approval required for RECURSIVE_SDLC recommendation
- [ ] If approved, create sub-initiative and re-enter Phase 1
- [ ] If rejected, proceed with DIRECT_TICKETS (user accepts complexity risk)
```

### 7. Quality Gate Check
- [ ] All 7 factors scored (no skipped factors)
- [ ] Recommendation justified with specific factors
- [ ] If RECURSIVE_SDLC, sub-initiative scope clearly defined
- [ ] If RECURSIVE_SDLC, rationale explains why decomposition needed

### 8. Present to User
- Show complexity score and breakdown
- Present recommendation with rationale
- If RECURSIVE_SDLC: "This milestone needs mini-SDLC due to [reasons]. Create sub-initiative `{{INIT_ID}}-{{MILESTONE_ID}}`?"
- If DIRECT_TICKETS: "Ready to generate tickets for this milestone?"
- **Wait for explicit user approval before any action**

---

## Out of Scope

- ❌ No automatic sub-initiative creation
- ❌ No ticket generation (separate phase)
- ❌ No mini-SDLC execution (requires user-initiated re-entry to Phase 1)
- ❌ No modification of parent implementation plan

---

**Next Phase (if DIRECT_TICKETS):** `generate-tickets.prompt.md` (Phase 6)

**Next Phase (if RECURSIVE_SDLC approved):** Re-enter `generate-prd.prompt.md` (Phase 1) with sub-initiative scope
