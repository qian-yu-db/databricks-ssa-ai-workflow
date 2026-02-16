# Vault Path Constants

Reference for all paths used by ASQ automation commands.

## Base Paths

| Constant | Value | Description |
|----------|-------|-------------|
| VAULT_ROOT | `~/workspace/databricks_knowledge_vault` | Obsidian vault root |
| CUSTOMERS_DIR | `02-customers/` | Customer engagement notes |
| TEMPLATES_DIR | `30-templates/` | Note templates |
| LOGS_DIR | `50-logs/` | Activity and time logs |

## File Patterns

| Pattern | Example | Used By |
|---------|---------|---------|
| `customer-{CODE}-AR-{ID}.md` | `customer-TRV-AR-000106904.md` | /asq-intake, /asq-research, /asq-architecture, /asq-status |
| `customer-{CODE}-AR-{ID}-architecture.mermaid` | `customer-TRV-AR-000106904-architecture.mermaid` | /asq-architecture |
| `YYYY/YYYY-WXX.md` | `2026/2026-W07.md` | /asq-status, /asq-timetrack |

## Full Paths

| Resource | Full Path |
|----------|-----------|
| Customer note | `~/workspace/databricks_knowledge_vault/02-customers/customer-{CODE}-AR-{ID}.md` |
| Architecture diagram | `~/workspace/databricks_knowledge_vault/02-customers/customer-{CODE}-AR-{ID}-architecture.mermaid` |
| Template | `~/workspace/databricks_knowledge_vault/30-templates/customer-note.md` |
| Weekly log | `~/workspace/databricks_knowledge_vault/50-logs/YYYY/YYYY-WXX.md` |

## Company Code Derivation

1. Take the account name from Salesforce `Account__r.Name`
2. Create a 2-4 letter uppercase abbreviation (e.g., Travelers → TRV, Acme Corp → ACME)
3. **Always check existing files** in `02-customers/` first: `ls ~/workspace/databricks_knowledge_vault/02-customers/customer-*` to see what codes are already in use
4. If the account already has notes with a code, reuse that code
5. Store derived codes in the customer note frontmatter for future reference
