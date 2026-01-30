# Plan Ticket

Build an execution-ready implementation plan for a GitHub issue using TDD and repo context.

## Input

Ticket identifier: $ARGUMENTS

## Instructions

1. Read and follow the full workflow in `.github/prompts/plan-ticket.prompt.md`
2. Apply behavioral guardrails from `.github/agents/plan-ticket.agent.md`
3. Read shared includes as referenced by the prompt:
   - `.github/prompts/includes/ticket-detection.md` for platform detection
   - `.github/prompts/includes/branch-commit-guidance.md` for git conventions
   - `.github/prompts/includes/signed-commits-requirement.md` for commit signing
   - `.github/agents/includes/tdd-enforcement-cycle.md` for TDD cycle
4. Use the tool mapping in `CLAUDE.md` to translate MCP tool references to Claude Code equivalents
5. Execute all steps (0 through 4) as defined in the prompt file
6. Output all 11 required sections and persist to `docs/plan/tickets/{{TICKET_ID}}-plan.md`

## Key Behavioral Rules (from agent file)

- No production code authored; planning only
- Search for reusable patterns before proposing new utilities
- Cite file paths for every reused pattern
- Flag all blockers for user clarification
- Evaluate scope decomposition for every ticket
- Await user approval before creating sub-issues
