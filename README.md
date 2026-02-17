# databricks-ssa-ai-workflow

Claude Code plugin marketplace for Databricks SSA AI workflows.

## Plugins

- **[asq-automation](./asq-automation/)** - Automate the ASQ (Approval Request / Specialist SA) lifecycle

## Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/qian-yu-db/databricks-ssa-ai-workflow.git
cd databricks-ssa-ai-workflow

# 2. Register as a Claude Code plugin marketplace
claude plugin marketplace add "$(pwd)"

# 3. Install the plugin
claude plugin install asq-automation@databricks-ssa-ai-workflow
```

Then start a new Claude Code session and type `/asq-` to see available commands.

See [asq-automation/README.md](./asq-automation/README.md) for full setup and usage instructions.
