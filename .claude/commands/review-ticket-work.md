# Review Ticket Work

Review local changes against a GitHub issue's acceptance criteria before pushing to remote.

## Input

Ticket identifier: $ARGUMENTS

## Instructions

1. Read and follow the full workflow in `.github/prompts/review-ticket-work.prompt.md`
2. Apply behavioral guardrails from `.github/agents/code-review.agent.md`
3. Read shared includes as referenced by the prompt:
   - `.github/prompts/includes/ticket-detection.md` for platform detection
4. Use the tool mapping in `CLAUDE.md` to translate MCP tool references to Claude Code equivalents
5. Execute all 7 steps as defined in the prompt file
6. Produce the review summary using the template in the prompt file

## Key Behavioral Rules (from prompt file)

- Map every acceptance criterion to implementation evidence
- Check for duplication, complexity, and business logic clarity
- Run all quality gates (tests, linting, coverage)
- Flag for human review if ACs cannot be verified or security issues detected
- Output recommendation: Ready to push / Ready with improvements / Blocked
