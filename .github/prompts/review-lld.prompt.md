---
description: Review LLD for implementation readiness and developer clarity with gap-fix workflow (software-architect mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `software-architect` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

**Gap Discovery Workflow:**
Refer to `.github/prompts/includes/gap-discovery-workflow.md` for review workflow.

---

**Goal:** Validate LLD enables developers to implement without further clarification.

> Output: Review report with gap list and suggested fixes. Modify LLD only with user approval.

## Inputs

Required:
- Initiative ID: {{INIT_ID}}
- LLD Document: `docs/design/lld/{{INIT_ID}}-lld.md`
- HLD Document: `docs/design/hld/{{INIT_ID}}-hld.md`
- PRD Document: `docs/prd/{{INIT_ID}}-prd.md`

Optional:
- Code Standards: {{ORG_CODE_STANDARDS}}

---

## Execution Steps

### 1. Load Documents
- Read LLD, HLD, and PRD
- Extract specifications, schemas, API contracts
- Review code standards

### 2. Implementation Completeness Check
For each HLD component:
- [ ] Module/package structure defined
- [ ] Class/module responsibilities clear
- [ ] API endpoints fully specified
- [ ] Database schema complete
- [ ] Error handling defined
- [ ] Configuration specified
- [ ] Test strategy included

### 3. Database Schema Validation
- [ ] Normalization decision documented (3NF+ OR denormalized with justification)
- [ ] All columns have types and constraints
- [ ] Indexes defined for query patterns
- [ ] Foreign keys and relationships clear
- [ ] Migration strategy defined

### 4. API Contract Completeness
For each API endpoint:
- [ ] Request schema complete
- [ ] Response schema complete (success)
- [ ] All error responses defined
- [ ] Authentication/authorization clear
- [ ] Rate limiting specified
- [ ] Timeout strategy defined

### 5. Test Coverage Assessment
- [ ] Unit test strategy covers all modules
- [ ] Integration test scenarios address all AC
- [ ] Contract tests defined for APIs
- [ ] Performance tests if NFRs demand
- [ ] Test data strategy defined

### 6. Error Handling Verification
- [ ] All failure modes identified
- [ ] Retry logic specified
- [ ] Circuit breaker thresholds defined
- [ ] Error codes and messages complete

### 7. Configuration & Feature Flag Validation
- [ ] All env vars documented
- [ ] Feature flags identified
- [ ] Defaults specified
- [ ] Validation logic defined

### 8. Ambiguity Detection (Developer Perspective)
Simulate developer questions:
- "How do I implement X?" → Should be answerable from LLD
- "What happens when Y fails?" → Error handling should address
- "What format for Z?" → Schema/contract should specify

### 9. HLD Traceability
For each HLD component:
- [ ] Detailed in LLD
- [ ] All interactions specified
- [ ] Data flow clear

### 10. Create Review Report
**Output:** `docs/design/lld/{{INIT_ID}}-lld-review.md`

Structure:
```markdown
# LLD Review: {{INIT_ID}}

## Review Summary
- **Reviewer:** [Agent/Name]
- **Date:** {{DATE}}
- **LLD Version:** [Ref or date]
- **Decision:** [APPROVED / APPROVED_WITH_CLARIFICATIONS / REMAND]

## Implementation Completeness
| HLD Component | Module Structure | API Spec | DB Schema | Error Handling | Config | Tests | Status |
|---------------|------------------|----------|-----------|----------------|--------|-------|--------|
| Component A   | ✅               | ✅       | ✅        | ✅             | ✅     | ✅    | ✅     |
| Component B   | ✅               | ❌       | ✅        | ⚠️             | ✅     | ✅    | ❌     |

## Database Schema Validation
- [✅/❌] Normalization decision documented
- [✅/❌] All columns typed and constrained
- [✅/❌] Indexes defined for queries
- [✅/❌] Foreign keys clear
- [✅/❌] Migration strategy defined

## API Contract Completeness
| Endpoint | Request | Response | Errors | Auth | Rate Limit | Timeout | Status |
|----------|---------|----------|--------|------|------------|---------|--------|
| POST /x  | ✅      | ✅       | ✅     | ✅   | ✅         | ✅      | ✅     |
| GET /y   | ✅      | ✅       | ⚠️     | ✅   | ✅         | ✅      | ⚠️     |

## Test Coverage Assessment
- [✅/⚠️] Unit test strategy complete
- [✅/⚠️] Integration scenarios cover all AC
- [✅/⚠️] Contract tests defined
- [✅/⚠️] Performance tests (if needed)

## Error Handling Verification
- [✅/❌] All failure modes addressed
- [✅/❌] Retry logic specified
- [✅/❌] Circuit breaker defined

## Gap Analysis

### BLOCKING Gaps
1. **Gap:** [Description]
   - **Location:** [Component/Section]
   - **Severity:** BLOCKING
   - **Impact:** [What developers can't implement]
   - **Suggested Fix:** [Concrete specification]

### HIGH Priority Gaps
[Same structure]

### MEDIUM Priority Gaps
[Same structure]

## Ambiguity Check
[List any specifications requiring developer assumptions]

## HLD Traceability
[Confirm all HLD components detailed in LLD]

## Decision
**[APPROVED / APPROVED_WITH_CLARIFICATIONS / REMAND]**

Rationale: [Why this decision]
```

### 11. Gap Discovery Workflow (if gaps found)
Follow `.github/prompts/includes/gap-discovery-workflow.md`.

### 12. Quality Gate Check
- [ ] All BLOCKING gaps have suggested fixes
- [ ] Missing test cases flagged as HIGH
- [ ] Ambiguous API contracts flagged as BLOCKING
- [ ] Decision is justified

### 13. Present Decision
- Show review summary and completeness matrix
- If APPROVED: "Ready to proceed to implementation planning"
- If APPROVED_WITH_CLARIFICATIONS: "Minor clarifications documented for developers"
- If REMAND: "Specification gaps must be addressed before planning"

---

## Out of Scope

- ❌ No implementation code
- ❌ No implementation planning or milestone breakdown
- ❌ No automatic LLD updates without user approval
- ❌ No developer assignment or sprint planning

---

**Next Phase:** `generate-implementation-plan.prompt.md` (Phase 4, Step 1) - if APPROVED
