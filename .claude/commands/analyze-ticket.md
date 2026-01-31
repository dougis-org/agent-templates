# Analyze Ticket

Validate a ticket-level plan file to ensure completeness, coverage, and proper structure.

## Input

Ticket identifier: $ARGUMENTS

## Instructions

1. Read and follow the full workflow in `.github/prompts/analyze-ticket.prompt.md`
2. Follow agent guidelines in `AGENTS.md`
3. Read shared includes as referenced by the prompt:
   - `.github/prompts/includes/ticket-detection.md` for platform detection
4. Use the tool mapping in `CLAUDE.md` to translate MCP tool references to Claude Code equivalents
5. Execute all 13 steps as defined in the prompt file
6. Produce a structured analysis report (read-only; do not modify files without explicit user approval)

## Key Behavioral Rules (from prompt file)

- STRICTLY READ-ONLY: Do not modify any files without explicit user approval
- NEVER hallucinate missing sections; state "section missing" if absent
- Make findings deterministic with stable IDs
- Severity: CRITICAL > HIGH > MEDIUM > LOW
- Only edit plan files (`docs/plan/tickets/*-plan.md`) if user explicitly requests it
