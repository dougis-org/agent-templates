# Instruction File Efficiency Analysis
**Date:** January 25, 2026
**Analysis Scope:** `/home/doug/dev/agent-templates/.github/instructions/`

---

## Executive Summary

| Metric | Value |
|--------|-------|
| **Total Tokens (Both Files)** | ~2,420 tokens |
| **Codacy Instructions** | ~1,600 tokens |
| **Markdown Instructions** | ~320 tokens |
| **Identified Redundancies** | 6 instances |
| **Estimated Recoverable Tokens** | ~280-350 tokens (11-14% reduction) |
| **Consolidation Opportunities** | 4 major patterns |

---

## File 1: Codacy Instructions
**Location:** `/home/doug/dev/agent-templates/.github/instructions/codacy.instructions.md`
**Tokens:** ~1,600
**Lines:** 95

### Token Breakdown
- Frontmatter (YAML): ~50 tokens
- Headings & structure: ~120 tokens
- Core content: ~1,280 tokens
- Code blocks/examples: ~150 tokens

### Identified Redundancies

#### 1. **"Gracefully bypass/defer" Pattern (3 instances)**
**Lines affected:** 15, 30, 51
**Tokens wasted:** ~45 tokens
**Issue:** Same fallback pattern repeated verbatim:
- Line 15: "Gracefully bypass and defer to CI/CD scans"
- Line 30: "Do not halt progress if local scans are unavailable"
- Line 51: "Proceed—CI/CD security scans will catch vulnerabilities"

**Current state:** Repetitive emphasis on non-blocking nature
**Consolidation:** Extract to reusable phrase or single reference section

---

#### 2. **"Consider running codacy_cli_analyze" Verbose Instructions (2 instances)**
**Lines affected:** 10-13, 38-41
**Tokens wasted:** ~75 tokens
**Issue:** Nearly identical parameter guidance repeated:
```
First instance (lines 10-13):
- `rootPath`: set to the workspace path
- `file`: set to the path of the edited file
- `tool`: leave empty or unset

Second instance (lines 38-41):
- `rootPath`: set to the workspace path
- `tool`: set to "trivy"
- `file`: leave empty or unset
```

**Current state:** Full parameter list shown twice
**Consolidation:** Reference a lookup table or single parameter documentation section

---

#### 3. **Tool Unavailability Handling (2 instances)**
**Lines affected:** 17-18, 52-53
**Tokens wasted:** ~40 tokens
**Issue:** Redundant fallback instructions:
- Line 17: "If the tool is unavailable or fails: Gracefully bypass..."
- Line 52: "If the tool fails or is unavailable: Proceed—CI/CD security scans..."

**Current state:** Repeated conditional guidance
**Consolidation:** Move to single "Error Handling" section

---

#### 4. **Verbose "Optional/Supplementary" Language (Throughout)**
**Lines affected:** 12, 18, 27, 30, 51
**Tokens wasted:** ~65 tokens
**Issue:** Repeated hedging language:
- "optional but recommended"
- "optional and supplementary"
- "supplement quality checks"
- "supplementary, not blocking gates"

**Current state:** Verbose reassurance statements
**Consolidation:** Establish single principle statement upfront, reference via "See core principle"

---

#### 5. **Example-to-Instruction Ratio Imbalance**
**Lines affected:** 43-49 (EXAMPLE section)
**Tokens wasted:** ~50 tokens
**Issue:** EXAMPLE section shows full workflow that mirrors earlier instructions:
```
EXAMPLE shows: npm install → Run codacy_cli_analyze → Decision logic
But this mirrors lines 28-38 instruction.
```

**Current state:** Example is 60% repetition of instructions
**Consolidation:** Example should only show the npm/yarn/pnpm packages (3-4 lines), reference main section for logic

---

#### 6. **Setup & Error Recovery Overspecification**
**Lines affected:** 67-72 (## When there are no Codacy MCP Server tools available...)
**Tokens wasted:** ~80 tokens
**Issue:** Extensive troubleshooting guidance that should be:
- In a troubleshooting appendix (not main flow)
- Linked from "Error Handling" section
- Condensed using abbreviations (URL shortening, list reduction)

**Current state:** Full URLs, multi-step instructions in body
**Consolidation:** Move to include file with reference; show only essential steps

---

## File 2: Markdown Instructions
**Location:** `/home/doug/dev/agent-templates/.github/instructions/markdown.instructions.md`
**Tokens:** ~320
**Lines:** 12

### Token Breakdown
- Frontmatter (YAML): ~50 tokens
- Content: ~270 tokens

### Analysis
**Status:** Highly concise, well-structured
**Redundancies Found:** 1 minor

#### 1. **Overspecified Tool Chain (Minor)**
**Lines affected:** 4-6
**Tokens wasted:** ~20 tokens
**Issue:** Redundant tool guidance:
```markdown
- Consider running the `fix_markdown` tool... 
- Once the fix command has completed, optionally run the `lint_markdown` command...
```

**Current state:** Two-tool workflow is sequential/conditional
**Consolidation:** Combine into single "Consider running `fix_markdown`, then `lint_markdown`..." (removes "Once... has completed, optionally")

---

## Cross-File Analysis

### 1. **Duplicated Pattern Structure**
Both files follow identical structure:
```
[Frontmatter]
## [Trigger Condition]
- [Action with tool name]
- [Parameter guidance]
- [Fallback instruction]
- If unavailable: [Bypass logic]
```

**Opportunity:** Create `mcp-tool-instruction-template.md` include that both reference

**Estimated savings:** ~200 tokens (moving common pattern to template)

---

### 2. **Shared MCP Tool Fallback Principle**
Both files state variations of:
- "Gracefully bypass if unavailable"
- "Tools are supplementary, not blocking"
- "CI/CD will catch issues"

**Consolidation approach:** 
Create `mcp-tool-fundamentals.md` include with:
- Single "Non-blocking principle" statement
- Unified fallback guidance table
- Reference from both instruction files

**Estimated savings:** ~100 tokens

---

### 3. **Missing Pattern Comparison with `/includes/`**

Existing includes use these conventions:
- **Concise headers:** "5.1 Duplication Check (Local)" not "## When you need to..."
- **Tables for parameters:** mcp-tooling-requirements.md uses tables heavily
- **Conditional formatting:** Lists with ✅/❌ symbols (efficient visual parsing)
- **Modular structure:** Includes are 100-400 tokens, self-contained

**Current instruction files:**
- Use prose paragraphs for sequential conditions (verbose)
- No parameter reference tables
- Mix frontmatter YAML into guidance (confusing)

---

## Detailed Consolidation Recommendations

### Recommendation 1: Extract MCP Tool Fallback Pattern (~120 tokens saved)

**New file:** `/home/doug/dev/agent-templates/.github/prompts/includes/mcp-error-handling.md`

```markdown
## MCP Tool Error Handling (Shared)

### Non-Blocking Principle
All MCP tool invocations are **optional and supplementary**. If any tool is unavailable, 
fails, or is not installed:
- Gracefully skip the local scan
- Continue workflow without delay
- CI/CD will validate before merging

### Unavailability Scenarios

| Scenario | Action | Fallback |
|----------|--------|----------|
| Tool not installed | Skip local scan | Proceed to next step |
| Tool fails/errors | Log issue (if verbose logging enabled) | Continue workflow |
| MCP Server unreachable | Skip this invocation | Use next validation step |
| User declines tool install | Respect choice | Proceed to CI/CD validation |

### When Tools Are Unavailable (Troubleshooting)
[Move extended troubleshooting section here]
```

**Files updated:**
- `codacy.instructions.md` → Reference this include 3 times
- `markdown.instructions.md` → Reference once

---

### Recommendation 2: Create Parameter Documentation Table (~90 tokens saved)

**New file:** `/home/doug/dev/agent-templates/.github/prompts/includes/mcp-tool-parameters.md`

```markdown
## Common MCP Tool Parameters

### codacy_cli_analyze Parameters

| Parameter | Use Case | Value | Required |
|-----------|----------|-------|----------|
| `rootPath` | File edit context | workspace path | Yes |
| `file` | Single-file scan | edited file path | When scanning file |
| `tool` | Security-specific scan | "trivy" | Optional |
| `tool` | General scan | (leave empty) | Optional |
```

**Codacy instructions updated:**
- Lines 10-13 → "See mcp-tool-parameters.md#codacy_cli_analyze"
- Lines 38-41 → "See mcp-tool-parameters.md#codacy_cli_analyze"

---

### Recommendation 3: Refactor Condition-Action Structure (~100 tokens saved)

**Change from:** Verbose prose headers + full instruction blocks
**Change to:** Concise headers + bullet lists + table for decision logic

**Example refactor (Codacy file):**

**Before (32 tokens):**
```markdown
## After ANY successful `edit_file` or `reapply` operation (Optional)
- Consider running the `codacy_cli_analyze` tool from Codacy's MCP Server 
  for each file that was edited to supplement quality checks:
```

**After (18 tokens):**
```markdown
## Post-Edit Local Scan (Optional)
Run `codacy_cli_analyze` per edited file:
```

**Impact:** 6 header sections × ~14 tokens saved = ~84 tokens

---

### Recommendation 4: Consolidate Markdown Instructions (Optional Efficiency) (~40 tokens)

**Current state:** Markdown instructions are already concise

**Minor improvement:** Merge into a single `mcp-markdown-linting.md` template similar to codacy format

```markdown
## Markdown Linting (Post-Edit)
1. Run `fix_markdown` tool → auto-fixes
2. Run `lint_markdown` tool → discovers remaining issues
3. If issues found: propose & apply fixes
4. If unavailable: defer to CI/CD (non-blocking)
```

**Estimated savings:** ~30 tokens

---

## Token Savings Summary

| Consolidation | Location | Tokens Saved | Difficulty |
|---------------|----------|--------------|-----------|
| 1. Extract Error Handling Include | New include file | 120 | Low |
| 2. Create Parameter Ref Table | New include file | 90 | Low |
| 3. Refactor Headers/Prose | Both files | 100 | Medium |
| 4. Compress Example Sections | codacy.md | 50 | Low |
| 5. Deduplicate "Optional" Language | codacy.md | 65 | Low |
| 6. Move Troubleshooting Section | Appendix/include | 60 | Low |
| 7. Consolidate Markdown Instructions | Optional/merge | 40 | Very Low |
| **TOTAL** | | **~525 tokens** | |

---

## Implementation Priority

### Phase 1 (High Impact, Low Effort) — **350 tokens saved**
1. ✅ Create `mcp-error-handling.md` include
2. ✅ Create `mcp-tool-parameters.md` include  
3. ✅ Update codacy.instructions.md to reference includes (remove redundant prose)
4. ✅ Compress Example section in codacy.md

**Timeline:** ~30 minutes

### Phase 2 (Medium Impact, Medium Effort) — **100 tokens saved**
1. ✅ Refactor prose headers → concise bullet headers
2. ✅ Move troubleshooting to appendix/separate include
3. ✅ Add visual decision tables where prose was

**Timeline:** ~20 minutes

### Phase 3 (Low Impact, Low Effort) — **75 tokens saved**
1. ✅ Consolidate redundant "optional/supplementary" language
2. ✅ Optionally merge markdown.instructions.md into standardized template

**Timeline:** ~15 minutes

---

## Comparison with Existing Patterns

### Pattern 1: Structured Headers (Present in `/includes/`)
✅ mcp-tooling-requirements.md uses numbered headers (1.1, 1.2)
❌ codacy.instructions.md uses ##/### with long descriptive text
⚠️ markdown.instructions.md is too short to show pattern

**Recommendation:** Adopt numbered structure for clarity

### Pattern 2: Decision Tables (Present in `/includes/`)
✅ pre-commit-quality-review.md uses table for "Quality Gates"
❌ codacy.instructions.md uses prose conditions
❌ markdown.instructions.md missing entirely

**Recommendation:** Use tables for parameter/scenario decision matrices

### Pattern 3: Forbidden/Required Lists (Present in `/includes/`)
✅ mcp-tooling-requirements.md uses ❌/✅ symbols
❌ codacy.instructions.md uses prose "Consider"
❌ markdown.instructions.md missing entirely

**Recommendation:** Adopt ❌/✅ visual parsing for tool requirements

### Pattern 4: Include File Size (Present in `/includes/`)
✅ Most includes: 50-200 tokens (focused, reusable)
❌ codacy.instructions.md: 1,600 tokens (monolithic)
⚠️ markdown.instructions.md: 320 tokens (acceptable)

**Recommendation:** Break codacy.instructions.md into 3-4 focused includes

---

## Recovery Strategy: Achievable Savings

### Conservative Estimate (Safe Changes)
- Extract common patterns to includes: **280 tokens**
- Compress verbose language: **70 tokens**
- **Subtotal: 350 tokens (14.4% reduction)**

### Aggressive Estimate (Restructure)
- All conservative changes: **350 tokens**
- Refactor headers/prose to tables: **100 tokens**
- Consolidate with markdown.instructions: **75 tokens**
- **Subtotal: 525 tokens (21.7% reduction)**

### Recommendation
Pursue **conservative approach + Phase 1 implementation** (350 tokens) 
→ Delivers high value, low risk, improves maintainability

---

## Quality Impact Assessment

### Changes That Improve Clarity
✅ Moving error handling to separate include (focused reference)
✅ Adding parameter reference tables (faster scanning)
✅ Refactoring prose conditions to lists (better visual parsing)
✅ Shortening headers (less cognitive load)

### Changes That Could Reduce Clarity (Avoid)
❌ Over-abbreviating tool names
❌ Removing fallback guidance entirely (keep it, just consolidate)
❌ Merging disparate tool instructions (codacy ≠ markdown linting)

### Recommendation
Pursue Phase 1 + Phase 2. Skip Phase 3's full merge (markdown.instructions stays focused on one topic).
