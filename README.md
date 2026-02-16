# ASQ Automation Plugin for Claude Code

Automate the full ASQ (Approval Request / Specialist SA) lifecycle in Claude Code: pull data from Salesforce, research across Glean/Slack/web, design architectures, sync status, and track time.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- Salesforce CLI (`sf`) authenticated
- MCP servers configured: Glean, Slack, JIRA, Confluence
- FE plugins: `fe-salesforce-tools`, `fe-google-tools` (for calendar/gmail/docs)
- A notes directory (see [Notes Directory Setup](#notes-directory-setup) below)

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

- **Markdown files = working notes**: Customer notes are plain markdown in `02-customers/` (works with Obsidian, VS Code, or any editor)
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

## Notes Directory Setup

This plugin stores customer notes as **plain markdown files** — no special tools required. You can browse them with Obsidian, VS Code, or any text editor.

### Required folder structure

Create this directory structure wherever you like (default: `~/workspace/databricks_knowledge_vault/`):

```
your-notes-directory/
├── 02-customers/              # Customer engagement notes (created by /asq-intake)
├── 30-templates/
│   └── customer-note.md       # Customer note template
└── 50-logs/                   # Weekly activity & time logs
    └── 2026/                  # Auto-created per year
```

**Quick setup (copy-paste):**

```bash
# Set your preferred path (change this to wherever you want)
NOTES_DIR=~/workspace/databricks_knowledge_vault

mkdir -p "$NOTES_DIR"/{02-customers,30-templates,50-logs}
```

### Customer note template

Create `30-templates/customer-note.md` with the base template. You can copy the one from this repo or create your own — the plugin will use it as a starting point and add ASQ-specific fields automatically.

A minimal template:

```markdown
---
type: customer
customer_name: {{title}}
status: status/active
created: {{date:YYYY-MM-DD}}
last_updated: {{date:YYYY-MM-DD}}
---

# {{title}}

## ASQ Summary

## Key Stakeholders
| Name | Role | Notes |
|------|------|-------|

## Engagement History
| Date | Type | Summary | Action Items |
|------|------|---------|--------------|

## Open Items
- [ ]
```

### Using a different directory

If your notes directory is **not** at `~/workspace/databricks_knowledge_vault/`, update the path in `skills/asq-lifecycle/resources/vault-paths.md` and `skills/asq-lifecycle/SKILL.md`. Both files reference the vault root path.

## Customization

### Salesforce Fields

The `SKILL.md` file contains the full SF field reference. Update if your org has custom fields.
