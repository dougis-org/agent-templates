---
description: Create tracking issues (GitHub/Jira) with hierarchical structure from milestone specifications (delivery-manager mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `delivery-manager` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

---

**Goal:** Create hierarchical tracking issues for milestone. This is an END STATE - no automatic execution chaining.

> Output: Tracking issues created in system. Requires user confirmation before creation.

## Inputs

Required:
- Initiative ID: {{INIT_ID}}
- Milestone ID: {{MILESTONE_ID}}
- Milestone Specification: (from `docs/plan/{{INIT_ID}}/implementation.md`)
- Milestone Evaluation: `docs/plan/{{INIT_ID}}/{{MILESTONE_ID}}-evaluation.md` (confirming DIRECT_TICKETS)
- LLD excerpt: (relevant sections for AC details)
- **Tracking system:** {{TRACKING_SYSTEM}} (GitHub or Jira)

---

## Execution Steps

### 1. Confirm Milestone Eligibility
- Verify evaluation report shows DIRECT_TICKETS recommendation
- If RECURSIVE_SDLC, abort: "This milestone requires recursive SDLC first"

### 2. Load Milestone Context
- Read milestone specification (objectives, components, AC)
- Read LLD excerpt for detailed AC and implementation notes
- Identify components to be implemented

### 3. Break Down Milestone into Work Items
Decompose milestone into granular, testable work items:
- API endpoint implementation (per endpoint or small groups)
- Database schema changes (per table or related groups)
- Service/business logic (per service or module)
- Integration implementations (per external system)
- Testing tasks (if substantial enough)

Each work item should be:
- 2-13 story points (split if larger)
- Independently testable
- Clear acceptance criteria from LLD

### 4. Map Dependencies Between Work Items
- Identify which work items depend on others
- Create blocked-by relationships
- Ensure DAG (no circular dependencies)
- Order by critical path

### 5. Select Tracking System Structure

**Option A: GitHub Issues**
- Create GitHub Milestone: `{{INIT_ID}}-{{MILESTONE_ID}}`
- Create GitHub Issues (tied to milestone)

**Option B: Jira Stories**
- Create Jira Epic: `{{INIT_ID}}-{{MILESTONE_ID}}`
- Create Jira Stories (as children under Epic)

### 6. Generate Tracking Issue Specifications

For each work item, create specification:

**Title:** `[{{INIT_ID}}-{{MILESTONE_ID}}-{{SEQ}}] {{TITLE}}`
- {{SEQ}}: Zero-padded sequential (01, 02, ...)
- {{TITLE}}: Short descriptive title (kebab-case friendly)

**Description:**
```markdown
## Context
Part of Milestone: {{MILESTONE_ID}} - [Objective]
Initiative: {{INIT_ID}}

## Objective
[What this work item delivers]

## Acceptance Criteria
- [ ] AC1 from LLD
- [ ] AC2 from LLD
- [ ] AC3 from LLD
- [ ] Tests pass
- [ ] Code review approved

## Implementation Notes
[Relevant LLD excerpts, API contracts, schema details]

## Dependencies
- Blocked by: #{{ISSUE_NUMBER}} (if applicable)

## Related Documentation
- LLD: `docs/design/lld/{{INIT_ID}}-lld.md` (Section X)
- Implementation Plan: `docs/plan/{{INIT_ID}}/implementation.md`
```

**Metadata:**
- Story Points: [1-13 SP]
- Priority: [Based on critical path and dependencies]
- Labels: [component, type (feature/bug/task/chore), milestone]
- Milestone/Epic: [Linked to parent]
- Dependencies: [Blocked-by links]

### 7. Create Ticket Manifest
**Output:** `docs/plan/{{INIT_ID}}/{{MILESTONE_ID}}-tickets.md`

Structure:
```markdown
# Ticket Manifest: {{INIT_ID}}-{{MILESTONE_ID}}

## Summary
- **Milestone:** {{MILESTONE_ID}}
- **Total Tickets:** [Count]
- **Total Story Points:** [Sum]
- **Tracking System:** [GitHub/Jira]
- **Milestone/Epic:** [Link to parent]
- **Created:** {{DATE}}

## Tracking Issues Created

### {{SYSTEM}} Milestone/Epic: [Link]

| Ticket ID | Title | Story Points | Dependencies | Status |
|-----------|-------|--------------|--------------|--------|
| [{{INIT_ID}}-{{MILESTONE_ID}}-01] | [Title] | 5 | None | Created |
| [{{INIT_ID}}-{{MILESTONE_ID}}-02] | [Title] | 8 | 01 | Created |
| [{{INIT_ID}}-{{MILESTONE_ID}}-03] | [Title] | 3 | 01 | Created |

## Dependency Graph
```
{{MILESTONE_ID}}-01 (no deps)
├─ {{MILESTONE_ID}}-02 (depends on 01)
└─ {{MILESTONE_ID}}-03 (depends on 01)
   └─ {{MILESTONE_ID}}-04 (depends on 03)
```

## Recommended Sprint Structure
Assuming 2-week sprints, {{WORKER_COUNT}} workers:

**Sprint 1:**
- Worker 1: {{MILESTONE_ID}}-01
- Worker 2: {{MILESTONE_ID}}-05 (parallel work)

**Sprint 2:**
- Worker 1: {{MILESTONE_ID}}-02 (unblocked after 01)
- Worker 2: {{MILESTONE_ID}}-03 (unblocked after 01)

[Continue for remaining sprints]

## Hierarchy Reference
- **Milestone/Epic:** [Link] → Represents {{MILESTONE_ID}}
- **Issues/Stories:** [Links] → Executable work items under milestone
```

### 8. Quality Gate Check
- [ ] All tickets follow naming convention
- [ ] Story points sum to milestone estimate (±10% acceptable)
- [ ] Dependencies explicitly linked
- [ ] AC are testable (no subjective "make it nice")
- [ ] Each ticket 2-13 SP (none larger)
- [ ] No duplicate work across tickets
- [ ] All AC traceable to LLD
- [ ] Tickets ordered by dependency (critical path first)

### 9. Present to User for Approval
Show ticket manifest with:
- Total ticket count and story points
- Dependency graph visualization
- Sample ticket descriptions (first 2-3)

Ask: "Create these {{COUNT}} tracking issues in {{SYSTEM}}?"
- If YES: Proceed to create in tracking system
- If NO/ADJUST: Modify specifications and re-present

### 10. Create Tracking Issues
Use appropriate MCP tools:
- **GitHub:** `gh-issues/issue_write`, `gh-projects/*`
- **Jira:** Use Jira API tools (if available) or provide manual creation script

For each tracking issue:
1. Create parent Milestone/Epic (if not exists)
2. Create Issue/Story with full specification
3. Link to parent Milestone/Epic
4. Add blocked-by relationships
5. Add labels and metadata

### 11. Confirm Creation
- Show links to created tracking issues
- Confirm manifest file created
- Mark this milestone as "Tickets Generated" in tracking

---

## Out of Scope

- ❌ **NO automatic chaining to ticket execution** (This is END STATE)
- ❌ No ticket assignment to developers
- ❌ No sprint planning or velocity tracking
- ❌ No status updates or workflow transitions
- ❌ No modification of implementation plan or LLD

---

## Next Steps (Manual - User Responsibility)

1. **Team Review:** Review generated tickets with team
2. **Assignment:** Assign tickets to developers
3. **Sprint Planning:** Organize tickets into sprints
4. **Execution:** Use TICKET_FLOW.md process for individual ticket execution

**TICKET_FLOW.md Integration:**
- `find-next-ticket` will scan these created issues
- Individual tickets executed via `plan-ticket` → `work-ticket` → `cut-pr` → `review-pr`
- This is a SEPARATE SUBPROCESS, not automatically initiated

---

**END STATE REACHED:** Ticket generation complete. Manual handoff to ticket execution process.
