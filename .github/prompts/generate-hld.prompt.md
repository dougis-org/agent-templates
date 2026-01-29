---
description: Create High-Level Design with architecture decisions and trade-off rationale (software-architect mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `software-architect` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

---

**Goal:** Design system architecture aligned with PRD and organizational standards.

> Output: HLD document with architecture diagrams, component specs, and technology decisions.

## Inputs

Required:
- Initiative ID: {{INIT_ID}}
- PRD Document: `docs/prd/{{INIT_ID}}-prd.md`

Optional:
- PRD Review: `docs/prd/{{INIT_ID}}-prd-review.md`
- Architecture Guidelines: {{ORG_ARCH_GUIDELINES}}
- Existing System Docs: {{EXISTING_SYSTEM_REFS}}

---

## Execution Steps

### 1. Load Requirements
- Read approved PRD
- Extract functional/non-functional requirements
- Note constraints and assumptions

### 2. Search Existing Patterns
- Use deepcontext/search_code to find established architecture patterns
- Identify reusable components in codebase
- Review organizational architecture standards
- Cite existing patterns where applicable

### 3. Design System Architecture
Major decisions:
- System decomposition (monolith, microservices, etc.)
- Component boundaries and responsibilities
- Communication patterns (sync/async, protocols)
- Data flow and state management
- Integration points with external systems

### 4. Select Technology Stack
For each technology choice, document:
- Options considered
- Trade-offs (pros/cons of each)
- Decision rationale (why chosen option best fits)
- Alignment with org standards

### 5. Design Security & Compliance
- Authentication strategy (OAuth, JWT, etc.)
- Authorization model (RBAC, ABAC, etc.)
- Data protection (encryption at rest/in transit)
- Compliance approach (GDPR, HIPAA, etc.)
- Security controls and monitoring

### 6. Create HLD Document
**Output:** `docs/design/hld/{{INIT_ID}}-hld.md`

Structure:
```markdown
# High-Level Design: {{INIT_ID}}

## Overview
[2-3 paragraphs: System purpose, key components, architecture style]

## System Architecture Diagram
[Boxes and arrows showing major components and interactions]

## Major Components
### Component 1: [Name]
- **Responsibility:** [What it does]
- **Technology:** [Tech stack]
- **Interfaces:** [APIs exposed]
- **Dependencies:** [Other components it calls]

[Repeat for each component]

## Component Interactions
[Sequence diagrams or flow descriptions for key scenarios]

## Technology Stack Decisions
### Decision 1: [Technology Choice]
- **Options Considered:** [A, B, C]
- **Pros/Cons:**
  - A: [Pros/Cons]
  - B: [Pros/Cons]
  - C: [Pros/Cons]
- **Decision:** [Chosen option]
- **Rationale:** [Why this option best]

[Repeat for each major technology decision]

## Conceptual Data Model
[Entity-relationship diagram or description]
- Key entities and relationships
- Data ownership (which component owns what)
- Data consistency strategy

## Security & Compliance
- **Authentication:** [Strategy]
- **Authorization:** [Model]
- **Data Protection:** [Encryption, masking]
- **Compliance:** [How requirements met]
- **Security Controls:** [Firewalls, WAF, etc.]

## Integration Points
### Integration 1: [External System]
- **Purpose:** [Why integrate]
- **Protocol:** [REST, gRPC, messaging]
- **Contract:** [High-level API]
- **Error Handling:** [Retry, fallback]

[Repeat for each external system]

## High-Level API Contracts
[Major API endpoints or message schemas - details in LLD]

## Deployment Architecture
- Hosting platform (cloud, on-prem)
- Scaling strategy (horizontal, vertical)
- Availability zones/regions
- Load balancing approach

## Scalability Approach
- How NFR targets will be met
- Bottleneck identification and mitigation
- Growth projections and capacity planning

## Risks & Mitigations
- [Risk 1] - Mitigation: [How addressed]
- [Risk 2] - Mitigation: [How addressed]
```

### 7. Quality Gate Check
- [ ] Architecture trade-offs documented for major decisions
- [ ] No single points of failure (or explicitly accepted as risk)
- [ ] All PRD functional requirements traceable to components
- [ ] Technology decisions justified and aligned with org standards
- [ ] Integration points with existing systems clear
- [ ] Security approach covers data, auth, encryption
- [ ] Scalability targets align with PRD NFRs

### 8. Present to User
- Show HLD summary and architecture diagram
- Highlight key technology decisions
- Ask: "Ready for HLD review phase?"

---

## Out of Scope

- ❌ No detailed API specifications (LLD phase)
- ❌ No database schema details (LLD phase)
- ❌ No implementation code or file structure
- ❌ No implementation planning or effort estimation

---

**Next Phase:** `review-hld.prompt.md` (Phase 2, Step 2)
