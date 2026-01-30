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
