# Review Quality Core (Shared)

Use these checks whenever reviewing code or changes against a ticket.

## Duplication Assessment
- Intra-file duplication: repeated patterns within the same file
- Cross-file duplication: similar logic across changed files
- Existing codebase conflicts: new code that duplicates existing utilities or patterns
- Recommendation: extract shared utilities or reuse existing ones where possible

## Complexity Evaluation
- Methods are focused and single-purpose
- Cyclomatic complexity stays within acceptable thresholds (flag if > 10)
- Method length stays reasonable (flag if > 20-30 lines)
- Nesting depth stays shallow (flag if > 3 levels)
- Classes keep dependencies minimal (flag if > 5 dependencies)

## Business Logic Clarity
- Intent is clear from code and comments
- Domain language is consistent with the ticket
- Assumptions are surfaced explicitly
- Business rules trace back to requirements
