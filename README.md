# budgeting-plugin

Claude Code plugin for household and personal budgeting — transaction logging, categorisation, budget creation, spending analysis, goal tracking, reporting, and forecasting. Ships a provisioning skill that scaffolds a fresh budgeting workspace on demand.

Part of the [danielrosehill Claude Code marketplace](https://github.com/danielrosehill/Claude-Code-Plugins).

## What you get

### Primitives (always available once the plugin is installed)

**Commands** (`/budgeting:*`):

- `setup-workspace` — interactive setup wizard that populates `context.md` with household profile, income, expenses, goals, and budget preferences.
- `log-transaction` — record a single transaction (manual entry) into the current month's ledger.
- `process-transactions` — import and categorise bulk transaction files from `transactions/import/`.
- `categorize` — review uncategorised or likely mis-categorised transactions; learns merchant→category rules.
- `create-monthly-budget` — build a monthly budget from context, history, and goals.
- `analyze-spending` — deep-dive analysis of spending patterns over a chosen period.
- `forecast-budget` — project cash flow and goal trajectories over 3/6/12 months with sensitivity analysis.
- `monthly-report` — comprehensive monthly financial report.
- `set-financial-goal` — define a new SMART financial goal.
- `review-goals` — review progress on all active goals and flag at-risk ones.

**Agents**:

- `budget-architect` — creates monthly and annual budgets.
- `expense-analyst` — analyses spending patterns and anomalies.
- `goal-tracker` — monitors progress toward savings and debt-payoff goals.
- `transaction-processor` — imports and categorises transaction data.
- `report-generator` — produces monthly, quarterly, and annual reports.
- `financial-advisor` — strategic guidance, scenario planning, larger decisions.

### Provisioning skill

- `/budgeting:new-workspace <name> [<target-path>] [--local-only] [--private]`

Scaffolds a new workspace (CLAUDE.md + context.md + budgets/transactions/reports/outputs structure), personalises it from `~/.claude/CLAUDE.md`, and (by default) creates a public GitHub repo for it. Use `--private` for real household data you don't want to publish.

## Pattern

Primitives live in the plugin → globally available from any cwd.
Workspace scaffolds are provisioned as **data** → no `.claude/` tree inside provisioned workspaces.
Plugin updates never touch your workspace data.

See [PLAN.md in Claude-Workspace-Reshaping-190426](https://github.com/danielrosehill/Claude-Workspace-Reshaping-190426) for the full pattern spec this plugin follows.

## Install

Via the danielrosehill marketplace:

```
/plugin marketplace add danielrosehill/Claude-Code-Plugins
/plugin install budgeting
```

## Quick start

```
/budgeting:new-workspace household-budget --private
cd ~/repos/github/my-repos/household-budget
/budgeting:setup-workspace
/budgeting:create-monthly-budget
```

## Privacy notes

Budgeting workspaces often contain sensitive financial data (account identifiers, balances, transaction history). A few built-in protections:

- The scaffold's `.gitignore` excludes raw transaction files by default.
- Use `--private` on `new-workspace` for real household data.
- Never commit full account numbers; prefer last-4 masked identifiers.
- Keep bank credentials in a password manager, never in the repo.

## License

MIT.
