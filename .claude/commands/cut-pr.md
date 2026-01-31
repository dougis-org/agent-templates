# Cut PR

Create a pull request from the current branch to the default branch with automatic title and description generation.

## Input

$ARGUMENTS

## Instructions

1. Read and follow the full workflow in `.github/prompts/cut-pr.prompt.md`
2. Use the tool mapping in `CLAUDE.md` to translate MCP tool references to Claude Code equivalents
3. Use `gh pr create` for PR creation (replaces MCP `github/create_pull_request`)
4. Execute all 4 phases as defined in the prompt file:
   - Phase 0: Pre-flight checks (uncommitted changes, unpushed commits, merge conflicts)
   - Phase 1: Gather repository context (PR template, ticket auto-detection, change analysis)
   - Phase 2: Generate PR content (semantic title, structured description)
   - Phase 3: Create the PR via `gh pr create`

## Key Behavioral Rules (from prompt file)

- Abort immediately if pre-flight checks fail, with clear error message
- Auto-detect ticket ID from branch name if not provided
- Use semantic commit-style PR titles: `<type>(<scope>): #<TICKET_ID> <summary>`
- Use repository PR template if one exists; otherwise use best-practices structure
- No user confirmation required for PR creation (auto-create after validation)
