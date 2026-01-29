---
description: Review PRD for completeness, clarity, and stakeholder alignment with gap-fix workflow (product-manager mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `product-manager` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

**Gap Discovery Workflow:**
Refer to `.github/prompts/includes/gap-discovery-workflow.md` for review workflow.

---

**Goal:** Validate PRD completeness and identify gaps before architecture phase.

> Output: Review report with gap list and suggested fixes. Modify PRD only with user approval.

## Inputs

Required:
- Initiative ID: {{INIT_ID}}
- PRD Document: `docs/prd/{{INIT_ID}}-prd.md`

Optional:
- Initiative Summary: `docs/initiatives/{{INIT_ID}}-summary.md`

---

## Execution Steps

### 1. Load PRD
- Read `docs/prd/{{INIT_ID}}-prd.md`
- Extract all sections, requirements, personas, metrics

### 2. Completeness Assessment
Check all required sections present:
- [ ] Executive Summary
- [ ] Business Objectives & Success Metrics
- [ ] User Personas (with journey maps)
- [ ] Functional Requirements (MoSCoW prioritized)
- [ ] Non-Functional Requirements
- [ ] Constraints & Assumptions
- [ ] Out-of-Scope (explicit)
- [ ] Open Questions

### 3. Clarity Review
Scan for ambiguous language:
- Vague terms: "intuitive", "user-friendly", "seamless", "easy"
- Unmeasurable goals: "improve experience", "faster", "better"
- Undefined terms without context

### 4. Gap Analysis
- **Missing Personas:** Are all user types covered?
- **Missing Requirements:** Do journeys reveal unaddressed needs?
- **Missing Constraints:** Technical, regulatory, business limits?
- **Unmeasurable Metrics:** Are success criteria SMART?
- **Conflicting Requirements:** Do any requirements contradict?

### 5. Stakeholder Alignment Check
- Do requirements cover all stakeholder needs from initiative summary?
- Are open questions assigned to appropriate owners?
- Are assumptions flagged for validation?

### 6. Create Review Report
**Output:** `docs/prd/{{INIT_ID}}-prd-review.md`

Structure:
```markdown
# PRD Review: {{INIT_ID}}

## Review Summary
- **Reviewer:** [Agent/Name]
- **Date:** {{DATE}}
- **PRD Version:** [Ref or date]
- **Decision:** [APPROVED / APPROVED_WITH_ASSUMPTIONS / REMAND]

## Completeness Assessment
- [✅/❌] Executive Summary
- [✅/❌] Business Objectives & Metrics
- [✅/❌] User Personas & Journeys
- [✅/❌] Functional Requirements (MoSCoW)
- [✅/❌] Non-Functional Requirements
- [✅/❌] Constraints & Assumptions
- [✅/❌] Out-of-Scope
- [✅/❌] Open Questions

## Clarity Issues
[List any ambiguous language with line references]

## Gap Analysis

### BLOCKING Gaps
1. **Gap:** [Description]
   - **Location:** [Section/Line]
   - **Severity:** BLOCKING
   - **Suggested Fix:** [Concrete addition/change]

### HIGH Priority Gaps
[Same structure]

### MEDIUM Priority Gaps
[Same structure]

## Stakeholder Alignment
- [✅/⚠️] All stakeholder needs addressed
- [✅/⚠️] Open questions have owners
- [✅/⚠️] Assumptions flagged for validation

## Recommendations
[Summary of needed changes]

## Decision
**[APPROVED / APPROVED_WITH_ASSUMPTIONS / REMAND]**

Rationale: [Why this decision]
```

### 7. Gap Discovery Workflow (if gaps found)
Follow `.github/prompts/includes/gap-discovery-workflow.md`.

### 8. Quality Gate Check
- [ ] All BLOCKING gaps have suggested fixes
- [ ] Severity assigned to all issues (BLOCKING/HIGH/MEDIUM/LOW)
- [ ] Recommendations are actionable
- [ ] Decision is justified

### 9. Present Decision
- Show review summary
- If APPROVED: "Ready to proceed to HLD"
- If APPROVED_WITH_ASSUMPTIONS: "Proceed with documented caveats"
- If REMAND: "Fixes needed before architecture phase"

---

## Out of Scope

- ❌ No technical architecture or system design
- ❌ No implementation details
- ❌ No automatic PRD updates without user approval
- ❌ No stakeholder communication (user responsibility)

---

**Next Phase:** `generate-hld.prompt.md` (Phase 2, Step 1) - if APPROVED
