---
description: Autonomous research across Glean, Slack, and web for an ASQ engagement
argument-hint: "AR-XXXXXX"
model: opus
---

# /asq-research - Autonomous ASQ Research

Launch autonomous multi-source research for an ASQ engagement. Searches Glean, Slack, web, and Google tools to gather technical information and writes findings directly into the customer note.

## Usage

```
/asq-research AR-000106904
```

## Workflow

### 0. Resolve Paths

Read `~/.claude/CLAUDE.md` and look for an `## ASQ Notes Configuration` section. If found, use those paths. Otherwise use defaults:
- `ASQ_NOTES_ROOT`: `~/asq-notes`
- `CUSTOMERS_DIR`: `customers/`

### 1. Parse AR ID

Extract the AR ID from the argument. Accept formats: `AR-000106904` or `000106904` (auto-prefix AR-).

### 2. Find Customer Note

Use Glob to find the customer note:
```
Glob for customer-*-AR-{ID}.md in {ASQ_NOTES_ROOT}/{CUSTOMERS_DIR}
```

If not found, tell the user:
> No customer note found for {AR ID}. Run `/asq-intake {AR ID}` first to create the note from Salesforce.

### 3. Read and Extract Context

Read the customer note and extract:
- **Customer name** from frontmatter `customer_name`
- **Account name** from the note content or frontmatter
- **Technical keywords** from ASQ Summary, Technical Environment, and Problem Statements sections
- **Open questions** from the Open Items section
- **Databricks services** mentioned (Vector Search, Unity Catalog, Delta Lake, MLflow, etc.)
- **Cloud provider** from Technical Environment
- **Data scale** and performance requirements

### 4. Launch Research Agent

Use the Task tool to spawn the `asq-automation:asq-researcher` agent:

```
Task(
    subagent_type="asq-automation:asq-researcher",
    prompt="Research for ASQ {AR ID} at {customer name}.

Customer note path: {full file path}

Context:
- Customer: {customer name}
- Account: {account name}
- Technical domain: {keywords}
- Databricks services: {services list}
- Cloud: {cloud provider}
- Key questions: {open questions}
- Challenges: {extracted challenges}

Please search Glean, Slack, and web for relevant information and write findings directly into the customer note at the path above.",
    model="opus"
)
```

### 5. Verify Results

After the agent completes:
1. Re-read the customer note to verify the Research Findings section was added
2. Check that sources are cited
3. Check that open items were updated

### 6. Show Summary

Display to the user:
- Research completed for {AR ID} ({customer name})
- Sources searched: Glean, Slack, Web
- Key findings count
- New open questions added
- Note path for review

## Notes

- Research is **additive** - it adds a Research Findings section without modifying existing content
- If Research Findings already exists, the agent will update/replace it with fresh findings
- The agent runs autonomously but all changes are to local notes (no external writes)
- Research typically takes 1-3 minutes depending on the breadth of sources found
