# SDLC Agents & Prompts Mapping

**Version:** 1.0  
**Last Updated:** January 28, 2026  
**Source:** [SDLC_INITIATIVE_PLANNING.md](../SDLC_INITIATIVE_PLANNING.md)  
**Pattern Reference:** [TICKET_FLOW.md](../TICKET_FLOW.md)

---

## Overview

This document maps the SDLC Initiative Planning phases to agents and prompts following the established pattern from TICKET_FLOW.md. Agents are consolidated by **persona alignment** where roles share similar focus, tooling needs, and decision-making perspectives.

**Key Principles:**
- 🎭 **Persona-Based Agents:** Agents represent distinct professional roles with specific expertise
- 📋 **Prompt-Per-Step:** Each SDLC step has a dedicated prompt defining execution steps and guardrails
- 🔧 **Tool Scoping:** Agents define available tools; prompts define how to use them
- 🔄 **Recursive Reuse:** Same agents/prompts handle recursive mini-SDLCs on reduced scope

---

## Table of Contents

1. [SDLC Step to Agent/Prompt Mapping](#sdlc-step-to-agentprompt-mapping)
2. [Consolidated Agent Definitions](#consolidated-agent-definitions)
3. [Prompt Specifications](#prompt-specifications)
4. [Recursive SDLC Handling](#recursive-sdlc-handling)
5. [Integration with Ticket Flow](#integration-with-ticket-flow)

---

## SDLC Step to Agent/Prompt Mapping

| Phase | Step | SDLC Step Name | Agent Mode | Prompt | Purpose |
|-------|------|----------------|------------|--------|---------|
| 1 | 1.1 | Initiative Discovery | `product-manager` | `discover-initiative.prompt.md` | Identify and scope new initiatives |
| 1 | 1.2 | PRD Generation | `product-manager` | `generate-prd.prompt.md` | Create Product Requirements Document |
| 1 | 1.3 | PRD Review | `product-manager` | `review-prd.prompt.md` | Review PRD for completeness & gaps |
| 2 | 2.1 | HLD Generation | `software-architect` | `generate-hld.prompt.md` | Generate High-Level Design artifact |
| 2 | 2.2 | HLD Review | `software-architect` | `review-hld.prompt.md` | Review HLD for architectural soundness |
| 3 | 3.1 | LLD Generation | `software-architect` | `generate-lld.prompt.md` | Generate Low-Level Design specifications |
| 3 | 3.2 | LLD Review | `software-architect` | `review-lld.prompt.md` | Review LLD for implementation readiness |
| 4 | 4.1 | Implementation Plan | `delivery-manager` | `generate-implementation-plan.prompt.md` | Break LLD into milestones & deliverables |
| 4 | 4.2 | Implementation Plan Review | `delivery-manager` | `review-implementation-plan.prompt.md` | Review plan for feasibility |
| 5 | 5.1 | Milestone Evaluation | `delivery-manager` | `evaluate-milestone.prompt.md` | Assess if milestone needs recursive SDLC |
| 6 | 6.1 | Ticket Generation | `delivery-manager` | `generate-tickets.prompt.md` | Create tracking issues from milestone specs |

### Phase Flow Summary

```text
Phase 1: Requirements & Vision
  └─ product-manager agent
     ├─ discover-initiative.prompt.md
     ├─ generate-prd.prompt.md
     └─ review-prd.prompt.md

Phase 2: High-Level Design
  └─ software-architect agent
     ├─ generate-hld.prompt.md
     └─ review-hld.prompt.md

Phase 3: Low-Level Design
  └─ software-architect agent
     ├─ generate-lld.prompt.md
     └─ review-lld.prompt.md

Phase 4: Implementation Planning
  └─ delivery-manager agent
     ├─ generate-implementation-plan.prompt.md
     └─ review-implementation-plan.prompt.md

Phase 5: Milestone Evaluation
  └─ delivery-manager agent
     └─ evaluate-milestone.prompt.md

Phase 6: Tracking Issue Creation (End State)
  └─ delivery-manager agent
     └─ generate-tickets.prompt.md
```

---

## Consolidated Agent Definitions

### Agent 1: `product-manager.agent.md`

**Persona:** Product Manager with focus on customer needs, problem statements, and solution design.

**Expertise Areas:**
- Customer journey mapping and persona development
- Business case articulation and ROI analysis
- Requirements elicitation and prioritization (MoSCoW)
- Stakeholder alignment and communication
- Market research and competitive analysis
- Success metrics definition (SMART criteria)

**Tool Access:**

| Category | Tools | Purpose |
|----------|-------|---------|
| Repository | `read/*`, `search`, `edit/createFile`, `edit/editFiles` | Read context, create/update PRD documents |
| GitHub | `gh-issues/read`, `gh-projects/read` | Read issue context, project status |
| Web | `web/fetch` | Market research, competitor analysis |
| Analysis | `sequentialthinking/*` | Complex decision reasoning |
| Documentation | `markdownlint/*` | Ensure document quality |

**Write Permissions:**
- `docs/prd/**` - PRD documents and reviews
- `docs/initiatives/**` - Initiative summaries

**Behavioral Focus:**
- Customer-centric language and perspective
- Clear problem-solution articulation
- Measurable success criteria
- Stakeholder needs coverage
- Ambiguity elimination in requirements

**Prompts Using This Agent:**
- `discover-initiative.prompt.md`
- `generate-prd.prompt.md`
- `review-prd.prompt.md`

---

### Agent 2: `software-architect.agent.md`

**Persona:** Software Architect with focus on system design, best practices, and maintainability.

**Expertise Areas:**
- System architecture patterns (microservices, event-driven, etc.)
- Technology stack evaluation and trade-off analysis
- Integration design and API contracts
- Security and compliance architecture
- Scalability and performance design
- Database modeling and data flow
- Existing codebase patterns and conventions

**Tool Access:**

| Category | Tools | Purpose |
|----------|-------|---------|
| Repository | `read/*`, `search`, `edit/createFile`, `edit/editFiles`, `deepcontext/*` | Read codebase, create/update design docs |
| GitHub | `gh-issues/read`, `github/search_code` | Understand requirements, find existing patterns |
| Analysis | `sequentialthinking/*` | Architecture decision reasoning |
| Documentation | `markdownlint/*` | Ensure document quality |
| Context | `upstash/context7/*` | Library/framework documentation |

**Write Permissions:**
- `docs/design/hld/**` - High-level design documents
- `docs/design/lld/**` - Low-level design documents

**Behavioral Focus:**
- Architecture defensibility with documented trade-offs
- Alignment with existing project patterns and conventions
- Non-functional requirements feasibility
- Security-first design thinking
- Maintainability and extensibility considerations
- Clear component boundaries and responsibilities

**Prompts Using This Agent:**
- `generate-hld.prompt.md`
- `review-hld.prompt.md`
- `generate-lld.prompt.md`
- `review-lld.prompt.md`

---

### Agent 3: `delivery-manager.agent.md`

**Persona:** Scrum/Delivery Manager with focus on execution planning, scheduling, and clear communication.

**Expertise Areas:**
- Work breakdown structure and decomposition
- Dependency mapping and critical path analysis
- Effort estimation and capacity planning
- Sprint/milestone planning and sequencing
- Risk identification and mitigation planning
- Stakeholder communication and progress tracking
- Ticket crafting with clear acceptance criteria

**Tool Access:**

| Category | Tools | Purpose |
|----------|-------|---------|
| Repository | `read/*`, `search`, `edit/createFile`, `edit/editFiles` | Read designs, create/update plans |
| GitHub | `gh-issues/*`, `gh-projects/*`, `gh-labels/*` | Create tickets, manage project items |
| Analysis | `sequentialthinking/*` | Decomposition and scheduling decisions |
| Documentation | `markdownlint/*` | Ensure document quality |

**Write Permissions:**
- `docs/plan/{{INIT_ID}}/**` - Implementation plans, evaluations, ticket manifests
- Tracking system issues (GitHub Issues, Jira Stories - create, label, link)

**Behavioral Focus:**
- Clear, actionable work items
- Explicit dependencies and blocking relationships
- Realistic effort estimates with justification
- Parallelization opportunities identification
- Risk-aware scheduling
- Testable acceptance criteria on all tickets

**Prompts Using This Agent:**
- `generate-implementation-plan.prompt.md`
- `review-implementation-plan.prompt.md`
- `evaluate-milestone.prompt.md`
- `generate-tickets.prompt.md`

---

## Prompt Specifications

### Phase 1 Prompts

#### `discover-initiative.prompt.md`

**Agent:** `product-manager`  
**Type:** Read-only scanner  
**Purpose:** Identify and vet new initiatives for SDLC

**Inputs:**
- Product backlog / roadmap references
- Stakeholder feedback or requests
- Strategic goals documentation

**Outputs:**
- Initiative Summary Document: `docs/initiatives/{{INIT_ID}}-summary.md`
  - Initiative ID and Name
  - Business Case (problem statement, expected value)
  - Key Stakeholders (with roles and contact)
  - Estimated Scope (T-shirt size: S/M/L/XL)
  - Timeline Constraints
  - Team Availability Assessment

**Quality Gate:**
- [ ] Business case is clear and measurable
- [ ] Stakeholders identified with ownership
- [ ] Scope estimate is justified
- [ ] No duplicate initiative exists

**Guardrails:**
- Read-only analysis; no side effects
- Flag ambiguous or incomplete business cases for clarification
- Do not assume stakeholder priorities; validate explicitly

---

#### `generate-prd.prompt.md`

**Agent:** `product-manager`  
**Type:** Writer  
**Purpose:** Create comprehensive Product Requirements Document

**Inputs:**
- Initiative Summary: `docs/initiatives/{{INIT_ID}}-summary.md`
- Stakeholder interview notes (if available)
- Market research / competitor analysis (if available)

**Outputs:**
- PRD Document: `docs/prd/{{INIT_ID}}-prd.md`
  - Executive Summary
  - Business Objectives & Success Metrics (SMART)
  - User Personas (with journey maps)
  - Functional Requirements (prioritized: MUST/SHOULD/COULD/WON'T)
  - Non-Functional Requirements (quantified targets)
  - Constraints & Assumptions
  - Out-of-Scope (explicit exclusions)
  - Open Questions (tracked for closure)

**Quality Gate Checklist:**
- [ ] All personas documented with journeys
- [ ] Success metrics are SMART
- [ ] Functional requirements use MoSCoW prioritization
- [ ] NFRs have quantified targets (latency, throughput, etc.)
- [ ] No ambiguous language ("intuitive", "user-friendly", etc.)
- [ ] Out-of-scope items explicitly listed
- [ ] All open questions have owners and target dates

**Guardrails:**
- Customer-centric language throughout
- Cite sources for any market claims
- Flag assumptions that need stakeholder validation
- Do not proceed with ambiguous requirements; clarify first

---

#### `review-prd.prompt.md`

**Agent:** `product-manager`  
**Type:** Analyst  
**Purpose:** Review PRD for completeness, clarity, and stakeholder alignment

**Inputs:**
- PRD Document: `docs/prd/{{INIT_ID}}-prd.md`
- Initiative Summary: `docs/initiatives/{{INIT_ID}}-summary.md`

**Outputs:**
- Review Report: `docs/prd/{{INIT_ID}}-prd-review.md`
  - Completeness Assessment (all sections present/missing)
  - Clarity Review (ambiguities identified with line references)
  - Gap Analysis (missing personas, requirements, constraints)
  - Risk Identification (unclear metrics, conflicting requirements)
  - Stakeholder Alignment Check
  - Recommendations with severity (BLOCKING/HIGH/MEDIUM/LOW)
  - Decision: APPROVED / APPROVED_WITH_ASSUMPTIONS / REMAND

**Decision Points:**
- ✅ **APPROVED:** No blocking gaps; proceed to HLD
- ⚠️ **APPROVED_WITH_ASSUMPTIONS:** Minor gaps documented; proceed with caveats
- 🔴 **REMAND:** Blocking gaps; return to `generate-prd` for updates

**Guardrails:**
- **Gap Discovery Workflow:**
  1. List ALL gaps discovered with specific section/line references
  2. For each gap, provide a suggested fix (concrete text or approach)
  3. Present full gap list with fixes to user
  4. Ask user: "Apply these fixes?" or "Would you like to adjust any suggestions?"
  5. Only modify PRD after explicit user approval
- MAY modify the PRD document with user approval
- Recommendations must cite specific sections/lines
- Blocking issues require clear remediation path
- Assumptions must be explicitly documented for downstream phases

---

### Phase 2 Prompts

#### `generate-hld.prompt.md`

**Agent:** `software-architect`  
**Type:** Writer  
**Purpose:** Create High-Level Design from approved PRD

**Inputs:**
- Approved PRD: `docs/prd/{{INIT_ID}}-prd.md`
- PRD Review (if exists): `docs/prd/{{INIT_ID}}-prd-review.md`
- Organization architecture guidelines
- Existing system documentation

**Outputs:**
- HLD Document: `docs/design/hld/{{INIT_ID}}-hld.md`
  - System Architecture Diagram (component boxes & interactions)
  - Major Components (with responsibilities)
  - Component Interactions (sequence/flow diagrams)
  - Technology Stack Decisions (with trade-off rationale)
  - Conceptual Data Model (entity relationships)
  - Security & Compliance Strategy
  - Integration Points (external systems, APIs)
  - High-Level API Contracts
  - Deployment Architecture
  - Scalability Approach

**Quality Gate Checklist:**
- [ ] Architecture trade-offs documented
- [ ] No single points of failure identified (or mitigated)
- [ ] All PRD requirements traceable to components
- [ ] Technology decisions justified
- [ ] Integration points with existing systems clear
- [ ] Security approach covers data, auth, encryption
- [ ] Scalability targets align with PRD NFRs

**Guardrails:**
- Search existing codebase for patterns before proposing new ones
- Cite existing architectural decisions that influence design
- Flag any PRD ambiguities discovered during design
- Do not over-engineer; right-size for requirements

---

#### `review-hld.prompt.md`

**Agent:** `software-architect`  
**Type:** Analyst  
**Purpose:** Review HLD for architectural soundness and PRD alignment

**Inputs:**
- HLD Document: `docs/design/hld/{{INIT_ID}}-hld.md`
- PRD Document: `docs/prd/{{INIT_ID}}-prd.md`
- Organization architecture standards

**Outputs:**
- Review Report: `docs/design/hld/{{INIT_ID}}-hld-review.md`
  - Architecture Consistency (standards alignment)
  - PRD Traceability Matrix (requirements → components)
  - Technology Decision Validation
  - Integration Feasibility Assessment
  - Security & Compliance Coverage
  - Scalability Feasibility Check (NFR achievability)
  - Risk Analysis & Mitigations
  - Decision: APPROVED / APPROVED_WITH_CONCERNS / REMAND / ESCALATE

**Decision Points:**
- ✅ **APPROVED:** Architecture sound; proceed to LLD
- ⚠️ **APPROVED_WITH_CONCERNS:** Minor concerns documented; proceed with mitigations
- 🔴 **REMAND:** Significant issues; return to `generate-hld` for revision
- 🔺 **ESCALATE:** Fundamental concerns; escalate to architecture review board

**Guardrails:**
- **Gap Discovery Workflow:**
  1. List ALL gaps discovered with specific section/line references
  2. For each gap, provide a suggested fix (concrete design change or addition)
  3. Present full gap list with fixes to user
  4. Ask user: "Apply these fixes?" or "Would you like to adjust any suggestions?"
  5. Only modify HLD after explicit user approval
- MAY modify the HLD document with user approval
- Validate against organizational standards explicitly
- Security concerns are always BLOCKING severity
- Traceability gaps must cite specific PRD requirements

---

### Phase 3 Prompts

#### `generate-lld.prompt.md`

**Agent:** `software-architect`  
**Type:** Writer  
**Purpose:** Create implementation-ready Low-Level Design

**Inputs:**
- Approved HLD: `docs/design/hld/{{INIT_ID}}-hld.md`
- HLD Review (if exists): `docs/design/hld/{{INIT_ID}}-hld-review.md`
- PRD Document: `docs/prd/{{INIT_ID}}-prd.md`
- Organization code standards

**Outputs:**
- LLD Document: `docs/design/lld/{{INIT_ID}}-lld.md`
  - Module/Package Structure
  - Class Diagrams & Responsibilities
  - API Specification (per endpoint):
    - Request schema (JSON schema or equivalent)
    - Response schema (success and error cases)
    - Status codes & error handling
    - Rate limiting & timeout strategy
  - Database Schema (normalized, with indexes)
  - Error Handling Framework
  - Logging Strategy (with correlation IDs)
  - Configuration (env vars, feature flags)
  - Test Strategy:
    - Unit test coverage targets
    - Integration test scenarios
    - Contract tests
    - Performance tests (if applicable)
  - Performance Targets (P50, P99, throughput)
  - Monitoring & Observability
  - Security Implementation Details

**Quality Gate Checklist:**
- [ ] API contracts are complete and versioning-aware
- [ ] Database schema normalization decision documented (3NF+ OR denormalized with justification and user approval)
- [ ] Database indexes defined for query patterns
- [ ] Error handling covers all failure modes
- [ ] Logging includes correlation IDs
- [ ] Feature flags defined for safe rollout
- [ ] Test strategy covers happy & sad paths
- [ ] Performance targets are measurable
- [ ] No ambiguities remain for implementation

**Guardrails:**
- Search existing codebase for reusable patterns
- Align with existing code conventions
- Flag any HLD inconsistencies discovered
- Design must be implementable by developers without further clarification

---

#### `review-lld.prompt.md`

**Agent:** `software-architect`  
**Type:** Analyst  
**Purpose:** Review LLD for implementation readiness

**Inputs:**
- LLD Document: `docs/design/lld/{{INIT_ID}}-lld.md`
- HLD Document: `docs/design/hld/{{INIT_ID}}-hld.md`
- PRD Document: `docs/prd/{{INIT_ID}}-prd.md`
- Code standards

**Outputs:**
- Review Report: `docs/design/lld/{{INIT_ID}}-lld-review.md`
  - Implementation Readiness Assessment
  - Component Detail Completeness
  - Database Schema Validation (normalization, constraints)
  - API Contract Completeness (all endpoints, all cases)
  - Test Coverage Assessment (all AC covered)
  - Error Handling Verification
  - Ambiguity Detection (questions developers would have)
  - HLD Traceability (all components detailed)
  - Decision: APPROVED / APPROVED_WITH_CLARIFICATIONS / REMAND

**Decision Points:**
- ✅ **APPROVED:** Implementation-ready; proceed to planning
- ⚠️ **APPROVED_WITH_CLARIFICATIONS:** Minor clarifications documented for developers
- 🔴 **REMAND:** Blocking gaps; return to `generate-lld` for completion

**Guardrails:**
- **Gap Discovery Workflow:**
  1. List ALL gaps discovered with specific section/line references
  2. For each gap, provide a suggested fix (concrete specification or detail)
  3. Present full gap list with fixes to user
  4. Ask user: "Apply these fixes?" or "Would you like to adjust any suggestions?"
  5. Only modify LLD after explicit user approval
- MAY modify the LLD document with user approval
- Simulate developer perspective: what questions would they have?
- Missing test cases are HIGH severity
- Ambiguous API contracts are BLOCKING

---

### Phase 4 Prompts

#### `generate-implementation-plan.prompt.md`

**Agent:** `delivery-manager`  
**Type:** Writer  
**Purpose:** Break LLD into executable milestones with timeline derived from work breakdown

**Inputs:**
- Approved LLD: `docs/design/lld/{{INIT_ID}}-lld.md`
- LLD Review (if exists): `docs/design/lld/{{INIT_ID}}-lld-review.md`
- Team capacity: **Number of parallel workers** (engineers/teams working simultaneously)
- Risk factors and dependencies

**Outputs:**
- Implementation Plan: `docs/plan/{{INIT_ID}}/implementation.md`
  - **Derived Timeline** & Critical Path (calculated from work breakdown and parallel capacity)
  - Milestone Breakdown (for each milestone):
    - Milestone ID (M1, M2, ...)
    - Objectives & Acceptance Criteria
    - Components/Features Included
    - Estimated Effort (story points or time)
    - Dependencies on Other Milestones
    - Success Metrics
    - Rollout/Deployment Plan
  - **Parallel Work Schedule** (based on number of workers)
  - Team Assignments (if known)
  - Risk Register with Mitigations
  - **Timeline Validation Note:** If derived timeline doesn't meet stakeholder needs, milestone scope re-evaluation required

**Quality Gate Checklist:**
- [ ] All LLD components assigned to a milestone
- [ ] Milestones are independently valuable (can ship alone)
- [ ] Dependencies are explicit and acyclic
- [ ] Effort estimates are time-boxed and justified
- [ ] Critical path is identified
- [ ] Parallel opportunities maximized
- [ ] Each milestone has measurable success criteria
- [ ] Rollout strategy is low-risk (canary/blue-green where applicable)

**Guardrails:**
- Focus on clear communication and sequencing
- **Timeline is DERIVED from work breakdown, not a constraint**
- Ask user for number of parallel workers at start
- Maximize parallel work opportunities given worker count
- If derived timeline is unacceptable to stakeholders, flag for **milestone scope re-evaluation** (not artificial compression)
- Do not create overly granular milestones (<1 week) or overly broad (>6 weeks)
- Dependencies must be justified (not assumed)

---

#### `review-implementation-plan.prompt.md`

**Agent:** `delivery-manager`  
**Type:** Analyst  
**Purpose:** Review implementation plan for feasibility

**Inputs:**
- Implementation Plan: `docs/plan/{{INIT_ID}}/implementation.md`
- LLD Document: `docs/design/lld/{{INIT_ID}}-lld.md`
- Team capacity information

**Outputs:**
- Review Report: `docs/plan/{{INIT_ID}}/implementation-review.md`
  - Feasibility Assessment (timeline vs. capacity)
  - LLD Coverage Check (all components assigned)
  - Dependency Analysis (critical path, circular deps)
  - Effort Validation (estimates reasonable)
  - Milestone Independence Check (parallel safety)
  - Rollout & Deployment Review
  - Risk Identification
  - Resource Allocation Assessment
  - Decision: APPROVED / APPROVED_WITH_ADJUSTMENTS / REMAND / ESCALATE

**Decision Points:**
- ✅ **APPROVED:** Plan is feasible; proceed to milestone evaluation
- ⚠️ **APPROVED_WITH_ADJUSTMENTS:** Minor timeline/scope adjustments noted
- 🔴 **REMAND:** Significant issues; return to `generate-implementation-plan`
- 🔺 **ESCALATE:** Requires scope/timeline negotiation with stakeholders

**Guardrails:**
- **Gap Discovery Workflow:**
  1. List ALL gaps discovered with specific section/line references
  2. For each gap, provide a suggested fix (milestone adjustment, dependency fix, estimate revision)
  3. Present full gap list with fixes to user
  4. Ask user: "Apply these fixes?" or "Would you like to adjust any suggestions?"
  5. Only modify implementation plan after explicit user approval
- MAY modify the implementation plan with user approval
- Challenge unrealistic estimates with rationale
- Circular dependencies are BLOCKING
- Missing LLD coverage is BLOCKING

---

### Phase 5 Prompts

#### `evaluate-milestone.prompt.md`

**Agent:** `delivery-manager`  
**Type:** Analyzer  
**Purpose:** Determine if milestone needs recursive SDLC or direct tickets

**Inputs:**
- Milestone Specification (from implementation plan)
- LLD excerpt (relevant sections)
- Team familiarity assessment
- Complexity scoring criteria

**Outputs:**
- Evaluation Report: `docs/plan/{{INIT_ID}}/{{MILESTONE_ID}}-evaluation.md`
  - Complexity Score (using weighted criteria)
  - Score Breakdown by Factor
  - Recommendation: DIRECT_TICKETS / RECURSIVE_SDLC
  - Rationale for Recommendation
  - If RECURSIVE_SDLC:
    - Sub-Initiative ID assignment
    - Scope definition for mini-SDLC
    - Expected output (reduced PRD focus)

**Complexity Scoring Criteria:**

| Factor | Low (1pt) | Medium (2pt) | High (3pt) |
|--------|-----------|--------------|------------|
| Components | 1-2 | 3-5 | 6+ |
| APIs | <5 | 5-15 | 15+ |
| DB Tables | <10 | 10-30 | 30+ |
| Integration Points | 0 | 1-2 | 3+ |
| Data Processing | Simple CRUD | Workflow logic | Complex algorithms |
| Team Familiarity | Familiar | Somewhat | Unfamiliar |
| Tech Stack Novelty | Proven | Mostly proven | New/experimental |

**Thresholds:**
- **≤10 points:** Low complexity → DIRECT_TICKETS
- **11-15 points:** Medium complexity → Evaluate (lean toward DIRECT_TICKETS unless risk factors)
- **≥16 points:** High complexity → RECURSIVE_SDLC

**Decision Points:**
- ✅ **DIRECT_TICKETS:** Proceed to `generate-tickets` for this milestone
- 🔄 **RECURSIVE_SDLC:** Create sub-initiative; re-enter Phase 1 with reduced scope

**Guardrails:**
- Score all factors; do not skip
- Justify recommendation with specific factors
- RECURSIVE_SDLC recommendation requires explicit sub-initiative scope definition
- User approval required before creating sub-initiative

---

### Phase 6 Prompts

#### `generate-tickets.prompt.md`

**Agent:** `delivery-manager`  
**Type:** Writer  
**Purpose:** Create tracking issues from milestone specifications

**Inputs:**
- Milestone Specification (from implementation plan)
- Milestone Evaluation (confirming DIRECT_TICKETS)
- LLD excerpt (relevant sections for AC details)
- Tracking system type (GitHub Issues, Jira Stories, etc.)

**Outputs:**

**Tracking Issue Hierarchy:**

**Option A: GitHub Issues**
- GitHub Milestone created: `{{INIT_ID}}-{{MILESTONE_ID}}`
- GitHub Issues created (tied to milestone):
  - Title: `[{{INIT_ID}}-{{MILESTONE_ID}}-{{SEQ}}] {{TITLE}}`
  - Description: Full context, linked to parent milestone/initiative
  - Acceptance Criteria: Checklist from LLD
  - Story Points & Priority
  - Labels: component, type
  - Dependencies: `blocked-by` links between issues
  - Milestone: Assigned to GitHub Milestone

**Option B: Jira Stories**
- Jira Epic created: `{{INIT_ID}}-{{MILESTONE_ID}}` (represents milestone)
- Jira Stories created (as child stories under Epic):
  - Summary: `[{{INIT_ID}}-{{MILESTONE_ID}}-{{SEQ}}] {{TITLE}}`
  - Description: Full context, linked to parent Epic/initiative
  - Acceptance Criteria: Checklist from LLD
  - Story Points & Priority
  - Labels: component, type
  - Epic Link: Linked to parent Epic
  - Dependencies: Blocker/Blocked-by links

**Common Output:**
- Ticket Manifest: `docs/plan/{{INIT_ID}}/{{MILESTONE_ID}}-tickets.md`
  - List of all tracking issues created
  - Dependency graph
  - Recommended sprint structure
  - Hierarchy reference (Milestone/Epic → Issues/Stories)

**Quality Gate Checklist:**
- [ ] All tickets follow naming convention
- [ ] Story points sum to milestone estimate (±10%)
- [ ] Dependencies explicitly linked
- [ ] AC are testable (no subjective language)
- [ ] Each ticket ≤13 SP (split if larger)
- [ ] No duplicate work across tickets
- [ ] All AC traceable to LLD
- [ ] Tickets ordered by dependency (critical path first)

**Guardrails:**
- This is an END STATE; do not chain to ticket execution
- Create hierarchical structure in tracking system (Milestone/Epic → Issues/Stories)
- All tracking issues must be independently testable
- Avoid creating issues that are too small (<2 SP) or too large (>13 SP)
- Dependencies must form a DAG (no circular dependencies)
- Get user confirmation before creating tracking issues in system

---

## Recursive SDLC Handling

When `evaluate-milestone` recommends RECURSIVE_SDLC, the milestone becomes a **sub-initiative** and re-enters the SDLC at Phase 1 with reduced scope.

### Recursive Flow

```text
Parent Initiative: INIT-2026-001
├── Phase 1-4: Complete for parent
├── Phase 5: Milestone Evaluation
│   ├── M1: DIRECT_TICKETS → Phase 6
│   ├── M2: RECURSIVE_SDLC ⚠️
│   │   └── Sub-Initiative: INIT-2026-001-M2
│   │       ├── Phase 1: generate-prd (scoped to M2 features)
│   │       ├── Phase 2: generate-hld (M2 architecture)
│   │       ├── Phase 3: generate-lld (M2 details)
│   │       ├── Phase 4: implementation plan (M2 milestones)
│   │       ├── Phase 5: evaluate milestones
│   │       │   ├── M2.1: DIRECT_TICKETS → Phase 6
│   │       │   └── M2.2: DIRECT_TICKETS → Phase 6
│   │       └── Phase 6: Tickets created for M2.1, M2.2
│   └── M3: DIRECT_TICKETS → Phase 6
└── Phase 6: Tickets created for M1, M3 (M2 handled recursively)
```

### Key Principles for Recursion

1. **Same Agents & Prompts:** Recursive SDLCs use the identical agent modes and prompts
2. **Reduced Scope:** The decomposed milestone defines the boundary of the sub-initiative
3. **Parent Skips Ticket Creation:** Milestones marked RECURSIVE_SDLC do not generate tickets at parent level
4. **ID Hierarchy:** Sub-initiatives use hierarchical IDs (e.g., `INIT-2026-001-M2`)
5. **Traceability Maintained:** Sub-initiative documents reference parent initiative

### Diagram Correction Note

The SDLC_INITIATIVE_PLANNING.md workflow diagram should be updated to reflect that:
- Milestones marked RECURSIVE_SDLC skip Phase 6 at the parent level
- Tickets are only created in the leaf nodes (direct ticket milestones)
- Parent initiative completion waits for all recursive sub-initiatives to complete

---

## Integration with Ticket Flow

Once tickets are created by `generate-tickets`, execution follows the **[TICKET_FLOW.md](../TICKET_FLOW.md)** process as a **separate subprocess**.

### Handoff Points

| SDLC Phase | SDLC Output | Ticket Flow Input |
|------------|-------------|-------------------|
| Phase 6: generate-tickets | GitHub Issues created | find-next-ticket scans for executable issues |

### Separation of Concerns

- **SDLC (This Document):** Strategic planning → Requirements → Design → Work Breakdown → Tickets
- **Ticket Flow:** Tactical execution → Plan → Implement → Review → PR → Merge

### No Automatic Chaining

- `generate-tickets` is an **end state** for the SDLC
- Iteration over tickets via TICKET_FLOW is a **manual, separate phase**
- This separation allows:
  - Human review of generated tickets before execution
  - Prioritization and re-ordering as needed
  - Team assignment and capacity balancing
  - Sprint planning ceremonies

---

## Summary: Consolidated Artifacts

### Agents to Create (3 total)

| Agent File | Persona | Phases | Prompts |
|------------|---------|--------|---------|
| `.github/agents/product-manager.agent.md` | Product Manager | 1 | 3 prompts |
| `.github/agents/software-architect.agent.md` | Software Architect | 2, 3 | 4 prompts |
| `.github/agents/delivery-manager.agent.md` | Delivery Manager | 4, 5, 6 | 5 prompts |

### Prompts to Create (12 total)

| Prompt File | Agent | Phase | Step |
|-------------|-------|-------|------|
| `.github/prompts/discover-initiative.prompt.md` | product-manager | 1 | 1.1 |
| `.github/prompts/generate-prd.prompt.md` | product-manager | 1 | 1.2 |
| `.github/prompts/review-prd.prompt.md` | product-manager | 1 | 1.3 |
| `.github/prompts/generate-hld.prompt.md` | software-architect | 2 | 2.1 |
| `.github/prompts/review-hld.prompt.md` | software-architect | 2 | 2.2 |
| `.github/prompts/generate-lld.prompt.md` | software-architect | 3 | 3.1 |
| `.github/prompts/review-lld.prompt.md` | software-architect | 3 | 3.2 |
| `.github/prompts/generate-implementation-plan.prompt.md` | delivery-manager | 4 | 4.1 |
| `.github/prompts/review-implementation-plan.prompt.md` | delivery-manager | 4 | 4.2 |
| `.github/prompts/evaluate-milestone.prompt.md` | delivery-manager | 5 | 5.1 |
| `.github/prompts/generate-tickets.prompt.md` | delivery-manager | 6 | 6.1 |

### Document Structure

```text
.github/
├── agents/
│   ├── product-manager.agent.md      (NEW)
│   ├── software-architect.agent.md   (NEW)
│   ├── delivery-manager.agent.md     (NEW)
│   └── [existing agents...]
└── prompts/
    ├── discover-initiative.prompt.md              (NEW)
    ├── generate-prd.prompt.md                     (NEW)
    ├── review-prd.prompt.md                       (NEW)
    ├── generate-hld.prompt.md                     (NEW)
    ├── review-hld.prompt.md                       (NEW)
    ├── generate-lld.prompt.md                     (NEW)
    ├── review-lld.prompt.md                       (NEW)
    ├── generate-implementation-plan.prompt.md     (NEW)
    ├── review-implementation-plan.prompt.md       (NEW)
    ├── evaluate-milestone.prompt.md               (NEW)
    ├── generate-tickets.prompt.md                 (NEW)
    └── [existing prompts...]
```

---

**Last Updated:** January 28, 2026  
**Version:** 1.0  
**Status:** Ready for implementation
