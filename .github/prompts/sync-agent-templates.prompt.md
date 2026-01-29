# Sync Agent Templates Prompt

## ⚠️ MODE REQUIREMENT

**This prompt requires the `work-ticket` chatmode to be active.**

If you selected a different chatmode, please:
1. Switch to `.github/chatmodes/work-ticket.chatmode.md`
2. Return to this prompt

The chatmode provides execution guardrails; this prompt provides specific workflow.

**Tool Requirements:**
Refer to `.github/prompts/includes/mcp-tooling-requirements.md` for mandatory MCP tool usage.

---

## Goal

Enable synchronization of `.github` folder contents from the `dougis-org/agent-templates` repository into any base repository, with automatic detection of new files, conflict detection for existing files, and user-controlled resolution strategies (overwrite all, overwrite none, or merge with template priority).

---

## Inputs

**Required:** None (executes in current working repository)

**Optional:**
- `MERGE_STRATEGY`: Override default merge behavior (values: "auto-copy-only", "overwrite-all", "overwrite-none", "merge")
- `TEMPLATE_REPO_URL`: Custom template repo URL (default: `https://github.com/dougis-org/agent-templates.git`)
- `DRY_RUN`: Preview changes without writing (default: false)

---

## Overview

This prompt orchestrates a 7-phase workflow to synchronize `.github` folder contents from agent-templates into a target repository:

1. **Workspace Validation:** Verify working repository is valid and accessible
2. **Acquire:** Clone agent-templates to temporary directory
3. **Discovery:** Enumerate all files in agent-templates `.github/` directory
4. **Categorize:** Classify files as new (auto-copy) or conflicting (exists in base)
5. **Auto-Copy:** Write new files to base repository without user confirmation
6. **Conflict Resolution:** Present conflicting files to user with resolution strategies
7. **Execute:** Perform user-selected strategy (overwrite all/none/merge)
8. **Cleanup:** Remove temporary clone directory

---

## Phase 0: Workspace Validation

### 0.1 Verify Working Repository

Confirm that:
1. Current directory is a Git repository (contains `.git/` directory)
2. `.github/` folder exists or can be created in base repository
3. Sufficient disk space available for temporary clone
4. Write permissions available for base repository

**On failure:**
- Error message: "Not a valid Git repository or .git directory not found"
- Guidance: "Execute this prompt from the root of your target repository"
- Exit gracefully

### 0.2 Detect Repository Context

1. Capture current working directory as `BASE_REPO_ROOT`
2. Detect base repository name from `.git/config` or fallback to directory name
3. Log repository context for user reference

**Output:** "Syncing from agent-templates into: `<BASE_REPO_NAME>`"

### 0.3 Generate Temporary Path

1. Generate timestamp: `TIMESTAMP=$(date +%Y%m%d%H%M%S)`
2. Create temp path in the system's temporary directory: `TEMP_CLONE_PATH="<system_temp_dir>/agent-templates-${TIMESTAMP}"`
3. Verify path uniqueness (ensure no collision with existing directories)

**Output:** "Temporary workspace: `<TEMP_CLONE_PATH>`"

---

## Phase 1: Acquire Agent Templates

### 1.1 Clone Agent Templates Repository

Attempt to clone agent-templates using git:

```
git clone https://github.com/dougis-org/agent-templates.git <TEMP_CLONE_PATH>
```

**On success:**
- Confirm clone completed
- Verify `.github/` directory exists in clone
- Output: "Cloned agent-templates to temporary directory"

**On failure (network error, auth failure):**
1. Show error message with reason
2. Offer user options:
   - [A] Retry clone (useful for transient network errors)
   - [B] Provide local path to pre-cloned agent-templates (for offline scenarios)
   - [C] Abort sync
3. Handle user selection:
   - [A] → Retry (max 2 attempts total)
   - [B] → Request path; validate `.github/` directory exists
   - [C] → Exit gracefully with cleanup

### 1.2 Verify Clone Integrity

1. Check that `.github/` directory exists in `TEMP_CLONE_PATH`
2. List contents of `.github/` to confirm files present
3. If missing: treat as error; offer retry or abort

**Output:** "Verified agent-templates clone integrity"

---

## Phase 2: Discovery

### 2.1 Enumerate Template Files

Recursively list all files in `<TEMP_CLONE_PATH>/.github/`:

1. Walk directory tree starting from `.github/`
2. Capture relative paths (relative to `.github/` directory)
3. Exclude:
   - `.git/` directories (should not be present in `.github/` typically)
   - `.gitignore` files
   - Binary files (e.g., based on file extension like .png, .jpg, .zip, .exe, or file size > 10MB)
4. Include all other files: `.md`, `.json`, `.yml`, `.sh`, etc.

### 2.2 Build Template Manifest

Create in-memory manifest:
```
TEMPLATE_MANIFEST = [
  {
    filePath: ".github/prompts/plan-ticket.prompt.md",
    existsInTemplate: true,
    existsInBase: null,  // To be determined in Phase 3
    status: null,        // To be determined in Phase 3
    contentHash: null    // Optional for quick comparison
  },
  ...
]
```

### 2.3 Report Discovery Results

Output summary:
```
Discovery Summary:
- Template files found: <COUNT>
- Categories:
  - Prompts: <COUNT>
  - Agents: <COUNT>
  - Workflows: <COUNT>
  - Other: <COUNT>
```

**Example output:**
```
Discovery Summary:
- Template files found: 18
- Categories:
  - Prompts: 5
  - Agents: 3
  - Workflows: 4
  - Instructions: 4
  - Other: 2
```

---

## Phase 3: Categorize Files

### 3.1 Check Base Repository for Each File

For each file in `TEMPLATE_MANIFEST`:

1. Check if file exists at `<BASE_REPO_ROOT>/.github/<filePath>`
2. Set `existsInBase` flag (true/false)
3. Update status:
   - If `existsInBase == false`: `status = 'auto-copy'`
   - If `existsInBase == true`: `status = 'conflict'`

### 3.2 Categorize Results

Count and separate:
- `autoCopyFiles = [files where status == 'auto-copy']`
- `conflictFiles = [files where status == 'conflict']`

### 3.3 Report Categorization

Output summary:
```
Categorization Summary:
- New files (will auto-copy): <AUTO_COPY_COUNT>
  <example files>
- Existing files (conflicts): <CONFLICT_COUNT>
  <example files>
```

**Example output:**
```
Categorization Summary:
- New files (will auto-copy): 12
  .github/prompts/work-ticket.prompt.md
  .github/agents/new-agent.md
  .github/workflows/new-workflow.yml
  ... and 9 more

- Existing files (conflicts): 6
  .github/prompts/plan-ticket.prompt.md
  .github/workflows/ci.yml
  ... and 4 more
```

---

## Phase 4: Auto-Copy Non-Conflicting Files

### 4.1 Process Auto-Copy Files

For each file in `autoCopyFiles`:

1. Read file content from `<TEMP_CLONE_PATH>/.github/<filePath>`
2. Determine target path: `<BASE_REPO_ROOT>/.github/<filePath>`
3. Create parent directories as needed (e.g., `.github/prompts/`, `.github/workflows/`)
4. Write file to target path with identical content
5. Record success in manifest

**MCP Tool Usage:**
- Use `list_dir` or similar to verify/create parent directories
- Use `read_file` to read template file
- Use `create_file` to write new file to base repo

### 4.2 Report Auto-Copy Results

Output:
```
Auto-Copy Results:
- Files copied: <SUCCESS_COUNT>
- Files skipped (errors): <ERROR_COUNT>

Copied files:
- .github/prompts/work-ticket.prompt.md ✓
- .github/agents/new-agent.md ✓
- ... and <REMAINING> more

<If errors>
Skipped files (check permissions):
- .github/workflows/error-file.yml ✗ (Permission denied)
- ... and <REMAINING> more
</If errors>
```

### 4.3 Summary

Output confirmation:
```
✓ Auto-copy phase complete: <SUCCESS_COUNT> files copied
```

---

## Phase 5: Conflict Resolution (User Input)

### 5.1 Present Conflicts to User

If `conflictFiles` is empty: Skip to Phase 6 (Cleanup).

If conflicts exist, display:

```
Conflict Resolution Required
=============================
<CONFLICT_COUNT> files exist in both repositories and require your decision.

Conflicting Files:
1. .github/prompts/plan-ticket.prompt.md
   - Base repo version: <size>, last modified: <date>
   - Template version: <size>, last modified: <date>

2. .github/workflows/ci.yml
   - Base repo version: <size>, last modified: <date>
   - Template version: <size>, last modified: <date>

... and <REMAINING> more files

Resolution Options:
[1] Overwrite all files
    → Replace all conflicting files with agent-templates versions
    → Recommended if agent-templates is canonical/newer

[2] Overwrite no files
    → Keep all existing base repo files unchanged
    → Recommended if base repo has customizations

[3] Merge files (with template priority)
    → Review each file side-by-side
    → Choose to accept or reject proposed merge per file
    → Recommended for selective updates

Enter your choice (1, 2, or 3):
```

### 5.2 Await User Input

- Prompt user for response: `1`, `2`, or `3`
- Validate input (numeric, in range)
- If invalid input: re-prompt with error message
- Store choice in `CONFLICT_RESOLUTION_STRATEGY`

**Timeout:** If applicable, default to strategy `2` (Overwrite no files) after 5 minutes

---

## Phase 6: Execute Resolution Strategy

### 6.1 Strategy: Overwrite All (Option 1)

For each file in `conflictFiles`:

1. Read file content from template: `<TEMP_CLONE_PATH>/.github/<filePath>`
2. Write to base repo: `<BASE_REPO_ROOT>/.github/<filePath>` (overwrite)
3. Record action in manifest

**MCP Tool Usage:**
- Use `read_file` to read template version
- Use `replace_string_in_file` or `create_file` to overwrite base repo file

**Output:**
```
Overwrite All Strategy
======================
Processing <CONFLICT_COUNT> files...

.github/prompts/plan-ticket.prompt.md ✓ (overwritten)
.github/workflows/ci.yml ✓ (overwritten)
... and <REMAINING> more

Summary: <SUCCESS_COUNT> files overwritten, <ERROR_COUNT> errors
```

### 6.2 Strategy: Overwrite None (Option 2)

No file operations performed. Skip directly to Phase 7.

**Output:**
```
Overwrite None Strategy
=======================
No changes will be made to existing files.

All <CONFLICT_COUNT> conflicting files remain unchanged in base repo.
```

### 6.3 Strategy: Merge with Template Priority (Option 3)

For each file in `conflictFiles`:

#### 6.3.1 Show Side-by-Side Comparison

```
File: .github/prompts/plan-ticket.prompt.md
==============================================================

BASE REPO VERSION (left):
<First 30 lines or full content if <30 lines>
... (truncated if longer)

TEMPLATE VERSION (right):
<First 30 lines or full content if <30 lines>
... (truncated if longer)

[View full diff? (y/n)]
```

If user selects view full diff:
- Display complete side-by-side comparison with diff markers (← base, → template)
- Use visual indicators for added/removed/modified lines

#### 6.3.2 Propose Merged Output

Generate merged output with logic:
1. **Detect merge conflicts:** Lines that differ between base and template
2. **Apply template priority:** Where conflicts exist, use template version
3. **Preserve base additions:** Lines in base but not in template are kept
4. **Add template additions:** Lines in template but not in base are added
5. **Show merged preview:**

```
PROPOSED MERGED OUTPUT (template priority):
<Merged content>
... (first 30 lines)

Accept this merge? (y/n)
```

#### 6.3.3 User Confirmation per File

- If yes: Write merged version to base repo file
- If no: Keep base repo file unchanged
- Record decision in manifest

**MCP Tool Usage:**
- Use `read_file` to read both versions
- Display diff logic locally (no special tool needed for text comparison)
- Use `replace_string_in_file` to write merged version if approved

#### 6.3.4 Iterate Through All Conflicts

Repeat 6.3.1-6.3.3 for each file in `conflictFiles`.

After each file:
```
Progress: <CURRENT>/<TOTAL> files processed
- Accepted: <COUNT>
- Rejected: <COUNT>
- Skipped: <COUNT>
```

#### 6.3.5 Merge Summary

After all files processed:

```
Merge Strategy Summary
======================
Files processed: <TOTAL>
- Accepted merges: <ACCEPTED>
- Rejected merges (kept base): <REJECTED>
- Skipped: <SKIPPED>

Details:
✓ .github/prompts/plan-ticket.prompt.md (merged)
✗ .github/workflows/ci.yml (kept base version)
... and <REMAINING> more
```

---

## Phase 7: Cleanup Temporary Clone

### 7.1 Verify Temporary Path

Confirm `TEMP_CLONE_PATH` exists and is a directory (safety check before deletion).

### 7.2 Remove Directory Recursively

Delete the temporary clone directory:

```
rm -rf <TEMP_CLONE_PATH>
```

**Alternative (if shell unavailable):**
- Use MCP file operations to recursively delete directory contents

### 7.3 Verify Cleanup

Confirm that `TEMP_CLONE_PATH` no longer exists.

**On failure:**
- Error message: "Cleanup failed: could not remove temporary directory at <TEMP_CLONE_PATH>"
- Guidance: "You may manually delete this directory: `rm -rf <TEMP_CLONE_PATH>`"

### 7.4 Cleanup Summary

Output final status:

```
Cleanup Complete
================
✓ Temporary directory removed: <TEMP_CLONE_PATH>
✓ No temporary artifacts remain

Sync Operation Summary:
- Auto-copied: <COUNT> files
- Overwritten: <COUNT> files (if Overwrite All strategy)
- Merged: <COUNT> files (if Merge strategy)
- Unchanged: <COUNT> files (if Overwrite None strategy)
- Total modified: <TOTAL_MODIFIED> files

Next steps:
1. Review changes in your base repo: git diff
2. Stage changes: git add .github/
3. Commit changes: git commit -m "chore: sync agent-templates"
4. Push to remote: git push
```

---

## Error Handling & Recovery

| Error | Recovery Strategy |
|-------|-------------------|
| Repository not found | Suggest repository URL; verify network access |
| Clone fails (network) | Offer retry or manual path option |
| Clone fails (auth) | Request credentials or suggest `git config` |
| Permission denied (write) | Suggest `chmod` or run with elevated permissions |
| Temp directory exists (collision) | Wait and retry (timestamp should be unique) |
| Disk space insufficient | Check available space; clean up other temp files |
| Merge produces invalid output | Show diff; let user approve/reject |
| User aborts (Ctrl+C) | Cleanup runs; notify user that temp files are being removed |

---

## Working Rules

1. **MCP-First:** Use MCP tools for file operations (list_dir, read_file, create_file, replace_string_in_file). Avoid shell commands except where necessary (e.g., `git clone`, `rm`).
2. **No Destructive Actions Without Confirmation:** Never overwrite or delete base repo files without user approval (except auto-copy of new files).
3. **Temp Directory Hygiene:** Always cleanup `TEMP_CLONE_PATH` on success or error.
4. **Clear Progress Updates:** Report status after each phase.
5. **User Control:** Provide options for conflict resolution; prioritize user choice.
6. **Fail-Fast:** Exit early on validation errors (Phase 0); provide clear guidance.
7. **Documentation:** Output examples and guidance for merge decisions; help users make informed choices.

---

## Testing & Validation

See `.github/prompts/__tests__/sync-agent-templates.test.md` for:
- Unit test specifications (9 test categories, 30+ individual tests)
- Parameterized test data (4 scenarios in `.github/prompts/test-data/sync-scenarios.json`)
- Manual QA checklist

**Key test scenarios:**
- Clean repository (all files auto-copy)
- Partial overlap (mixed auto-copy and conflicts)
- Full conflict (all files conflict)
- Deep nested directories (tests path handling)
- Error scenarios (network failure, permission denied, invalid repo)

Run tests after implementation to verify all acceptance criteria met.

---

## References

- **Issue:** [#3 - Add sync agent-templates prompt](https://github.com/dougis-org/agent-templates/issues/3)
- **Plan:** [docs/plan/tickets/3-plan.md](docs/plan/tickets/3-plan.md)
- **Test Suite:** [.github/prompts/__tests__/sync-agent-templates.test.md](.github/prompts/__tests__/sync-agent-templates.test.md)
- **Test Data:** [.github/prompts/test-data/sync-scenarios.json](.github/prompts/test-data/sync-scenarios.json)
- **Related:** [README.md syncing section](../../../README.md#syncing-agent-templates)

---

## Appendix: Future Enhancements

**Out of scope for v1, but documented for future consideration:**

- Automatic scheduling (periodic sync via GitHub Actions)
- Dependency resolution (sync only specific .github subdirectories)
- Rollback capability (undo last sync with version control)
- Conflict resolution suggestions (AI-based merge proposals)
- Configuration file (sync preferences stored in base repo)
- Two-way sync (push changes back to agent-templates)

---

**End of Prompt**
