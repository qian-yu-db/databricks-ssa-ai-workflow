---
name: asq-lifecycle
description: "Use this skill for ANY mention of ASQ, AR tickets, approval requests, or specialist SA engagements. Triggers on: 'ASQ', 'AR-', 'approval request', 'specialist request', 'asq-intake', 'asq-research', 'asq-architecture', 'asq-status', 'asq-timetrack'. Provides context about ASQ lifecycle management, Salesforce fields, note paths, and integration patterns."
---

# ASQ Lifecycle Management

Core context for managing ASQ (Approval Request / Specialist SA) engagements across Salesforce, local notes, and communication tools.

## Notes Directory Structure

**IMPORTANT: Check the user's `CLAUDE.md` for an `## ASQ Notes Configuration` section.** If present, use those paths instead of the defaults below.

### Default Paths

| Path | Purpose |
|------|---------|
| `~/asq-notes/` | Notes root (override via CLAUDE.md `ASQ_NOTES_ROOT`) |
| `customers/` | Customer engagement notes (override via `CUSTOMERS_DIR`) |
| `templates/customer-note.md` | Customer note template (override via `TEMPLATES_DIR`) |
| `logs/YYYY/YYYY-WXX.md` | Weekly activity logs (override via `LOGS_DIR`) |

### Path Resolution

Before using any path, check `~/.claude/CLAUDE.md` for overrides:
1. Look for `## ASQ Notes Configuration` section
2. If found, use the specified `ASQ_NOTES_ROOT`, `CUSTOMERS_DIR`, `TEMPLATES_DIR`, `LOGS_DIR`
3. If not found, use defaults: `~/asq-notes/`, `customers/`, `templates/`, `logs/`

### Note Naming Convention

`customer-{CODE}-AR-{ID}.md` where:
- `{CODE}` = uppercase 2-4 letter abbreviation derived from account name (e.g., Travelers → TRV, Acme Corp → ACME)
- `{ID}` = AR number (e.g., 000106904)
- Always check existing files in the customers directory for reuse of existing company codes before creating new ones

## Salesforce ApprovalRequest__c Key Fields

### Read Fields (for queries)
| Field | Description |
|-------|-------------|
| `Name` | AR auto-number (e.g., AR-000106904) |
| `Status__c` | Current status (New, In Progress, Complete, Assigned, etc.) |
| `Request_Type__c` | Type (e.g., `Specialist SA (SSA) Request`) |
| `Support_Type__c` | Engagement type (e.g., `Production Architecture Review & Design`) |
| `Request_Description__c` | Detailed request description |
| `Situation_Details__c` | Customer context/initiatives |
| `Account__r.Name` | Account name (relationship field) |
| `Start_Date__c` | Requested start date |
| `End_Date__c` | Target end date |
| `Approved_Start_Date__c` | Approved start date |
| `Approved_End_Date__c` | Approved end date |
| `Estimated_Duration__c` | Estimated total effort in days |
| `Actual_Effort_in_Days__c` | Cumulative days of effort used (updated by /asq-timetrack) |
| `Requestor__r.Name` | Who requested |
| `Resource__r.Name` | Assigned SSA |
| `Urgency__c` | Priority level |
| `EngagementType__c` | Transactional or Long-term |
| `Approved_Investment_Per_Period__c` | Days per period |
| `Approved_Frequency__c` | Weekly/Monthly/Every Other Week |

### Write Fields (for updates)
| Field | Description |
|-------|-------------|
| `Request_Status_Notes__c` | Free-text status notes (updated by /asq-status) |
| `Actual_Effort_in_Days__c` | Cumulative effort in days (updated by /asq-timetrack). New value = current + this week's days. ALWAYS confirm with user before updating. |

### Common Queries

```bash
# Get ASQ by AR number
sf data query -q "SELECT Id, Name, Status__c, Request_Type__c, Support_Type__c, Urgency__c, Request_Description__c, Situation_Details__c, Start_Date__c, End_Date__c, Estimated_Duration__c, Actual_Effort_in_Days__c, Account__r.Name, Requestor__r.Name, Resource__r.Name, Approved_Start_Date__c, Approved_End_Date__c, Approved_Investment_Per_Period__c, Approved_Frequency__c, EngagementType__c FROM ApprovalRequest__c WHERE Name = 'AR-XXXXXX'" --json

# Update status notes (ALWAYS confirm with user first)
sf data update record -s ApprovalRequest__c -w "Name='AR-XXXXXX'" -v "Request_Status_Notes__c='Status update text'"

# Update actual effort in days (ALWAYS confirm with user first)
# new_value = current Actual_Effort_in_Days__c + this_week_days
sf data update record -s ApprovalRequest__c -i {SF_RECORD_ID} -v "Actual_Effort_in_Days__c={new_value}"
```

## Extended Frontmatter for Customer Notes

When creating customer notes from ASQ data, extend the standard template frontmatter with:

```yaml
asq_id: AR-XXXXXX
support_type: Production Architecture Review & Design
start_date: YYYY-MM-DD
end_date: YYYY-MM-DD
estimated_days_total: X
actual_effort_days: X
```

## Integration Patterns

### Glean (Internal Knowledge)
- Use `glean_read_api_call("search.query", {...})` for searching internal docs
- Search for: architecture guides, SSA playbooks, similar engagement notes

### Slack (Communication)
- Use `slack_read_api_call("search.messages", {"query": "..."})` for finding discussions
- Search for: account channels, technology discussions, prior SSA conversations

### Google Tools
- Gmail: Search for customer correspondence
- Calendar: Find meetings with customer
- Docs: Find shared documents

## Safety Rules

1. **SF writes require user confirmation** - Never update Salesforce without showing the planned change and getting explicit approval
2. **Local notes are source of truth** - Customer notes live in the notes directory, SF has metadata
3. **Research is additive** - Research findings append to notes, never overwrite existing content
4. **Company codes are stable** - Once a code is assigned to an account, always reuse it
