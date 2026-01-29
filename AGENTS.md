# Agent Guidelines

These guidelines apply to all agents and prompts in this repository. They are mandatory unless a higher‑priority instruction explicitly overrides them.

## Core Principles (MUST)

1. **Quality First**: All work must meet acceptance criteria, include tests, and pass linting before completion.
2. **TDD Enforcement**: Follow RED → GREEN → REFACTOR. Write meaningful tests before production logic.
3. **Accuracy & Determinism**: Do not guess. Verify with sources in this repo or explicit user input.
4. **MCP Tooling Required**: Use MCP tools for file operations, search, and GitHub actions. Avoid shell commands for file reads/writes/searches.
5. **No Silent Assumptions**: Explicitly document assumptions and open questions.

## Testing Standards (MUST)

- **All code must be tested.** Prefer integration over mocks; mock only when necessary and justified.
- **Parameterized tests by default** for multiple scenarios, boundary conditions, or edge cases.
- **External data sources** required for parameterized tests (JSON/CSV/providers), with file path specified.
- **Justify simple tests** when not parameterized (smoke test, architecture validation, one‑time setup/teardown).

## Reuse & Duplication Prevention (MUST)

- **Search for existing utilities** before creating new ones: `*Validator`, `*Builder`, `*Factory`, `*TestDataProvider`.
- **Cite reusable utilities** with file paths when planned or used.
- **Justify new utilities** with search evidence if no suitable alternative exists.

## Observability & Rollout (MUST)

- Include metrics/logs and at least one alert or SLO for feature work.
- Provide rollback steps for any migration or risky change.

## MCP Over Shell (MUST)

- Use MCP tools for:
  - File reads/writes/edits
  - Directory listings
  - Searches (file or content)
  - GitHub issue/PR interactions
- Shell commands are only allowed when no MCP tool exists and must be explicitly justified.

## Documentation & Linting (MUST)

- Update relevant docs when behavior changes.
- Resolve all markdownlint and ESLint issues introduced by changes.

## Output Discipline (SHOULD)

- Keep responses short and action‑oriented.
- Report blockers immediately.
- Provide exact file paths when citing utilities or edits.
