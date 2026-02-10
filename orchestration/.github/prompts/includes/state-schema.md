# Orchestrator State Schema

## State File Location

```text
docs/plan/tickets/{TICKET_ID}-orchestrator-state.json
```

- GitHub issues: strip leading `#` (e.g., `#7` → `7-orchestrator-state.json`)
- Jira tickets: use key as-is (e.g., `PROJ-123-orchestrator-state.json`)

---

## State Structure

```json
{
  "ticketId": "#7",
  "platform": "github",
  "currentPhase": "DISCOVERY",
  "currentSegment": 0,
  "planFilePath": null,
  "branchName": null,
  "prUrl": null,
  "prNumber": null,
  "checkpoints": {
    "planApproved": false,
    "planApprovedAt": null,
    "prApproved": false,
    "prApprovedAt": null
  },
  "retryCount": {},
  "maxRetries": 2,
  "phaseHistory": [],
  "artifacts": {},
  "createdAt": "2026-01-31T04:15:00Z",
  "updatedAt": "2026-01-31T04:15:00Z"
}
```

---

## Field Definitions

| Field | Type | Description |
| ----- | ---- | ----------- |
| `ticketId` | string | Original ticket identifier (`#7` or `PROJ-123`) |
| `platform` | `"github"` or `"jira"` | Detected platform |
| `currentPhase` | WorkflowPhase | Current phase in the workflow |
| `currentSegment` | number (0-3) | Current segment (0=init, 1=plan, 2=impl, 3=finalize) |
| `planFilePath` | string or null | Path to plan file once created |
| `branchName` | string or null | Working branch name |
| `prUrl` | string or null | Pull request URL once created |
| `prNumber` | number or null | Pull request number |
| `checkpoints.planApproved` | boolean | Human approved the plan |
| `checkpoints.planApprovedAt` | string or null | ISO 8601 approval timestamp |
| `checkpoints.prApproved` | boolean | Human approved the PR |
| `checkpoints.prApprovedAt` | string or null | ISO 8601 approval timestamp |
| `retryCount` | Record&lt;phase, number&gt; | Retry attempts per phase |
| `maxRetries` | number | Max retries before escalation (default: 2) |
| `phaseHistory` | PhaseRecord[] | Audit trail of all phases |
| `artifacts` | Record&lt;string, string&gt; | Named artifacts (file paths, URLs) |
| `createdAt` | string | ISO 8601 creation timestamp |
| `updatedAt` | string | ISO 8601 last-update timestamp |

---

## Phase Record Structure

Each completed phase appends a record:

```json
{
  "phase": "PLANNING",
  "segment": 1,
  "startedAt": "2026-01-31T04:20:00Z",
  "completedAt": "2026-01-31T04:35:00Z",
  "status": "completed",
  "summary": "Plan created with 11 sections, no decomposition needed",
  "artifacts": ["docs/plan/tickets/7-plan.md"],
  "error": null
}
```

| Field | Type | Description |
| ----- | ---- | ----------- |
| `phase` | WorkflowPhase | Phase name |
| `segment` | number | Segment number (1, 2, or 3) |
| `startedAt` | string | ISO 8601 start timestamp |
| `completedAt` | string or null | ISO 8601 completion timestamp |
| `status` | `"in-progress"` or `"completed"` or `"failed"` | Phase status |
| `summary` | string | Short description of outcome |
| `artifacts` | string[] | Paths or URLs produced |
| `error` | string or null | Error message if failed |

---

## WorkflowPhase Values

```text
DISCOVERY
PLANNING
ANALYSIS
PLAN_CHECKPOINT
IMPLEMENTATION
LOCAL_REVIEW
PR_CREATION
PR_CHECKPOINT
CODE_REVIEW
DONE
```

---

## Initialization Rules

1. Parse ticket ID using ticket detection logic
2. Create state object with `currentPhase = "DISCOVERY"` and `currentSegment = 0`
3. Persist to state file path
4. Validate JSON is valid before writing

## Resumption Rules

1. Check for existing state file at expected path
2. If found and no `RESUME_FROM_PHASE` specified: ask user to resume or start fresh
3. If `RESUME_FROM_PHASE` specified: validate phase is reachable, set as current, continue
4. If state file corrupted: create backup with `.backup` suffix, offer reset or manual recovery

## Update Rules

1. Update `updatedAt` on every write
2. Append to `phaseHistory` on every phase transition
3. Write atomically (write to temp, validate, rename)
4. Never delete phase history entries (append-only)
