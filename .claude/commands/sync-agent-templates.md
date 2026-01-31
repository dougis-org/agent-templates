# Sync Agent Templates

Synchronize `.github` folder contents from the agent-templates repository into the current repository.

## Input

$ARGUMENTS

## Instructions

1. Read and follow the full workflow in `.github/prompts/sync-agent-templates.prompt.md`
2. Use the tool mapping in `CLAUDE.md` to translate MCP tool references to Claude Code equivalents
3. Execute all 8 phases as defined in the prompt file:
   - Phase 0: Workspace validation
   - Phase 1: Clone agent-templates to temp directory
   - Phase 2: Discover template files
   - Phase 3: Categorize as new or conflicting
   - Phase 4: Auto-copy new files
   - Phase 5: Present conflicts for user resolution
   - Phase 6: Execute chosen resolution strategy
   - Phase 7: Cleanup temporary clone

## Key Behavioral Rules (from prompt file)

- Never overwrite or delete base repo files without user approval (except auto-copy of new files)
- Always cleanup temporary directory on success or error
- Provide clear progress updates after each phase
- Default to "overwrite none" if user does not respond to conflict resolution
- See `.github/prompts/__tests__/sync-agent-templates.test.md` for test scenarios
