# Claude Code Project Instructions

## Mandatory Guidelines

Read and follow `AGENTS.md` for all work. Its MUST rules override any conflicting guidance.

Key mandates from `AGENTS.md`:

- **Quality First:** All work must meet acceptance criteria, include tests, and pass linting
- **TDD Enforcement:** RED → GREEN → REFACTOR. Tests before production logic
- **Accuracy:** Do not guess. Verify with repo sources or explicit user input
- **No Silent Assumptions:** Document assumptions and open questions explicitly

## Workflow

Follow the end-to-end ticket workflow defined in `TICKET_FLOW.md`:

```
find-next-ticket → plan-ticket → analyze-ticket → work-ticket → review-ticket-work → cut-pr → review-pr
```

For larger initiatives spanning multiple teams, follow `SDLC_INITIATIVE_PLANNING.md`.

## Available Commands

These slash commands map to the GitHub Copilot prompts and agents defined in `.github/prompts/` and `.github/agents/`. Each command references those files directly to avoid duplication.

| Command | Copilot Prompt | Copilot Agent | Purpose |
|---------|---------------|---------------|---------|
| `/project:find-next-ticket` | `.github/prompts/find-next-ticket.prompt.md` | `.github/agents/find-next-ticket.agent.md` | Select next executable GitHub issue |
| `/project:plan-ticket` | `.github/prompts/plan-ticket.prompt.md` | `.github/agents/plan-ticket.agent.md` | Create TDD implementation plan |
| `/project:analyze-ticket` | `.github/prompts/analyze-ticket.prompt.md` | - | Validate a ticket plan for gaps |
| `/project:work-ticket` | `.github/prompts/work-ticket.prompt.md` | `.github/agents/work-ticket.agent.md` | Execute plan with TDD + quality gates |
| `/project:review-ticket-work` | `.github/prompts/review-ticket-work.prompt.md` | - | Review local changes before push |
| `/project:review-pr` | `.github/prompts/review-pr.prompt.md` | `.github/agents/code-review.agent.md` | Review a pull request |
| `/project:cut-pr` | `.github/prompts/cut-pr.prompt.md` | - | Create PR from current branch |
| `/project:sync-agent-templates` | `.github/prompts/sync-agent-templates.prompt.md` | - | Sync .github/ from agent-templates repo |

## Tool Mapping: Copilot MCP → Claude Code

The Copilot prompts and agents reference MCP tools. When executing those workflows in Claude Code, use these native equivalents:

### File Operations

| Copilot MCP Tool | Claude Code Equivalent |
|---|---|
| `desktop-commander/read_file` | `Read` tool |
| `desktop-commander/edit_block` | `Edit` tool |
| `desktop-commander/write_file` | `Write` tool |
| `desktop-commander/create_directory` | `Bash` (`mkdir -p`) |
| `desktop-commander/list_directory` | `Glob` tool or `Bash` (`ls`) |
| `desktop-commander/move_file` | `Bash` (`mv`) |
| `github/get_file_contents` | `Read` tool (local) or `Bash` (`gh api`) |
| `github/create_or_update_file` | `Edit`/`Write` tool + `Bash` (`git commit`) |

### Search & Discovery

| Copilot MCP Tool | Claude Code Equivalent |
|---|---|
| `desktop-commander/start_search` (files) | `Glob` tool |
| `desktop-commander/start_search` (content) | `Grep` tool |
| `deepcontext/search_codebase` | `Task` tool (Explore agent) |

### GitHub & Issue Tracking

| Copilot MCP Tool | Claude Code Equivalent |
|---|---|
| `github/create_pull_request` | `Bash` (`gh pr create`) |
| `github/create_branch` | `Bash` (`git switch -c`) |
| `github/list_branches` | `Bash` (`git branch` / `gh api`) |
| `github/list_commits` | `Bash` (`git log`) |
| `github/push_files` | `Bash` (`git add` + `git commit` + `git push`) |
| `github/search_code` | `Grep` tool or `Bash` (`gh search code`) |
| `github/search_issues` | `Bash` (`gh search issues`) |
| `gh-issues/issue_read` | `Bash` (`gh issue view`) |
| `gh-issues/issue_write` | `Bash` (`gh issue edit` / `gh issue create`) |
| `gh-issues/add_issue_comment` | `Bash` (`gh issue comment`) |
| `gh-issues/search_issues` | `Bash` (`gh search issues`) |
| `gh-actions/*` | `Bash` (`gh run list` / `gh run view`) |

### Testing & Quality

| Copilot MCP Tool | Claude Code Equivalent |
|---|---|
| `run_in_terminal` | `Bash` tool |
| `execute/runTests` | `Bash` (`npm test` or project test command) |
| `markdownlint/*` | `Bash` (`npx markdownlint-cli2`) |
| `codacy-mcp-server/*` | `Bash` (Codacy CLI or CI results) |
| `playwright/*` | `Bash` (`npx playwright test`) |

### Reasoning & Planning

| Copilot MCP Tool | Claude Code Equivalent |
|---|---|
| `sequentialthinking/*` | Built-in reasoning (native to Claude) |
| `agent` (sub-agent delegation) | `Task` tool |

## Shared Includes

These files contain shared guidance referenced by multiple prompts and agents. Read them when the prompt or agent file directs you to:

- `.github/prompts/includes/mcp-tooling-requirements.md` — Tool usage mandate (use tool mapping above for Claude Code equivalents)
- `.github/prompts/includes/ticket-detection.md` — GitHub/Jira platform auto-detection
- `.github/prompts/includes/branch-commit-guidance.md` — Branch naming and commit conventions
- `.github/prompts/includes/signed-commits-requirement.md` — GPG/SSH commit signing toggle
- `.github/prompts/includes/mode-enforcement.md` — Mode requirement template
- `.github/prompts/includes/ac-validation-template.md` — Acceptance criteria table template
- `.github/prompts/includes/issue-severity-scale.md` — Severity classification scale
- `.github/prompts/includes/pre-commit-quality-review.md` — Pre-commit quality checklist
- `.github/agents/includes/tdd-enforcement-cycle.md` — RED/GREEN/REFACTOR cycle

## Instructions

- `.github/instructions/codacy.instructions.md` — Codacy code quality tool configuration
- `.github/instructions/markdown.instructions.md` — Markdown linting rules

## Adapting MCP Tool References

When a Copilot prompt says "use MCP tool X", substitute the Claude Code equivalent from the mapping table above. The workflow logic, phases, quality gates, and behavioral guardrails remain identical — only the tool invocations change.

When a prompt references `.github/prompts/includes/mcp-tooling-requirements.md`, apply its principles (prefer structured tools over raw shell commands) using Claude Code's native tools: `Read`, `Edit`, `Write`, `Glob`, `Grep`, and `Bash` for git/gh CLI operations.
