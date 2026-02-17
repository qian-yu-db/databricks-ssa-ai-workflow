---
description: Weekly time summary and Salesforce days sync for active ASQs
model: sonnet
---

# /asq-timetrack - Time Tracking Summary

Generate a weekly time summary across all active ASQs and sync effort (in days) to Salesforce.

## Usage

```
/asq-timetrack
```

No arguments needed. Automatically finds all active ASQs.

## Workflow

### 0. Resolve Paths

Read `~/.claude/CLAUDE.md` and look for an `## ASQ Notes Configuration` section. If found, use those paths. Otherwise use defaults:
- `ASQ_NOTES_ROOT`: `~/asq-notes`
- `CUSTOMERS_DIR`: `customers/`
- `LOGS_DIR`: `logs/`

### 1. Find Active ASQs

Search for all active customer notes:

1. Use Glob to find all `customer-*-AR-*.md` files in `{ASQ_NOTES_ROOT}/{CUSTOMERS_DIR}`
2. Read each file's frontmatter
3. Filter to only notes where `status: status/active`
4. Extract from each: AR ID (from `asq_id` frontmatter), customer name, company code

If no active ASQs found, inform the user and stop.

### 2. Read Weekly Log

Determine the current ISO week number and year. Check if a weekly log exists at:
`{ASQ_NOTES_ROOT}/{LOGS_DIR}/{YYYY}/{YYYY}-W{XX}.md`

If it exists, read it for any already-logged time entries this week.

### 3. Estimate Days from Activity

For each active ASQ, estimate days spent this week by reviewing:
- Engagement History entries in the customer note dated this week
- Any time entries in the weekly log for this ASQ
- Present estimates to user for adjustment (use day fractions, e.g., 0.5, 1, 1.5, 2)

### 4. Query Salesforce

Query SF for current effort data on all active ASQs:

```bash
sf data query -q "SELECT Id, Name, Actual_Effort_in_Days__c, Estimated_Duration__c, Account__r.Name FROM ApprovalRequest__c WHERE Name IN ('AR-XXX','AR-YYY')" --json
```

Calculate remaining days as: `Estimated_Duration__c - Actual_Effort_in_Days__c`

### 5. Present Comparison Table

Display a table comparing estimated vs recorded days:

```
| ASQ ID | Customer | This Week | SF Days Used | SF Total Days | Remaining | New Days Used |
|--------|----------|-----------|--------------|---------------|-----------|---------------|
| AR-XXX | NM       | 1         | 2.0          | 5.0           | 2.0       | 3.0           |
| AR-YYY | ACME     | 0.5       | 3.0          | 10.0          | 6.5       | 3.5           |
```

Where `New Days Used = current Actual_Effort_in_Days__c + This Week`.

Allow user to adjust the "This Week" estimates before proceeding.

### 6. Confirm Salesforce Updates

**CRITICAL: Always show the planned updates and wait for explicit user confirmation before writing to Salesforce.**

Show each planned SF update:
```
Planned Salesforce updates:
1. AR-000106904 (NM): Actual_Effort_in_Days__c 2.0 → 3.0 (+1.0 days this week)
2. AR-000107000 (ACME): Actual_Effort_in_Days__c 3.0 → 3.5 (+0.5 days this week)

This will update actual effort in Salesforce.
```

Use AskUserQuestion to confirm:
- Option 1: "Update Salesforce" - proceed with the update
- Option 2: "Adjust estimates" - let user modify the values
- Option 3: "Skip SF update" - only update local notes/log

### 7. Update Salesforce

If confirmed, update each record. The new value is cumulative:

```
new_value = current Actual_Effort_in_Days__c + this_week_days
```

```bash
sf data update record -s ApprovalRequest__c -i {SF_RECORD_ID} -v "Actual_Effort_in_Days__c={new_value}"
```

### 8. Save to Weekly Log

Append or create the weekly log entry at `{ASQ_NOTES_ROOT}/{LOGS_DIR}/{YYYY}/{YYYY}-W{XX}.md`:

```markdown
## Time Summary - Week {XX}

| ASQ | Customer | Days This Week | Total Days Used | Remaining |
|-----|----------|----------------|-----------------|-----------|
| AR-XXX | NM | 1.0 | 3.0 | 2.0 |

Updated in Salesforce: {timestamp}
```

### 9. Show Completion Summary

Display:
- Total days logged this week across all ASQs
- Updated Salesforce records
- Any ASQs approaching their day limits (< 20% of Estimated_Duration__c remaining)
- Weekly log path
