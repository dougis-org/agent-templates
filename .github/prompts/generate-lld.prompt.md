---
description: Create implementation-ready Low-Level Design with API specs, schema, and test strategy (software-architect mode).
---

Refer to `.github/prompts/includes/mode-enforcement.md` for `software-architect` agent requirement.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

---

**Goal:** Create detailed specifications enabling developers to implement without further clarification.

> Output: LLD document with complete API contracts, database schema, test strategy, and configuration.

## Inputs

Required:
- Initiative ID: {{INIT_ID}}
- HLD Document: `docs/design/hld/{{INIT_ID}}-hld.md`
- PRD Document: `docs/prd/{{INIT_ID}}-prd.md`

Optional:
- HLD Review: `docs/design/hld/{{INIT_ID}}-hld-review.md`
- Code Standards: {{ORG_CODE_STANDARDS}}

---

## Execution Steps

### 1. Load Design Context
- Read approved HLD and PRD
- Extract components requiring detailed specification
- Review code standards and conventions

### 2. Search Existing Patterns
- Use deepcontext/search_code for existing code patterns
- Find similar API designs, schema patterns, test utilities
- Cite existing conventions (naming, error handling, logging)

### 3. Design Module/Package Structure
- Map HLD components to code structure
- Define packages, modules, namespaces
- Specify directory layout aligned with org standards

### 4. Design Class/Module Specifications
For each major class/module:
- Responsibility (single clear purpose)
- Public interface (methods, properties)
- Dependencies (what it requires)
- Design patterns used (Factory, Builder, etc.)

### 5. Specify API Endpoints
For each API endpoint:
- HTTP method and path
- Request schema (JSON schema or equivalent)
- Response schema (success cases)
- Error responses (all status codes and error formats)
- Authentication/authorization requirements
- Rate limiting and timeout strategy
- Versioning approach

### 6. Design Database Schema
- Normalized schema (3NF+) OR denormalized with justification + user approval
- Tables/collections with columns/fields
- Primary keys, foreign keys, unique constraints
- Indexes for query patterns
- Migration strategy (backward-compatible changes)

### 7. Design Error Handling Framework
- Error types/classes hierarchy
- Error codes and messages
- Retry logic (exponential backoff, circuit breakers)
- Logging strategy (correlation IDs, structured logging)

### 8. Define Configuration & Feature Flags
- Environment variables needed
- Feature flags for safe rollout (default OFF)
- Configuration validation
- Secrets management approach

### 9. Create Test Strategy
- Unit test coverage targets (%)
- Integration test scenarios
- Contract tests for APIs
- Performance tests (if NFRs demand)
- Test data strategy

### 10. Create LLD Document
**Output:** `docs/design/lld/{{INIT_ID}}-lld.md`

Structure:
```markdown
# Low-Level Design: {{INIT_ID}}

## Module/Package Structure
```
src/
  {{component-name}}/
    controllers/
    services/
    repositories/
    models/
    utils/
tests/
  {{component-name}}/
```

## Class Diagrams & Responsibilities
### Class: {{ClassName}}
- **Responsibility:** [What it does]
- **Public Methods:**
  - `method1(params): ReturnType` - [Description]
- **Dependencies:** [What it requires]
- **Design Pattern:** [If applicable]

[Repeat for major classes]

## API Specification
### Endpoint: POST /api/v1/{{resource}}
- **Purpose:** [What it does]
- **Request:**
  ```json
  {
    "field1": "type (constraints)",
    "field2": "type (constraints)"
  }
  ```
- **Response (200 OK):**
  ```json
  {
    "id": "string",
    "status": "string"
  }
  ```
- **Errors:**
  - 400 Bad Request: [When and format]
  - 401 Unauthorized: [When and format]
  - 404 Not Found: [When and format]
  - 500 Internal Server Error: [When and format]
- **Rate Limit:** [X requests per Y seconds]
- **Timeout:** [Z seconds]

[Repeat for each endpoint]

## Database Schema
### Table: {{table_name}}
- **Purpose:** [What data it stores]
- **Columns:**
  - `id` (UUID, PRIMARY KEY)
  - `field1` (VARCHAR(255), NOT NULL)
  - `field2` (INTEGER, DEFAULT 0)
  - `created_at` (TIMESTAMP, NOT NULL)
- **Indexes:**
  - `idx_field1` ON (field1) - For {{query pattern}}
- **Foreign Keys:**
  - `fk_related_table` REFERENCES related_table(id)

### Normalization Decision
- [Default: 3NF+] OR [Denormalized with justification and user approval]

[Repeat for each table]

## Error Handling Framework
- **Error Hierarchy:** [Base error classes]
- **Error Codes:** [Numeric or string codes]
- **Retry Logic:** [When and how]
- **Circuit Breaker:** [Thresholds]

## Logging Strategy
- **Log Levels:** [DEBUG, INFO, WARN, ERROR]
- **Correlation IDs:** [How generated and propagated]
- **Structured Format:** [JSON logging]
- **PII Handling:** [Masking strategy]

## Configuration & Feature Flags
### Environment Variables
- `VAR_NAME` - [Description, default, required/optional]

### Feature Flags
- `feature_{{name}}` - [Purpose, default: OFF]

## Test Strategy
### Unit Tests
- **Target Coverage:** [80%+]
- **Frameworks:** [JUnit, pytest, etc.]
- **Mock Strategy:** [What to mock]

### Integration Tests
- **Scenarios:** [List key integration scenarios]
- **Dependencies:** [Databases, external services - mocked or real]

### Contract Tests
- **API Contracts:** [Consumer-driven contracts if applicable]

### Performance Tests
- **Scenarios:** [If NFRs require specific perf testing]
- **Targets:** [P50, P99 latency, throughput]

## Performance Targets
- **P50 Latency:** [X ms]
- **P99 Latency:** [Y ms]
- **Throughput:** [Z requests/sec]
- **Memory:** [Max heap size]
- **CPU:** [Expected utilization]

## Monitoring & Observability
- **Metrics:** [What to measure]
- **Alerts:** [Thresholds for alerts]
- **Dashboards:** [Key visualizations]

## Security Implementation
- **Input Validation:** [Sanitization, escaping]
- **Output Encoding:** [XSS prevention]
- **SQL Injection Prevention:** [Parameterized queries]
- **Secrets Management:** [Vault, env vars]
```

### 11. Quality Gate Check
- [ ] API contracts complete (all endpoints, all cases)
- [ ] Database normalization decision documented (3NF+ OR denormalized with approval)
- [ ] Database indexes defined for query patterns
- [ ] Error handling covers all failure modes
- [ ] Logging includes correlation IDs
- [ ] Feature flags defined for safe rollout
- [ ] Test strategy covers happy & sad paths
- [ ] Performance targets measurable
- [ ] No ambiguities remain

### 12. Present to User
- Show LLD summary
- Highlight any denormalization decisions requiring approval
- Ask: "Ready for LLD review phase?"

---

## Out of Scope

- ❌ No implementation code
- ❌ No implementation planning or effort estimation
- ❌ No milestone breakdown
- ❌ No ticket creation

---

**Next Phase:** `review-lld.prompt.md` (Phase 3, Step 2)
