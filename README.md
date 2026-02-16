# ASQ Automation Plugin for Claude Code

Automate the full ASQ (Approval Request / Specialist SA) lifecycle in Claude Code: pull data from Salesforce, research across Glean/Slack/web, design architectures, sync status, and track time.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- Salesforce CLI (`sf`) authenticated
- MCP servers configured: Glean, Slack, JIRA, Confluence
- FE plugins: `fe-salesforce-tools`, `fe-google-tools` (for calendar/gmail/docs)
- Obsidian vault at `~/workspace/databricks_knowledge_vault/`

## Installation

Add to your `~/.claude/settings.json` under `enabledPlugins`:

```json
{
  "enabledPlugins": {
    "asq-automation@qian-yu-db/databricks-ssa-ai-workflow": true
  }
}
```

Then restart Claude Code. The plugin will be fetched from GitHub automatically.

## Commands

| Command | Description | Model |
|---------|-------------|-------|
| `/asq-intake AR-XXXXXX` | Pull ASQ from Salesforce, create Obsidian customer note | sonnet |
| `/asq-research AR-XXXXXX` | Autonomous research across Glean, Slack, and web | opus |
| `/asq-architecture AR-XXXXXX` | Design solution architecture with mermaid diagram | opus |
| `/asq-status AR-XXXXXX` | Gather activity from Slack/Calendar/Gmail, sync to SF | sonnet |
| `/asq-timetrack` | Weekly time summary across all active ASQs | sonnet |

## Typical Workflow

```
1. /asq-intake AR-000106904        # Create customer note from Salesforce
2. /asq-research AR-000106904      # Research the engagement topic
3. /asq-architecture AR-000106904  # Design the solution
4. /asq-status AR-000106904        # Weekly status updates
5. /asq-timetrack                  # End-of-week time sync
```

## How It Works

- **Obsidian = working notes**: Customer notes live in your vault at `02-customers/`
- **Salesforce = system of record**: Hours and status sync back to SF (always with confirmation)
- **Research is autonomous**: The `/asq-research` agent searches Glean, Slack, and web in parallel
- **Everything is additive**: Commands add sections to notes, never overwrite existing content

## File Structure

```
.claude-plugin/plugin.json          # Plugin manifest
commands/
  asq-intake.md                     # SF -> Obsidian note
  asq-research.md                   # Launches research agent
  asq-architecture.md               # Solution design + mermaid
  asq-status.md                     # Activity search + SF sync
  asq-timetrack.md                  # Weekly time summary
agents/
  asq-researcher.md                 # Autonomous multi-source research agent
skills/
  asq-lifecycle/
    SKILL.md                        # Always-loaded ASQ context
    resources/vault-paths.md        # Vault path constants
```

## Customization

### Vault Paths

Edit `skills/asq-lifecycle/resources/vault-paths.md` to change vault locations if your Obsidian vault is at a different path.

### Salesforce Fields

The `SKILL.md` file contains the full SF field reference. Update if your org has custom fields.
