---
description: Scan and identify new initiatives from backlog/roadmap for SDLC entry (product-manager mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `product-manager` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

---

**Goal:** Identify and scope a new strategic initiative for SDLC processing.

> Output: Initiative summary document. No side effects; read-only analysis.

## Inputs

Required:
- Initiative request source: {{BACKLOG_REFERENCE}} (roadmap doc, stakeholder request, strategic goal)

Optional:
- Stakeholder contacts: {{STAKEHOLDER_LIST}}
- Timeline constraint: {{TIMELINE_CONSTRAINT}}

---

## Execution Steps

### 1. Scan Initiative Source
- Review backlog/roadmap for new initiatives
- Identify initiatives marked "ready for planning" or equivalent
- Prioritize by strategic value, urgency, stakeholder requests

### 2. Validate Business Case
- Ensure clear problem statement exists
- Identify measurable value/outcome (revenue, cost reduction, user satisfaction)
- Confirm initiative is not duplicate of existing work

### 3. Gather Initiative Context
- Initiative name and identifier (create if needed: `INIT-YYYY-###`)
- Key stakeholders (with roles: sponsor, product owner, technical lead)
- Estimated scope (T-shirt: S/M/L/XL based on complexity)
- Timeline constraints (deadlines, dependencies on other initiatives)
- Team availability (engineers, designers, QA available)

### 4. Create Initiative Summary
**Output:** `docs/initiatives/{{INIT_ID}}-summary.md`

Structure:
```markdown
# Initiative Summary: {{INIT_ID}}

## Overview
- **ID:** {{INIT_ID}}
- **Name:** {{INITIATIVE_NAME}}
- **Status:** Discovery
- **Created:** {{DATE}}

## Business Case
- **Problem Statement:** [What problem are we solving?]
- **Expected Value:** [Revenue, cost reduction, user impact]
- **Success Criteria:** [How will we know we succeeded?]

## Stakeholders
- **Sponsor:** [Name, role, contact]
- **Product Owner:** [Name, role, contact]
- **Technical Lead:** [Name, role, contact]
- **Other Stakeholders:** [List with roles]

## Scope Estimate
- **Size:** [S/M/L/XL]
- **Rationale:** [Why this size estimate?]
- **Components Impacted:** [High-level list]

## Timeline
- **Target Start:** [Date or Q#]
- **Target Completion:** [Date or Q#]
- **Constraints:** [Dependencies, deadlines]

## Team
- **Available Engineers:** [Count and skill areas]
- **Available Designers:** [Count]
- **Available QA:** [Count]
- **Other Resources:** [Data, platform, etc.]

## Next Steps
- [ ] Stakeholder alignment meeting scheduled
- [ ] PRD generation (Phase 1, Step 2)
```

### 5. Quality Gate Check
- [ ] Business case is clear and measurable
- [ ] Stakeholders identified with ownership
- [ ] Scope estimate is justified (not "M" without rationale)
- [ ] No duplicate initiative exists
- [ ] Team availability confirmed

### 6. Present to User
- Show initiative summary
- Confirm: "Ready to proceed to PRD generation?" or flag blockers

---

## Out of Scope

- ❌ No PRD creation (next phase)
- ❌ No technical architecture decisions
- ❌ No detailed requirements gathering
- ❌ No timeline commitments (estimates only)
- ❌ No automatic progression to next phase

---

**Next Phase:** `generate-prd.prompt.md` (Phase 1, Step 2)
