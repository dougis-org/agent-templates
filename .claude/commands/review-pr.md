# Review Pull Request

Review a pull request against GitHub issue acceptance criteria with focus on code quality.

## Input

Ticket identifier and PR reference: $ARGUMENTS

## Instructions

1. Read and follow the full workflow in `.github/prompts/review-pr.prompt.md`
2. Apply behavioral guardrails from `.github/agents/code-review.agent.md`
3. Read shared includes as referenced by the prompt:
   - `.github/prompts/includes/ticket-detection.md` for platform detection
4. Use the tool mapping in `CLAUDE.md` to translate MCP tool references to Claude Code equivalents
5. Use `gh pr view` and `gh pr diff` for PR details (replaces MCP `github/pull_request_read`)
6. Execute all 9 steps as defined in the prompt file
7. Produce the review summary using the template in the prompt file

## Key Behavioral Rules (from agent file)

- Focus on code under review, not tangential improvements
- Distinguish "must fix" (blocking) from "nice to have" (suggestion)
- Base feedback on established principles, not personal preference
- Assume positive intent from code authors
- No automatic code modifications (review only)
- Severity levels: Blocking > Warning > Suggestion > Praise
