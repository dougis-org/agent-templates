# Quality Gates

## Per-Phase Quality Gates

Each phase must pass its quality gate before the orchestrator advances to the next phase.

---

### DISCOVERY Gate

| Check | Validation | Blocker? |
| ----- | --------- | -------- |
| Ticket loaded | Ticket ID resolved, title and description cached | Yes |
| Platform confirmed | GitHub or Jira platform established | Yes |

---

### PLANNING Gate

| Check | Validation | Blocker? |
| ----- | --------- | -------- |
| Plan file exists | File at `docs/plan/tickets/{TICKET_ID}-plan.md` | Yes |
| Plan completeness | All 11 required sections present | Yes |
| No placeholders | No ALL-CAPS placeholder text remaining | Yes |
| Decomposition evaluated | Section 6 contains explicit keep/split decision | Yes |
| ACs normalized | Section 3 has numbered, testable criteria | Yes |

---

### ANALYSIS Gate

| Check | Validation | Blocker? |
| ----- | --------- | -------- |
| Analysis complete | Analysis report produced | Yes |
| No CRITICAL issues | Zero CRITICAL severity findings | Yes |
| HIGH issues documented | All HIGH issues have remediation plan | No (warning) |

---

### PLAN_CHECKPOINT Gate (Human)

| Check | Validation | Blocker? |
| ----- | --------- | -------- |
| Human approval | `checkpoints.planApproved == true` | Yes |

---

### IMPLEMENTATION Gate

| Check | Validation | Blocker? |
| ----- | --------- | -------- |
| Tests written | Test files created (RED phase complete) | Yes |
| Tests pass | All tests pass locally (GREEN phase complete) | Yes |
| Build succeeds | No compile errors | Yes |
| Linters pass | All applicable linters clean | Yes |
| Coverage maintained | No unjustified coverage regression | Yes |

---

### LOCAL_REVIEW Gate

| Check | Validation | Blocker? |
| ----- | --------- | -------- |
| Self-review complete | Review report generated | Yes |
| No blocking issues | No RED-status items in review | Yes |

---

### PR_CREATION Gate

| Check | Validation | Blocker? |
| ----- | --------- | -------- |
| PR exists | GitHub API confirms PR open | Yes |
| PR has description | Title + body populated | Yes |
| Ticket linked | PR references ticket ID | Yes |
| Branch clean | No uncommitted changes | Yes |

---

### PR_CHECKPOINT Gate (Human)

| Check | Validation | Blocker? |
| ----- | --------- | -------- |
| Human approval | `checkpoints.prApproved == true` | Yes |

---

### CODE_REVIEW Gate

| Check | Validation | Blocker? |
| ----- | --------- | -------- |
| Review complete | Code review submitted | Yes |
| CI/CD green | All status checks pass | Yes |
| PR merged | GitHub API confirms merge | Yes |

---

## Failure Handling

When a quality gate fails:

1. **Log the failure** in phase history with error details
2. **Retry the phase** (up to `maxRetries`)
3. **If retries exhausted**, present failure to human:

```text
=== QUALITY GATE FAILURE ===

Phase: [PHASE]
Gate: [GATE_NAME]
Attempts: [COUNT]/[MAX_RETRIES]
Error: [DETAILS]

Options:
1. Retry phase (reset retry count)
2. Skip gate (requires justification — logged in state)
3. Abort workflow (state preserved for later resume)
```

1. **Never silently skip a blocker gate** — human must explicitly choose option 2 with justification
