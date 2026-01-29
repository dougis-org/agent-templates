# Implementation Plan: Issue #3

## 1) Summary

- **Ticket:** [#3](https://github.com/dougis-org/agent-templates/issues/3)
- **One-liner:** Create a prompt that enables other repositories to clone agent-templates, compare and selectively copy .github folder contents with conflict resolution
- **Related milestone(s):** N/A
- **Out of scope:**
  - Automatic modifications to target repository configuration files outside .github folder
  - Synchronization of non-.github directories
  - Two-way sync or push-back to agent-templates
  - Version control of copied files beyond Git tracking
  - Custom merge strategies beyond user-selected options (overwrite all/none/merge)

---

## 2) Requirements

1. Provide a standalone prompt at `.github/prompts/sync-agent-templates.prompt.md` that runs from any base repository.
2. Acquire the agent-templates `.github` contents without using shell file operations; prefer MCP tools for file access and writes.
3. Detect and categorize files as new (auto-copy) or conflicting (exists in both repos).
4. Auto-copy new files without user prompts.
5. Require explicit user confirmation before any overwrite or merge of existing files.
6. Provide three deterministic user options for conflicts: overwrite all, overwrite none, attempt merge (template prioritized).
7. For merge option, show side-by-side comparison and proposed merged output per file before writing.
8. Always clean up temporary artifacts (clone/archive extraction) on success or error.
9. Provide clear progress and summary output for each phase.

---

## 3) Acceptance Criteria (normalized)

1. Prompt file exists at `.github/prompts/sync-agent-templates.prompt.md`
2. When executed from a base repository, the prompt clones agent-templates to a temporary folder
3. Prompt compares `.github` folder contents between agent-templates and base repository
4. Files that do NOT exist in base repo are copied automatically without prompting
5. Files that exist in BOTH repos are listed for user review (not copied immediately)
6. User is presented with three options for conflicting files:
   - "Overwrite all files"
   - "Overwrite no files"
   - "Attempt to merge, prioritizing agent-templates versions"
7. If user selects merge, each file is compared side-by-side with the intended output shown
8. After file operations complete, the temporary clone folder is removed
9. Prompt execution completes without leaving temporary artifacts

---

## 4) Implementation Design

### Current State
- agent-templates repository contains `.github/agents/`, `.github/prompts/`, `.github/instructions/`, `.github/workflows/`, and `.github/ISSUE_TEMPLATE/` directories
- No existing synchronization mechanism exists
- Other repositories must manually copy files to adopt agent-templates patterns

### Proposed Changes
Create a new prompt file that orchestrates a multi-step workflow:
1. **Acquire Template Phase:** Prefer GitHub MCP APIs to fetch repository contents; if unavailable, request user approval to use `git clone` for a temporary local copy of agent-templates.
2. **Discovery Phase:** List all files in agent-templates `.github/` directory
3. **Comparison Phase:** For each file, check if it exists in base repo `.github/` directory
4. **Auto-Copy Phase:** Copy non-conflicting files (not in base repo) immediately
5. **Conflict Resolution Phase:** Present conflicting files to user with options
6. **Execution Phase:** Execute user's selected strategy (overwrite all/none/merge)
7. **Cleanup Phase:** Remove temporary working directory

### Data Model / Schema
No database or schema changes; file-system operations only.

**File Manifest Structure (in-memory):**
```typescript
{
  filePath: string;           // Relative path from .github/
  existsInBase: boolean;      // True if file exists in base repo
  existsInTemplate: boolean;  // True if file exists in agent-templates
  status: 'auto-copy' | 'conflict' | 'skip';
  contentHash?: string;       // Optional: for quick comparison
}
```

### APIs & Contracts
No new APIs. Uses existing MCP tools:
- `mcp_desktop-command_*` for file operations
- `list_dir` for directory traversal
- `read_file` for content comparison
- `create_file` / `replace_string_in_file` for writing files

### File-Level Change List

#### New Files
**(New)** `.github/prompts/sync-agent-templates.prompt.md`: Main prompt file with 7-phase workflow for syncing agent-templates to target repository

**(New)** `.github/prompts/test-data/sync-scenarios.json`: Test data provider with parameterized scenarios for sync operations (clean repo, partial overlap, full conflict)

**(New)** `.github/prompts/__tests__/sync-agent-templates.test.md`: Unit test suite with 9 test cases covering clone, discovery, categorization, merge, cleanup, and error handling

#### Modified Files
**(Updated)** `README.md`: Add "Syncing Agent Templates" section with usage instructions for new prompt

**(Updated)** `TICKET_FLOW.md`: Reference sync prompt in setup/prerequisites section

### Feature Flags
None required (prompt execution is user-initiated and explicit).

### Config
No environment variables or configuration files. All paths derived from execution context.

### External Dependencies
- Git CLI (for cloning agent-templates)
- MCP Desktop Commander server (file operations)
- GitHub MCP server (optional: for fetching latest commit info)

### Backward Compatibility Strategy
N/A - this is a new capability. No existing functionality to preserve.

### Observability
- User-facing progress updates at each phase
- File operation summaries (X files copied, Y skipped, Z merged)
- Error messages for failed operations with remediation guidance

### Security & Privacy
- No sensitive data handling
- Read-only operations on agent-templates repository
- Write operations only to user's base repository after confirmation
- Temporary clone uses system temp directory with unique timestamp suffix
- Cleanup ensures no credentials or sensitive data left behind

### Alternatives Considered
1. **Git submodule approach:** Rejected due to complexity and poor UX for non-Git experts
2. **Manual copy instructions:** Rejected as error-prone and not automatable
3. **NPM package distribution:** Rejected as prompts are not JavaScript artifacts
4. **Symlinks:** Rejected due to cross-platform compatibility issues

---

### Step-by-Step Implementation Plan (TDD)

#### Phase 1: RED (Test First)

##### 1.1 Create Test Data Providers
**(New)** `.github/prompts/test-data/sync-scenarios.json`:
```json
{
  "scenarios": [
    {
      "name": "clean-repo-all-new-files",
      "baseRepoFiles": [],
      "templateFiles": [".github/prompts/plan-ticket.prompt.md"],
      "expectedAutoCopy": 1,
      "expectedConflicts": 0
    },
    {
      "name": "partial-overlap",
      "baseRepoFiles": [".github/prompts/plan-ticket.prompt.md"],
      "templateFiles": [".github/prompts/plan-ticket.prompt.md", ".github/prompts/work-ticket.prompt.md"],
      "expectedAutoCopy": 1,
      "expectedConflicts": 1
    }
  ]
}
```

##### 1.2 Create Unit Tests (FAIL initially)
**(New)** `.github/prompts/__tests__/sync-agent-templates.test.md`:
Test cases:
- `test_clone_creates_temp_directory` - Verify temp clone path created with timestamp
- `test_discover_github_folder_contents` - Verify all .github files enumerated
- `test_categorize_files_auto_copy_vs_conflict` - Verify correct categorization
- `test_merge_strategy_prioritizes_template` - Verify merge produces expected output
- `test_cleanup_removes_temp_folder` - Verify no artifacts remain
- `test_user_confirmation_required_for_conflicts` - Verify no writes without approval
- `test_overwrite_all_replaces_existing_files` - Verify full replacement
- `test_overwrite_none_preserves_base_files` - Verify no changes to base
- `test_invalid_repo_path_fails_gracefully` - Verify error handling

Run tests → expect FAIL (no implementation exists).

#### Phase 2: GREEN (Implementation)

##### 2.1 Create Prompt File Structure
**(New)** `.github/prompts/sync-agent-templates.prompt.md`:
Sections:
- Header with mode enforcement reference
- Tool requirements reference
- Goal statement
- Input parameters (none required; executes in current workspace)
- Phase 0: Workspace validation
- Phase 1: Clone agent-templates
- Phase 2: Discover .github contents
- Phase 3: Categorize files
- Phase 4: Auto-copy non-conflicting files
- Phase 5: Present conflicts to user
- Phase 6: Execute user selection
- Phase 7: Cleanup temp directory
- Working rules & conventions

##### 2.2 Implement Acquire Logic
In `sync-agent-templates.prompt.md` Phase 1:
```markdown
### Phase 1: Acquire Agent Templates
1.1 Generate temp path: `/tmp/agent-templates-${timestamp}`
1.2 Prefer GitHub MCP API to fetch repository contents into `<temp_path>`
1.3 If MCP repo fetch is unavailable, request user approval and use `git clone https://github.com/dougis-org/agent-templates.git <temp_path>`
1.4 Verify acquisition success (check for `.github` directory)
1.5 On failure: abort with error message and cleanup
```

##### 2.3 Implement Discovery Logic
Phase 2:
```markdown
### Phase 2: Discover .github Contents
2.1 List all files in `<temp_path>/.github/**` (recursive)
2.2 Exclude: `.git`, `.gitignore`, binary files
2.3 Build manifest: [{filePath, existsInTemplate: true}]
2.4 For each file, check if exists in `<base_repo>/.github/<filePath>`
2.5 Update manifest: set `existsInBase` flag
```

##### 2.4 Implement Categorization Logic
Phase 3:
```markdown
### Phase 3: Categorize Files
3.1 For each file in manifest:
   - If existsInBase == false: status = 'auto-copy'
   - If existsInBase == true: status = 'conflict'
3.2 Count auto-copy vs conflict files
3.3 Report to user: "Found X new files, Y conflicts"
```

##### 2.5 Implement Auto-Copy Logic
Phase 4:
```markdown
### Phase 4: Auto-Copy Non-Conflicting Files
4.1 For each file where status == 'auto-copy':
   - Read content from `<temp_path>/.github/<filePath>`
   - Create directory structure in `<base_repo>/.github/` if needed
   - Write file to `<base_repo>/.github/<filePath>`
4.2 Report: "Copied X files successfully"
4.3 List copied files for user confirmation
```

##### 2.6 Implement Conflict Resolution UI
Phase 5:
```markdown
### Phase 5: Present Conflicts to User
5.1 List all files where status == 'conflict':
   - Show file path
   - Show last modified date in both repos (optional)
5.2 Present options:
   - [1] Overwrite all files (replace base with template)
   - [2] Overwrite no files (keep base versions)
   - [3] Attempt to merge (prioritize template, show diff for each)
5.3 Await user input (1, 2, or 3)
```

##### 2.7 Implement Execution Strategies
Phase 6:
```markdown
### Phase 6: Execute User Selection
6.1 If option == 1 (Overwrite all):
   - For each conflict file: read from template, write to base
6.2 If option == 2 (Overwrite no files):
   - Skip all conflict files, report "No changes made to existing files"
6.3 If option == 3 (Merge):
   - For each conflict file:
     - Read both versions
     - Show side-by-side comparison
     - Show proposed merged output (template sections take priority)
     - Ask user: "Accept this merge? (y/n)"
     - If yes: write merged version to base
     - If no: keep base version unchanged
   - Report merge results
```

##### 2.8 Implement Cleanup
Phase 7:
```markdown
### Phase 7: Cleanup Temporary Clone
7.1 Verify temp path still exists
7.2 Remove directory recursively: `rm -rf <temp_path>`
7.3 Verify deletion succeeded
7.4 Report: "Cleanup complete. Temporary files removed."
```

Run tests → expect PASS.

#### Phase 3: Refactor (Quality)

##### 3.1 Extract Reusable Patterns
- Search for existing file comparison utilities and test-data providers across the repo; no reusable utilities found outside prompts.
- If future prompts introduce similar workflows, extract common file operations to `.github/prompts/includes/file-operations.md`.

##### 3.2 Simplify Complex Sections
- Ensure each phase has <20 lines of instructions
- Break down merge logic if it exceeds readability threshold
- Add inline examples for user-facing prompts

##### 3.3 Pre-PR Duplication & Complexity Review
**Duplication checks:**
- Compare with existing prompts for common patterns (clone, file operations, user confirmation)
- No duplication found (this is a unique workflow)

**Complexity checks:**
- Merge logic is most complex (Phase 6.3); acceptable given user interaction requirement
- No cyclomatic complexity issues (linear workflow with clear branches)

**Static analysis:**
- Run markdownlint on new prompt file
- Apply formatting (`prettier` if configured)

##### 3.4 Documentation Updates
**(Updated)** `README.md`:
Add section:
```markdown
### Syncing Agent Templates
To sync the latest prompts and agents from this repository into another project:
1. Navigate to your target repository
2. Run: `sync-agent-templates` prompt
3. Follow the interactive prompts to resolve conflicts
```

**(Updated)** `TICKET_FLOW.md`:
Add reference to sync prompt in "Setup & Prerequisites" section (if exists, otherwise create).

---

## 5) Test Plan & Pre-Commit Quality Review

### Parameterized Test Strategy
All sync scenarios use data provider: `.github/prompts/test-data/sync-scenarios.json`
- Scenarios: clean repo, partial overlap, full conflict, invalid repo path
- Data provider specifies: base repo file list, template file list, expected outcomes
- Reserve simple tests for: single smoke test (end-to-end happy path), cleanup verification

### Test Coverage by Category

#### Happy Paths
**Parameterized source:** `sync-scenarios.json` (scenarios: clean-repo-all-new-files, partial-overlap)
- Verify files copied when no conflicts
- Verify user presented with options when conflicts exist
- Verify successful cleanup after completion

#### Edge/Error Cases
**Parameterized source:** `sync-scenarios.json` (scenarios: network-failure, permission-denied, invalid-merge)
- Clone fails due to network error → error message with retry guidance
- Permission denied on write → error message with remediation steps
- Merge produces invalid markdown → user shown diff, can reject merge

#### Regression
**Simple test with justification:** N/A (new feature, no historical bugs)

#### Contract/API
**Simple test with justification:** Validate MCP tool usage (file operations, directory listing) - architectural validation

#### Performance
**Simple test with justification:** Measure time for 50-file sync operation → expect <30 seconds

#### Security/Privacy
**Simple tests:**
- Verify no credentials stored in temp directory after cleanup
- Verify temp path uses unique timestamp (no collision risk)
- Verify read-only operations on agent-templates (no modifications to template repo)

#### Manual QA Checklist
1. [ ] Run prompt in clean repository (no .github folder) → verify all files copied
2. [ ] Run prompt in repository with existing .github folder → verify conflict detection
3. [ ] Select "Overwrite all" → verify base files replaced with template versions
4. [ ] Select "Overwrite no files" → verify no changes to base files
5. [ ] Select "Merge" option → verify user shown diffs and can approve/reject each
6. [ ] Abort prompt mid-execution → verify cleanup still runs
7. [ ] Test on Windows, Linux, Mac → verify cross-platform compatibility
8. [ ] Verify temp directory removed after successful completion
9. [ ] Verify temp directory removed after error/abort

---

### Pre-Commit Quality Review (Required)
1. **Duplication review:** Ensure no copy-paste logic across phases; consolidate repeated prompt steps if found.
2. **Complexity reduction:** Simplify merge logic to clear, single-responsibility sub-steps.
3. **Dead code removal:** Remove unused steps or optional flows not invoked.
4. **Static analysis:** Run markdownlint on new/updated prompt docs and resolve all findings.

---

## 6) Risk & Rollout

### Risks & Mitigations

| Risk | Severity | Likelihood | Mitigation | Fallback |
|------|----------|-----------|------------|----------|
| **Repository acquisition fails** (network, auth) | High | Medium | Check connectivity; provide retry steps; request user approval before fallback | User manually provides local path |
| **Merge logic produces invalid output** | High | Low | Show proposed merge before writing; require confirmation | User manually merges files; overwrite option bypasses merge |
| **Temp directory not cleaned up** | Medium | Low | Ensure cleanup on success and error; use try-finally pattern in logic | User manually deletes temp directory if notified of path |
| **Large .github folder** (slow operations) | Low | Low | Report progress during discovery and copy phases | Acceptable delay <1 min |
| **Permission errors** (read/write) | Medium | Medium | Check permissions before attempting operations; provide clear error messages | User manually copies files with elevated permissions |
| **Cross-platform path issues** | Low | Low | Use platform-agnostic path separators; test on Windows/Linux/Mac | Document known issues; users can manually adjust paths |

### Rollout & Monitoring Plan

### Feature Flags
None required. Prompt execution is explicit user action.

### Deployment Steps
1. Merge PR with new prompt file to main branch
2. Document in release notes / CHANGELOG
3. Announce to users via README update and (optional) GitHub Discussion
4. Monitor GitHub issues for user feedback on sync experience

### Dashboards & Key Metrics
No runtime metrics (prompt is client-side execution). Success measured by:
- User adoption (mentions in issues/discussions)
- Bug reports (filed issues related to sync prompt)
- Feedback quality (positive vs negative comments)

### Alerts
None required (no server-side component).

### Success Metrics / KPIs
- At least 3 repositories adopt agent-templates via sync prompt within 2 weeks
- Zero critical bugs reported within 1 week of release
- Positive user feedback (≥80% satisfaction in informal polling)

### Rollback Procedure
If prompt causes issues:
1. Revert commit that added `.github/prompts/sync-agent-templates.prompt.md`
2. Push revert to main branch
3. Notify users via GitHub issue: "Sync prompt temporarily disabled; manual copy instructions provided"
4. Fix bugs in separate branch
5. Re-release with updated version

Commands are executed via MCP tooling or GitHub UI; avoid shell commands.

---

## 7) Observability

1. User-facing progress updates for each phase (Acquire, Discover, Compare, Auto-copy, Conflict Resolution, Execute, Cleanup).
2. File operation summary: X files copied, Y conflicts skipped/merged, Z errors.
3. Error messaging with remediation guidance and explicit cleanup confirmation.

---

## 8) Effort & Dependencies

### Effort
**Medium (M)** - Estimated 4-6 hours total
- Prompt design & documentation: 2 hours
- Implementation & testing: 2 hours
- Manual QA & edge case handling: 1-2 hours

Justification:
- Moderate complexity due to multi-phase workflow
- User interaction adds time for UX design
- File operations are well-understood (not novel)
- No backend/API integration required

### Dependencies
- GitHub MCP server (for issue context; optional for repository content retrieval)
- MCP Desktop Commander server (file operations)
- Optional: Git CLI if user approves fallback acquisition

---

## 9) Open Questions / Assumptions

### Assumptions
1. The prompt will be a standalone `.github/prompts/sync-agent-templates.prompt.md` file.
2. Temporary clone location will be in system temp directory (e.g., `/tmp/agent-templates-<timestamp>`).
3. User has read access to agent-templates repository (public or authenticated).
4. Merge strategy uses template-priority comparison and explicit per-file confirmation.
5. File comparison is content-based (not timestamp-based).
6. MCP tools are used for file operations; shell file operations are forbidden.
7. User confirmation required before any destructive operations.
8. Cleanup happens automatically after successful completion or on error.

### Open Questions
None blocking. If merge conflicts are complex, user can manually resolve after initial attempt.

---

## 10) Related Tickets

- GitHub Issue [#3](https://github.com/dougis-org/agent-templates/issues/3)

---

## 11) Decomposition (if applicable)

Single deliverable recommended.
- Scope is a single prompt + documentation updates in this repo.
- All ACs map to a single workflow with shared data flow and no separate subsystems.
- Effort is <1 day of work; no parallelization needed.

---

## Appendix: Handoff Package

---

## 10) Handoff Package

### Links
- **GitHub Issue:** [#3](https://github.com/dougis-org/agent-templates/issues/3)
- **Branch:** `feature/3-sync-agent-templates-prompt`
- **Plan File:** `docs/plan/tickets/3-plan.md`

### Key Commands
**Build/Test:**
```bash
# No build step required (prompt file only)
# Manual testing via prompt execution
```

**Run Prompt:**
```bash
# From target repository:
# Invoke sync-agent-templates mode
```

**Validate:**
```bash
# Lint prompt file
npx markdownlint .github/prompts/sync-agent-templates.prompt.md

# Check for broken links (if applicable)
npx markdown-link-check .github/prompts/sync-agent-templates.prompt.md
```

### Known Gotchas / Watchpoints
1. **Temp directory cleanup:** Ensure cleanup runs even if user aborts mid-execution
2. **Merge conflicts:** Complex merge scenarios may require manual intervention; user should be prepared
3. **Cross-platform paths:** Test on Windows to ensure path separators handled correctly
4. **Git authentication:** If agent-templates is private (unlikely), user needs auth setup
5. **Large file handling:** If .github folder has large binaries, copy may be slow; add progress indicators

---

## Traceability Map

| Criterion # | Requirement | Milestone | Task(s) | Flag(s) | Test(s) |
|-------------|-------------|-----------|---------|---------|---------|
| 1 | Prompt file exists at `.github/prompts/sync-agent-templates.prompt.md` | N/A | Create prompt file with 7-phase workflow | None | `test_prompt_file_exists` |
| 2 | Clone agent-templates to temporary folder | N/A | Implement Phase 1: Clone logic | None | `test_clone_creates_temp_directory` |
| 3 | Compare `.github` folder contents | N/A | Implement Phase 2: Discovery logic | None | `test_discover_github_folder_contents` |
| 4 | Auto-copy non-conflicting files | N/A | Implement Phase 4: Auto-copy logic | None | `test_auto_copy_new_files` |
| 5 | List conflicting files for user review | N/A | Implement Phase 5: Conflict resolution UI | None | `test_categorize_files_auto_copy_vs_conflict` |
| 6 | Present three options (overwrite all/none/merge) | N/A | Implement Phase 5: User option prompts | None | `test_user_confirmation_required_for_conflicts` |
| 7 | Execute merge with side-by-side comparison | N/A | Implement Phase 6.3: Merge strategy | None | `test_merge_strategy_prioritizes_template` |
| 8 | Remove temporary clone folder | N/A | Implement Phase 7: Cleanup logic | None | `test_cleanup_removes_temp_folder` |
| 9 | No temporary artifacts remain after execution | N/A | Implement Phase 7: Verify deletion | None | `test_no_artifacts_after_cleanup` |

---

**Plan Complete**
