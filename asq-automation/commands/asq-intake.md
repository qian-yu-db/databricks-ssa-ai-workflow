---
description: Pull ASQ data from Salesforce and create a customer note
argument-hint: "AR-XXXXXX"
model: sonnet
---

# /asq-intake - Import ASQ from Salesforce

Create a structured customer note from Salesforce ASQ (Approval Request) data.

## Usage

```
/asq-intake AR-000106904
```

## Workflow

### 0. Resolve Paths

Read `~/.claude/CLAUDE.md` and look for an `## ASQ Notes Configuration` section. If found, use those paths. Otherwise use defaults:
- `ASQ_NOTES_ROOT`: `~/asq-notes`
- `CUSTOMERS_DIR`: `customers/`
- `TEMPLATES_DIR`: `templates/`

### 1. Parse AR ID

Extract the AR ID from the argument. Accept formats: `AR-000106904` or `000106904` (auto-prefix AR-).

### 2. Query Salesforce

Run this query using Bash to get the ASQ record:

```bash
sf data query -q "SELECT Id, Name, Status__c, Request_Type__c, Support_Type__c, Urgency__c, Request_Description__c, Situation_Details__c, Start_Date__c, End_Date__c, Estimated_Duration__c, Hours_Consumed__c, Total_Hours__c, Remaining_Hours_of_Investment__c, Account__r.Name, Requestor__r.Name, Resource__r.Name, Approved_Start_Date__c, Approved_End_Date__c, Approved_Investment_Per_Period__c, Approved_Frequency__c, EngagementType__c FROM ApprovalRequest__c WHERE Name = 'AR-XXXXXX'" --json
```

If the query returns no results, inform the user and stop.

### 3. Derive Company Code

1. Extract the account name from `Account__r.Name`
2. Check existing files for this account:
   ```
   Glob for customer-*-AR-*.md in {ASQ_NOTES_ROOT}/{CUSTOMERS_DIR}
   ```
3. If the account already has notes, reuse the existing company code from the filename
4. If new account, derive a 2-4 letter uppercase code:
   - Single word: first 3-4 letters (e.g., Travelers → TRV)
   - Multi-word: first letter of each word (e.g., Acme Corp → AC)
   - Common patterns: remove "Inc", "Corp", "LLC", "Co" before deriving

### 4. Check for Existing Note

Check if `customer-{CODE}-AR-{ID}.md` already exists. If yes, ask the user whether to overwrite or update.

### 5. Read Template

Read the template from `{ASQ_NOTES_ROOT}/{TEMPLATES_DIR}/customer-note.md`. If the template doesn't exist, use a minimal built-in template.

### 6. Create Customer Note

Write the note to `{ASQ_NOTES_ROOT}/{CUSTOMERS_DIR}/customer-{CODE}-AR-{ID}.md` with:

**Frontmatter** (extend template frontmatter):
```yaml
---
type: customer
customer_name: {Account Name}
industry:
account_tier:
status: status/active
created: {today YYYY-MM-DD}
last_updated: {today YYYY-MM-DD}
asq_id: AR-XXXXXX
support_type: {Support_Type__c}
start_date: {Start_Date__c or Approved_Start_Date__c}
end_date: {End_Date__c or Approved_End_Date__c}
estimated_days: {Estimated_Duration__c}
total_hours: {Total_Hours__c}
hours_consumed: {Hours_Consumed__c}
---
```

**ASQ Summary section**: Populate with:
- Request description (Request_Description__c)
- Situation details (Situation_Details__c)
- Support type, urgency, engagement type
- Date range and estimated effort
- Requestor and assigned resource names

**Technical Environment section**: Pre-fill cloud provider if mentioned in description.

**Key Stakeholders table**: Add requestor and resource from SF data.

### 7. Generate Open Items

Based on what's missing or needs clarification, auto-generate open items:
- [ ] Confirm primary technical contact at customer
- [ ] Schedule initial discovery call
- [ ] Review existing Databricks workspace setup
- [ ] Identify success criteria and timeline milestones
- Add more based on gaps in the SF data (e.g., missing industry, missing cloud provider)

### 8. Show Summary

Display to the user:
- Note created at: {path}
- Customer: {account name}
- ASQ: {AR ID} - {support type}
- Status: {status}
- Date range: {start} to {end}
- Hours: {consumed}/{total} ({remaining} remaining)
- Open items count
