# Work Ticket

Execute an approved implementation plan for a GitHub issue using strict TDD, quality gates, and issue synchronization.

## Input

Ticket identifier: $ARGUMENTS

## Instructions

1. Read and follow the full workflow in `.github/prompts/work-ticket.prompt.md`
2. Apply behavioral guardrails from `.github/agents/work-ticket.agent.md`
3. Read shared includes as referenced by the prompt:
   - `.github/prompts/includes/ticket-detection.md` for platform detection
   - `.github/prompts/includes/branch-commit-guidance.md` for git conventions
   - `.github/prompts/includes/signed-commits-requirement.md` for commit signing
   - `.github/agents/includes/tdd-enforcement-cycle.md` for TDD cycle
4. Use the tool mapping in `CLAUDE.md` to translate MCP tool references to Claude Code equivalents
5. Execute all phases (0 through 9) as defined in the prompt file

## Key Behavioral Rules (from agent file)

- TDD is non-negotiable: RED → GREEN → REFACTOR
- No production logic without corresponding tests
- Quality gates must all pass before completion
- No scope expansion beyond the plan
- No automatic git commits without user confirmation
