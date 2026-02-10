# Segment Handoff Protocol

## Purpose

Define how context and artifacts pass between automated segments and human gates,
ensuring no information is lost across boundaries.

---

## Handoff Structure

Each segment produces a **handoff payload** stored in the orchestrator state under `artifacts`:

```json
{
  "artifacts": {
    "segment1_planFile": "docs/plan/tickets/7-plan.md",
    "segment1_analysisSummary": "No CRITICAL issues; 2 MEDIUM findings",
    "segment1_riskLevel": "MEDIUM",
    "segment1_decomposition": "single-ticket",
    "segment2_branchName": "feature/7-add-auth-endpoint",
    "segment2_prUrl": "https://github.com/org/repo/pull/42",
    "segment2_prNumber": 42,
    "segment2_testCount": 24,
    "segment2_coveragePercent": 87,
    "segment2_qualityGateStatus": "ALL_PASS",
    "segment3_mergeCommitSha": "abc123def",
    "segment3_postMergeCiStatus": "GREEN"
  }
}
```

---

## Segment 1 → Gate 1 Handoff

**Producer:** segment-1-plan.prompt.md
**Consumer:** gate-1-plan-review.prompt.md

**Required artifacts:**

| Artifact | Key | Description |
| -------- | --- | ----------- |
| Plan file path | `segment1_planFile` | Path to the completed plan file |
| Analysis summary | `segment1_analysisSummary` | One-line analysis result |
| Risk level | `segment1_riskLevel` | Overall risk: LOW / MEDIUM / HIGH / CRITICAL |
| Decomposition | `segment1_decomposition` | `"single-ticket"` or `"split-N"` with slice details |
| AC count | `segment1_acCount` | Number of acceptance criteria |

**Gate 1 presentation** reads these artifacts to build the checkpoint summary.

---

## Gate 1 → Segment 2 Handoff

**Producer:** gate-1-plan-review.prompt.md (on approval)
**Consumer:** segment-2-implement.prompt.md

**Required context:**

| Context | Source | Description |
| ------- | ------ | ----------- |
| Plan file | `state.planFilePath` | Approved plan for implementation |
| Ticket data | Cached from discovery | Title, description, ACs |
| Approval status | `checkpoints.planApproved` | Must be `true` |

---

## Segment 2 → Gate 2 Handoff

**Producer:** segment-2-implement.prompt.md
**Consumer:** gate-2-pr-review.prompt.md

**Required artifacts:**

| Artifact | Key | Description |
| -------- | --- | ----------- |
| Branch name | `segment2_branchName` | Feature branch with changes |
| PR URL | `segment2_prUrl` | Pull request URL |
| PR number | `segment2_prNumber` | Pull request number |
| Test count | `segment2_testCount` | Total tests written |
| Coverage | `segment2_coveragePercent` | Code coverage percentage |
| Quality status | `segment2_qualityGateStatus` | `ALL_PASS` or details of failures |
| Files changed | `segment2_filesChanged` | Count of files modified |
| AC coverage | `segment2_acCoverage` | Map of AC# → PASS/FAIL |

**Gate 2 presentation** reads these artifacts to build the PR checkpoint summary.

---

## Gate 2 → Segment 3 Handoff

**Producer:** gate-2-pr-review.prompt.md (on approval)
**Consumer:** segment-3-finalize.prompt.md

**Required context:**

| Context | Source | Description |
| ------- | ------ | ----------- |
| PR URL | `state.prUrl` | Approved PR for merge |
| PR number | `state.prNumber` | PR number for API calls |
| Ticket data | Cached from discovery | For issue closure |
| Approval status | `checkpoints.prApproved` | Must be `true` |

---

## Segment 3 → DONE

**Producer:** segment-3-finalize.prompt.md

**Final artifacts:**

| Artifact | Key | Description |
| -------- | --- | ----------- |
| Merge SHA | `segment3_mergeCommitSha` | Merged commit SHA |
| Post-merge CI | `segment3_postMergeCiStatus` | `GREEN` or failure details |
| Final summary | `segment3_summary` | Complete workflow summary |

---

## Context Preservation Rules

1. **Never discard state** between segments — all prior artifacts remain accessible
2. **Append, don't overwrite** — new artifacts add to existing; only update timestamps
3. **Human feedback** from gate rejections is stored in phase history for audit
4. **On retry**, preserve prior attempt artifacts with attempt-number suffix
