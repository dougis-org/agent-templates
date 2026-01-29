---
description: Create comprehensive Product Requirements Document from initiative summary (product-manager mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `product-manager` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

---

**Goal:** Create customer-focused PRD with SMART metrics, MoSCoW-prioritized requirements, and complete stakeholder coverage.

> Output: PRD document. Ask clarifying questions if requirements ambiguous.

## Inputs

Required:
- Initiative ID: {{INIT_ID}}
- Initiative Summary: `docs/initiatives/{{INIT_ID}}-summary.md`

Optional:
- Stakeholder interview notes: {{INTERVIEW_NOTES}}
- Market research: {{MARKET_RESEARCH_LINKS}}
- Competitor analysis: {{COMPETITOR_DOCS}}

---

## Execution Steps

### 1. Load Initiative Context
- Read `docs/initiatives/{{INIT_ID}}-summary.md`
- Extract business case, stakeholders, scope estimate
- Identify open questions from summary

### 2. Gather Requirements
- Review stakeholder feedback/interviews
- Analyze market research for user needs
- Study competitor solutions for feature gaps
- Identify user personas (roles, goals, pain points)

### 3. Create PRD Document
**Output:** `docs/prd/{{INIT_ID}}-prd.md`

Structure:
```markdown
# Product Requirements Document: {{INIT_ID}}

## Executive Summary
[2-3 paragraphs: What, Why, Who, When]

## Business Objectives
- **Primary Goal:** [What business metric improves?]
- **Secondary Goals:** [Other benefits]
- **Success Metrics (SMART):**
  - Metric 1: [Specific, Measurable, Achievable, Relevant, Time-bound]
  - Metric 2: [...]

## User Personas
### Persona 1: [Role Name]
- **Description:** [Who they are, context]
- **Goals:** [What they want to achieve]
- **Pain Points:** [Current problems]
- **Journey Map:** [Key steps they take]

[Repeat for each persona]

## Functional Requirements

### MUST (Critical for launch)
- REQ-001: [User can/system shall] [action] [context]
- REQ-002: [...]

### SHOULD (Important but not blocking)
- REQ-010: [...]

### COULD (Nice to have)
- REQ-020: [...]

### WON'T (Explicitly out of scope)
- [Feature X] - Rationale: [Why not now]

## Non-Functional Requirements
- **Performance:** [P99 latency <X ms, throughput Y req/s]
- **Availability:** [99.9% uptime]
- **Security:** [Authentication, authorization, encryption requirements]
- **Compliance:** [GDPR, HIPAA, etc.]
- **Scalability:** [User/data growth targets]

## Constraints & Assumptions
### Constraints
- [Technical, regulatory, business constraints]

### Assumptions
- [What we're assuming true; needs validation]

## Out of Scope
- [Feature A] - Rationale: [...]
- [Feature B] - Rationale: [...]

## Open Questions
- [ ] Question 1 - Owner: [Name], Due: [Date]
- [ ] Question 2 - Owner: [Name], Due: [Date]

## Appendices
- Journey maps (detailed)
- Research references
- Stakeholder interview summaries
```

### 4. Quality Gate Check
- [ ] All personas documented with journeys
- [ ] Success metrics are SMART (no vague "improve experience")
- [ ] Functional requirements use MoSCoW prioritization
- [ ] NFRs have quantified targets (latency, throughput, etc.)
- [ ] No ambiguous language ("intuitive", "user-friendly", "seamless")
- [ ] Out-of-scope items explicitly listed with rationale
- [ ] Open questions have owners and target dates

### 5. Present to User
- Show PRD summary
- Highlight any assumptions requiring stakeholder validation
- Ask: "Ready for PRD review phase?" or flag gaps

---

## Out of Scope

- ❌ No technical architecture or system design
- ❌ No implementation planning or effort estimation
- ❌ No API specifications or database schema
- ❌ No automatic progression to review phase

---

**Next Phase:** `review-prd.prompt.md` (Phase 1, Step 3)
