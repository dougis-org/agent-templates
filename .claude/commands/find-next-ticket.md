# Find Next Ticket

Identify the single next executable GitHub issue based on dependency ordering, priority, and milestone.

## Input

$ARGUMENTS

## Instructions

1. Read and follow the full workflow in `.github/prompts/find-next-ticket.prompt.md`
2. Apply behavioral guardrails from `.github/agents/find-next-ticket.agent.md`
3. Use the tool mapping in `CLAUDE.md` to translate MCP tool references to Claude Code equivalents
4. Use `gh` CLI for all GitHub issue queries (replaces MCP `gh-issues/*` tools)
5. Execute the selection algorithm as defined in the prompt file

## Key Behavioral Rules (from agent file)

- Read-only mode: NEVER modify issues, add comments, or transition statuses
- Output ONLY `#<number>` if a startable issue exists, nothing else
- If no startable issue: output concise blocker explanation
- Treat only explicit "blocks" link relationships as dependencies
- A predecessor is satisfied ONLY if state == closed
