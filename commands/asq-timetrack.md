---
description: Weekly time summary and Salesforce hours sync for active ASQs
model: sonnet
---

# /asq-timetrack - Time Tracking Summary

Generate a weekly time summary across all active ASQs and sync hours to Salesforce.

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

### 3. Estimate Hours from Activity

For each active ASQ, estimate hours spent this week by reviewing:
- Engagement History entries in the customer note dated this week
- Any time entries in the weekly log for this ASQ
- Present estimates to user for adjustment

### 4. Query Salesforce

Query SF for current hours data on all active ASQs:

```bash
sf data query -q "SELECT Id, Name, Hours_Consumed__c, Total_Hours__c, Remaining_Hours_of_Investment__c, Account__r.Name FROM ApprovalRequest__c WHERE Name IN ('AR-XXX','AR-YYY')" --json
```

### 5. Present Comparison Table

Display a table comparing estimated vs recorded hours:

```
| ASQ ID | Customer | This Week (est) | SF Hours Used | SF Total Hours | SF Remaining | New Hours Used |
|--------|----------|-----------------|---------------|----------------|--------------|----------------|
| AR-XXX | TRV      | 4h              | 12.0          | 40.0           | 28.0         | 16.0           |
| AR-YYY | ACME     | 2h              | 8.0           | 20.0           | 12.0         | 10.0           |
```

Allow user to adjust the "This Week" estimates before proceeding.

### 6. Confirm Salesforce Updates

**CRITICAL: Always show the planned updates and wait for explicit user confirmation.**

Show each planned SF update:
```
Planned Salesforce updates:
1. AR-000106904 (TRV): Hours_Consumed__c 12.0 → 16.0
2. AR-000107000 (ACME): Hours_Consumed__c 8.0 → 10.0

Confirm? (yes/no)
```

Use AskUserQuestion to get confirmation.

### 7. Update Salesforce

If confirmed, update each record:

```bash
sf data update record -s ApprovalRequest__c -i {SF_RECORD_ID} -v "Hours_Consumed__c={new_hours}"
```

### 8. Save to Weekly Log

Append or create the weekly log entry at `{ASQ_NOTES_ROOT}/{LOGS_DIR}/{YYYY}/{YYYY}-W{XX}.md`:

```markdown
## Time Summary - Week {XX}

| ASQ | Customer | Hours This Week | Total Hours Used | Remaining |
|-----|----------|-----------------|------------------|-----------|
| AR-XXX | TRV | 4.0 | 16.0 | 24.0 |

Updated in Salesforce: {timestamp}
```

### 9. Show Completion Summary

Display:
- Total hours logged this week across all ASQs
- Updated Salesforce records
- Any ASQs approaching their hour limits (< 20% remaining)
- Weekly log path
