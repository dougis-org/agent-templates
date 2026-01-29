---
description: Review HLD for architectural soundness and PRD alignment with gap-fix workflow (software-architect mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `software-architect` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

**Gap Discovery Workflow:**
Refer to `.github/prompts/includes/gap-discovery-workflow.md` for review workflow.

---

**Goal:** Validate architecture soundness, standards alignment, and PRD traceability.

> Output: Review report with gap list and suggested fixes. Modify HLD only with user approval.

## Inputs

Required:
- Initiative ID: {{INIT_ID}}
- HLD Document: `docs/design/hld/{{INIT_ID}}-hld.md`
- PRD Document: `docs/prd/{{INIT_ID}}-prd.md`

Optional:
- Architecture Standards: {{ORG_ARCH_STANDARDS}}

---

## Execution Steps

### 1. Load Documents
- Read HLD and PRD
- Extract architecture decisions, components, technology choices
- Load organizational architecture standards

### 2. Architecture Consistency Check
Validate against org standards:
- [ ] Follows organizational architecture patterns
- [ ] Technology choices approved/standard
- [ ] Naming conventions followed
- [ ] Security patterns aligned with org policies
- [ ] Deployment approach follows org guidelines

### 3. PRD Traceability Matrix
For each MUST requirement in PRD:
- Map to component(s) in HLD
- Flag unmapped requirements as BLOCKING gaps

For each component in HLD:
- Trace back to PRD requirement(s)
- Flag orphan components (not driven by requirements)

### 4. Technology Decision Validation
For each technology choice:
- [ ] Trade-offs documented (options considered)
- [ ] Decision rationale present
- [ ] Alignment with org standards validated
- [ ] Risk assessment included

### 5. Integration Feasibility Assessment
For each external integration:
- [ ] Integration contract clear
- [ ] Error handling strategy defined
- [ ] Compatibility with external system verified
- [ ] Fallback/degradation approach defined

### 6. Security & Compliance Coverage
- [ ] Authentication strategy addresses all user types
- [ ] Authorization model covers all data access patterns
- [ ] Data protection meets compliance requirements
- [ ] Security controls sufficient for threat model

### 7. Scalability Feasibility Check
- [ ] Architecture supports PRD NFRs (latency, throughput)
- [ ] Bottlenecks identified and mitigated
- [ ] Scaling strategy defined (horizontal/vertical)
- [ ] Cost implications considered

### 8. Create Review Report
**Output:** `docs/design/hld/{{INIT_ID}}-hld-review.md`

Structure:
```markdown
# HLD Review: {{INIT_ID}}

## Review Summary
- **Reviewer:** [Agent/Name]
- **Date:** {{DATE}}
- **HLD Version:** [Ref or date]
- **Decision:** [APPROVED / APPROVED_WITH_CONCERNS / REMAND / ESCALATE]

## Architecture Consistency
- [✅/❌] Follows org patterns
- [✅/❌] Technology choices approved
- [✅/❌] Naming conventions followed
- [✅/❌] Security patterns aligned

## PRD Traceability Matrix
| PRD Requirement | HLD Component(s) | Status |
|-----------------|------------------|--------|
| REQ-001         | Component A, B   | ✅     |
| REQ-002         | UNMAPPED         | ❌     |

**Unmapped Requirements:** [List with severity BLOCKING]

## Technology Decision Validation
[Assessment of each major technology choice]

## Integration Feasibility
[Assessment of external integrations]

## Security & Compliance Coverage
- [✅/⚠️] Authentication adequate
- [✅/⚠️] Authorization adequate
- [✅/⚠️] Data protection adequate
- [✅/⚠️] Compliance requirements met

## Scalability Feasibility
[Can NFRs be achieved with this architecture?]

## Gap Analysis

### BLOCKING Gaps
1. **Gap:** [Description]
   - **Location:** [Section/Component]
   - **Severity:** BLOCKING
   - **Suggested Fix:** [Concrete design change]

### HIGH Priority Gaps
[Same structure]

### MEDIUM Priority Gaps
[Same structure]

## Risks & Mitigations
[Identified architectural risks and proposed mitigations]

## Decision
**[APPROVED / APPROVED_WITH_CONCERNS / REMAND / ESCALATE]**

Rationale: [Why this decision]
```

### 9. Gap Discovery Workflow (if gaps found)
Follow `.github/prompts/includes/gap-discovery-workflow.md`.

### 10. Quality Gate Check
- [ ] All BLOCKING gaps identified (especially unmapped PRD requirements)
- [ ] Security concerns flagged as BLOCKING
- [ ] Technology decisions validated against org standards
- [ ] Scalability assessment complete
- [ ] Decision is justified

### 11. Present Decision
- Show review summary and traceability matrix
- If APPROVED: "Ready to proceed to LLD"
- If APPROVED_WITH_CONCERNS: "Proceed with documented mitigations"
- If REMAND: "Fixes needed before LLD"
- If ESCALATE: "Architecture review board approval required"

---

## Out of Scope

- ❌ No detailed API specifications (LLD phase)
- ❌ No implementation code
- ❌ No automatic HLD updates without user approval
- ❌ No architecture board coordination (user responsibility)

---

**Next Phase:** `generate-lld.prompt.md` (Phase 3, Step 1) - if APPROVED
