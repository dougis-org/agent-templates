# agent-templates

A central location to keep agent definitions and prompts as well as workflows we want to use often.

## Quick Start

### Syncing Agent Templates

To synchronize the latest prompts, agents, and workflows from this repository into another project:

1. **Navigate to your target repository** (the repository where you want to copy agent-templates files)

2. **Run the sync prompt** by opening [.github/prompts/sync-agent-templates.prompt.md](.github/prompts/sync-agent-templates.prompt.md) in your prompt runner (for example, GitHub Copilot Chat) and following the instructions in that file.

3. **Choose your conflict resolution strategy:**
   - **Overwrite all files**: Replace existing `.github` files with agent-templates versions (recommended if agent-templates is canonical)
   - **Overwrite no files**: Keep existing `.github` files unchanged (recommended if you have customizations)
   - **Merge with template priority**: Review each conflicting file side-by-side and approve/reject merges individually

4. **Verify and commit:**
   ```bash
   git diff .github/
   git add .github/
   git commit -m "chore: sync agent-templates"
   git push
   ```

### Features

- **Automatic New File Detection:** Copies any new `.github` files not present in your repository
- **Conflict Detection:** Identifies files that exist in both repositories
- **Multiple Resolution Strategies:** Choose how to handle conflicts
- **Side-by-Side Merge Preview:** Review and approve merges before writing
- **Automatic Cleanup:** Temporary files removed after sync completes
- **Cross-Platform Compatible:** Works on Linux, macOS, and Windows

### What Gets Synced

The `.github` directory typically contains:
- `.github/prompts/` - Reusable prompts for agents and workflows
- `.github/agents/` - Agent definitions and capabilities
- `.github/workflows/` - GitHub Actions workflows
- `.github/instructions/` - Process and coding standards
- `.github/ISSUE_TEMPLATE/` - Issue templates

For details about the sync workflow, see [.github/prompts/sync-agent-templates.prompt.md](.github/prompts/sync-agent-templates.prompt.md).

---

## Orchestrator Agent

The **Orchestrator Agent** automates the complete ticket workflow (TICKET_FLOW.md) by delegating to specialized sub-agents, enforcing quality gates, and pausing at human review checkpoints for approval.

### Key Features

- **Automated Workflow Coordination:** Executes all phases of TICKET_FLOW.md in sequence
- **Quality Gate Enforcement:** Refuses to advance workflow if sub-agents report quality failures
- **Human Checkpoints:** Pauses at two defined stages (post-plan and post-PR) for explicit user approval
- **Intelligent Feedback Routing:** Routes user feedback to appropriate sub-agents based on content analysis
- **State Tracking & Resumption:** Persists workflow state to enable resumption from any phase
- **Multi-Platform Support:** Works with both GitHub issues and Jira tickets

### Getting Started

1. **Run the orchestrator prompt:**
   ```
   Open .github/prompts/orchestrate-ticket.prompt.md in your prompt runner (GitHub Copilot Chat, etc.)
   ```

2. **Provide ticket ID:**
   ```
   #7                 (GitHub issue)
   PROJ-123           (Jira ticket)
   ```

3. **Review at checkpoints:**
   - **Plan Checkpoint:** Review plan, risks, and decomposition recommendation. Approve or reject with feedback.
   - **PR Checkpoint:** Review PR details, test results, and quality gates. Approve for merge or reject with feedback.

4. **Track progress:**
   - View workflow state: `show state`
   - Review prior phase outputs: `review plan`, `review pr`, `review history`
   - Resume interrupted workflow: `resume workflow` (auto-loads saved state)

### Workflow Phases

```
1. DISCOVERY        → Load ticket from GitHub/Jira
2. PLANNING         → Invoke plan-ticket sub-agent
3. ANALYSIS         → Invoke analyze-ticket sub-agent
4. PLAN_CHECKPOINT  → Human approval (required)
5. IMPLEMENTATION   → Invoke work-ticket sub-agent
6. LOCAL_REVIEW     → Invoke review-ticket-work sub-agent
7. PR_CREATION      → Invoke cut-pr sub-agent
8. PR_CHECKPOINT    → Human approval (required)
9. CODE_REVIEW      → Invoke review-pr sub-agent (final review & merge)
10. DONE            → Workflow complete
```

### Documentation

- **Agent Definition:** [.github/agents/ticket-orchestrator.agent.md](.github/agents/ticket-orchestrator.agent.md)
- **Prompt File:** [.github/prompts/orchestrate-ticket.prompt.md](.github/prompts/orchestrate-ticket.prompt.md)
- **State Management:** [.github/agents/includes/orchestrator-state-management.md](.github/agents/includes/orchestrator-state-management.md)
- **Checkpoint Protocol:** [.github/agents/includes/human-checkpoint-protocol.md](.github/agents/includes/human-checkpoint-protocol.md)
- **Test Suite:** [.github/prompts/__tests__/orchestrate-ticket.test.md](.github/prompts/__tests__/orchestrate-ticket.test.md)
- **Test Data:** [.github/prompts/test-data/orchestrator-scenarios.json](.github/prompts/test-data/orchestrator-scenarios.json)

### Related

See [TICKET_FLOW.md](./TICKET_FLOW.md) for the complete workflow definition and phase descriptions.
