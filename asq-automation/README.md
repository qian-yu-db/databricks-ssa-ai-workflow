# ASQ Automation Plugin for Claude Code

Automate the full ASQ (Approval Request / Specialist SA) lifecycle in Claude Code: pull data from Salesforce, research across Glean/Slack/web, design architectures, sync status, and track time.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- Salesforce CLI (`sf`) authenticated
- MCP servers configured: Glean, Slack, JIRA, Confluence
- FE plugins: `fe-salesforce-tools`, `fe-google-tools` (for calendar/gmail/docs)

## Installation

### 1. Enable the plugin

Add to your `~/.claude/settings.json` under `enabledPlugins`:

```json
{
  "enabledPlugins": {
    "asq-automation@qian-yu-db/databricks-ssa-ai-workflow": true
  }
}
```

### 2. Set up your notes directory

```bash
# Set your preferred path (default: ~/asq-notes)
NOTES_DIR=~/asq-notes

mkdir -p "$NOTES_DIR"/{customers,templates,logs}
```

### 3. Create the customer note template

Create `templates/customer-note.md` in your notes directory:

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

### Business Context
- **Industry**:
- **Company Size**:
- **Business Challenges**:

### Technical Environment
- **Cloud Provider**:
- **Databricks Workspace(s)**:
- **Key Technologies**:

### Project Scope
- **Use Case**:
- **Success Criteria**:
- **Timeline**:

## Key Stakeholders
| Name | Role | Notes |
|------|------|-------|

## Engagement History
| Date | Type | Summary | Action Items |
|------|------|---------|--------------|

## Open Items
- [ ]
```

### 4. (Optional) Customize paths

If you use a different directory structure, add this to your `~/.claude/CLAUDE.md`:

```markdown
## ASQ Notes Configuration
- ASQ_NOTES_ROOT: ~/my/custom/path
- CUSTOMERS_DIR: customers/
- TEMPLATES_DIR: templates/
- LOGS_DIR: logs/
```

Then restart Claude Code.

## Commands

| Command | Description | Model |
|---------|-------------|-------|
| `/asq-intake AR-XXXXXX` | Pull ASQ from Salesforce, create customer note | sonnet |
| `/asq-research AR-XXXXXX` | Autonomous research across Glean, Slack, and web | opus |
| `/asq-architecture AR-XXXXXX` | Design solution architecture with mermaid diagram | opus |
| `/asq-status AR-XXXXXX [--days N]` | Gather activity from Slack/Calendar/Gmail, sync to SF (default: 7 days) | sonnet |
| `/asq-timetrack` | Weekly time summary across all active ASQs | sonnet |

## Typical Workflow

```
1. /asq-intake AR-000106904        # Create customer note from Salesforce
2. /asq-research AR-000106904      # Research the engagement topic
3. /asq-architecture AR-000106904  # Design the solution
4. /asq-status AR-000106904        # Status update (last 7 days)
   /asq-status AR-000106904 --days 14  # Or look back further
5. /asq-timetrack                  # End-of-week time sync
```

## How It Works

- **Markdown files = working notes**: Customer notes are plain markdown in `customers/` (works with any editor)
- **Salesforce = system of record**: Hours and status sync back to SF (always with confirmation)
- **Research is autonomous**: The `/asq-research` agent searches Glean, Slack, and web in parallel
- **Everything is additive**: Commands add sections to notes, never overwrite existing content
- **Paths are configurable**: Override defaults via `CLAUDE.md` to fit your directory structure

## Directory Structure

```
your-notes-dir/              # Default: ~/asq-notes
├── customers/               # Customer engagement notes (created by /asq-intake)
├── templates/
│   └── customer-note.md     # Customer note template
└── logs/                    # Weekly activity & time logs
    └── 2026/                # Auto-created per year
        └── 2026-W07.md
```

## Plugin Structure

```
.claude-plugin/plugin.json          # Plugin manifest
commands/
  asq-intake.md                     # SF -> customer note
  asq-research.md                   # Launches research agent
  asq-architecture.md               # Solution design + mermaid
  asq-status.md                     # Activity search + SF sync
  asq-timetrack.md                  # Weekly time summary
agents/
  asq-researcher.md                 # Autonomous multi-source research agent
skills/
  asq-lifecycle/
    SKILL.md                        # Always-loaded ASQ context
    resources/vault-paths.md        # Path configuration reference
```

## Customization

### Salesforce Fields

The `SKILL.md` file contains the full SF field reference. Update if your org has custom fields.

### Path Overrides

See `skills/asq-lifecycle/resources/vault-paths.md` for the full path configuration reference and override instructions.
