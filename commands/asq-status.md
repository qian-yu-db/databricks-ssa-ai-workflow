---
description: Update ASQ status from Slack/Calendar/Gmail activity and sync to Salesforce
argument-hint: "AR-XXXXXX"
model: sonnet
---

# /asq-status - Status Update & Salesforce Sync

Gather recent activity from Slack, Calendar, and Gmail for an ASQ engagement, update the customer note, and sync status to Salesforce.

## Usage

```
/asq-status AR-000106904
```

## Workflow

### 1. Parse AR ID

Extract the AR ID from the argument. Accept formats: `AR-000106904` or `000106904` (auto-prefix AR-).

### 2. Find and Read Customer Note

Use Glob to find `customer-*-AR-{ID}.md` in `~/workspace/databricks_knowledge_vault/02-customers/`.

If not found, tell the user:
> No customer note found for {AR ID}. Run `/asq-intake {AR ID}` first.

Read the note and extract:
- Customer name and account name
- ASQ ID and support type
- Current engagement history entries
- Current open items
- Last updated date from frontmatter

### 3. Search Recent Activity (Last 2 Weeks)

Calculate the date range: today minus 14 days to today.

#### 3a. Search Slack

```
slack_read_api_call("search.messages", {
    "query": "{account name} OR {AR ID}",
    "count": 30
})
```

Also search for the customer name if different from account name. Look for:
- Discussion threads about this engagement
- Meeting follow-ups
- Technical questions or answers
- Decision points and agreements

#### 3b. Search Google Calendar

Use the `fe-google-tools:google-calendar` skill to find meetings:
- Search for meetings with customer name in title or attendees
- Look at the last 2 weeks
- Extract meeting titles, dates, and attendees

#### 3c. Search Gmail

Use the `fe-google-tools:gmail` skill to search:
- Emails to/from customer contacts
- Emails mentioning the account name or AR ID
- Look at the last 2 weeks
- Extract key correspondence summaries

#### 3d. Search Google Docs (Optional)

Use the `fe-google-tools:google-docs` skill if available:
- Find recently modified docs mentioning the customer
- Look for shared architecture docs, meeting notes, etc.

### 4. Update Customer Note

#### 4a. Update Engagement History

Add new entries to the Engagement History table with discovered activity. Use the Edit tool to add rows to the table:

```markdown
| {date} | {type: Call/Email/Slack/Workshop} | {summary of activity} | {action items identified} |
```

Sort newest first (reverse chronological).

#### 4b. Update Open Items

Review action items from discovered activity and update the Open Items checklist:
- Mark completed items as `- [x]`
- Add new action items discovered from meetings/emails/Slack
- Keep uncompleted items

#### 4c. Update Frontmatter

Update `last_updated` in frontmatter to today's date.

### 5. Update Weekly Log

Determine current ISO week. Create or update the weekly log at:
`~/workspace/databricks_knowledge_vault/50-logs/{YYYY}/{YYYY}-W{XX}.md`

Create the directory if needed. Append an entry:

```markdown
### {AR ID} - {Customer Name}
- Activities: {count} Slack threads, {count} meetings, {count} emails
- Key updates: {brief summary}
- Open items: {count active items}
```

### 6. Draft Salesforce Status Update

Compose a concise status update for the `Request_Status_Notes__c` field based on:
- Recent activities summarized
- Current engagement phase
- Key outcomes or decisions
- Next steps

Format as a dated entry that can be appended:
```
{YYYY-MM-DD}: {Concise status summary. Key activities, outcomes, and next steps.}
```

### 7. Confirm Salesforce Update

**CRITICAL: Never write to Salesforce without explicit user confirmation.**

Show the user:
```
Planned Salesforce update for {AR ID}:

Field: Request_Status_Notes__c
Value: "{status text}"

This will update the ASQ status notes in Salesforce.
```

Use AskUserQuestion to confirm:
- Option 1: "Update Salesforce" - proceed with the update
- Option 2: "Edit first" - let user modify the text
- Option 3: "Skip SF update" - only update local notes

### 8. Update Salesforce (If Confirmed)

First query for the record ID:
```bash
sf data query -q "SELECT Id FROM ApprovalRequest__c WHERE Name = '{AR ID}'" --json
```

Then update:
```bash
sf data update record -s ApprovalRequest__c -i {RECORD_ID} -v "Request_Status_Notes__c='{status text}'"
```

### 9. Show Completion Summary

Display:
- Status updated for {AR ID} ({customer name})
- Activity found: {X} Slack threads, {X} meetings, {X} emails
- Engagement history: {X} new entries added
- Open items: {X} active, {X} completed
- Weekly log updated: {log path}
- Salesforce: {updated/skipped}
