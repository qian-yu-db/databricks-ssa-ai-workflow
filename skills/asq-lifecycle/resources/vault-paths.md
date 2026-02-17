# ASQ Notes Path Configuration

Reference for all paths used by ASQ automation commands. Paths can be overridden in your personal `CLAUDE.md` file.

## Base Paths

| Constant | Default | Description |
|----------|---------|-------------|
| ASQ_NOTES_ROOT | `~/asq-notes` | Root directory for all ASQ notes |
| CUSTOMERS_DIR | `customers/` | Customer engagement notes |
| TEMPLATES_DIR | `templates/` | Note templates |
| LOGS_DIR | `logs/` | Activity and time logs |

## File Patterns

| Pattern | Example | Used By |
|---------|---------|---------|
| `customer-{CODE}-AR-{ID}.md` | `customer-TRV-AR-000106904.md` | /asq-intake, /asq-research, /asq-architecture, /asq-status |
| `customer-{CODE}-AR-{ID}-architecture.mermaid` | `customer-TRV-AR-000106904-architecture.mermaid` | /asq-architecture |
| `YYYY/YYYY-WXX.md` | `2026/2026-W07.md` | /asq-status, /asq-timetrack |

## Full Paths

| Resource | Full Path |
|----------|-----------|
| Customer note | `{ASQ_NOTES_ROOT}/customers/customer-{CODE}-AR-{ID}.md` |
| Architecture diagram | `{ASQ_NOTES_ROOT}/customers/customer-{CODE}-AR-{ID}-architecture.mermaid` |
| Template | `{ASQ_NOTES_ROOT}/templates/customer-note.md` |
| Weekly log | `{ASQ_NOTES_ROOT}/logs/YYYY/YYYY-WXX.md` |

## Overriding Paths

If your directory structure differs from the defaults, add this to your `~/.claude/CLAUDE.md`:

```markdown
## ASQ Notes Configuration
- ASQ_NOTES_ROOT: ~/my/custom/path
- CUSTOMERS_DIR: my-customers/
- TEMPLATES_DIR: my-templates/
- LOGS_DIR: my-logs/
```

The plugin will use these overrides when present in CLAUDE.md.

## Company Code Derivation

1. Take the account name from Salesforce `Account__r.Name`
2. Create a 2-4 letter uppercase abbreviation (e.g., Travelers → TRV, Acme Corp → ACME)
3. **Always check existing files** in the customers directory first to see what codes are already in use
4. If the account already has notes with a code, reuse that code
5. Store derived codes in the customer note frontmatter for future reference
