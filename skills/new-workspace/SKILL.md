---
name: new-workspace
description: Provision a new budgeting workspace on disk. Use when the user wants to start tracking a new household or personal budget. Accepts a workspace name and optional target path. Scaffolds the workspace, personalises CLAUDE.md and context.md from the user's global memory, and (by default) creates a GitHub repo.
disable-model-invocation: true
allowed-tools: Bash(mkdir *), Bash(cp *), Bash(cat *), Bash(git init *), Bash(git add *), Bash(git commit *), Bash(gh repo create *), Bash(gh auth status), Bash(git push *), Read
---

# Provision Budgeting Workspace

Creates a new workspace for household or personal budget tracking. This plugin's commands (`/budgeting:log-transaction`, `/budgeting:categorize`, `/budgeting:forecast-budget`, etc.) are globally available once installed — this skill only provisions the **data scaffold** (CLAUDE.md, context.md, budgets/, transactions/, etc.) that those commands read from and write to.

## Arguments

`$ARGUMENTS` is parsed as:

- **First positional**: workspace name (kebab-case, used as directory and GitHub repo name). Required.
- **Second positional** (optional): target parent path. Defaults to `~/repos/github/my-repos`.
- **`--local-only`** (optional): skip GitHub repo creation and push. Default: create a public GitHub repo and push.
- **`--private`** (optional): create the GitHub repo as private. Default: public.

### Examples

```
/budgeting:new-workspace household-budget
/budgeting:new-workspace family-finances --private
/budgeting:new-workspace scratch-budget --local-only
```

## Procedure

### 1. Parse arguments

Extract workspace name, target parent path, and flags from `$ARGUMENTS`. If workspace name is missing, ask the user for it before proceeding.

### 2. Resolve the scaffold path

The bundled scaffold lives at `${CLAUDE_SKILL_DIR}/../../template/`. Confirm it exists before copying.

### 3. Read ambient facts

Read `~/.claude/CLAUDE.md` if it exists. Extract OS, locale, timezone, currency hints, and user identity facts. These personalise the workspace's CLAUDE.md and context.md at step 6.

### 4. Create the workspace directory

```bash
mkdir -p <target-parent>/<workspace-name>
cp -r ${CLAUDE_SKILL_DIR}/../../template/. <target-parent>/<workspace-name>/
```

Do **not** copy any `.claude/` tree. The plugin's primitives are global.

### 5. Personalise CLAUDE.md and context.md

Open the new workspace's `CLAUDE.md` and:

- Replace any placeholder identity with facts from step 3.
- Add a short header noting the workspace name.
- Embed OS/locale/timezone where relevant.

Open `context.md` and pre-fill whatever ambient facts make sense (timezone, currency if inferrable, locale). Leave the rest as placeholders for the user to fill in via `/budgeting:setup-workspace`.

### 6. Prompt for workspace-specific facts

Ask the user only for facts this plugin can't infer:

- **Currency** (e.g., USD, EUR, ILS) if not obvious from locale.
- **Budget period** (monthly by default; weekly/custom available).
- **Household size** (optional — can be deferred to setup-workspace).

Write answers into `context.md`.

### 7. Initialise git and (optionally) publish

```bash
cd <target-parent>/<workspace-name>
git init
git add .
git commit -m "Initial workspace from budgeting plugin"
```

Unless `--local-only` is set:

```bash
gh repo create <workspace-name> --<public|private> --source=. --push
```

Use `--public` by default, `--private` if flag was passed.

**Note on privacy**: budgeting workspaces often contain sensitive financial data. Remind the user that `.gitignore` already excludes raw transaction files, but they should review before committing anything sensitive. Consider `--private` for real household data.

### 8. Print next steps

Tell the user:

- Workspace path.
- Which plugin commands apply next:
  - `/budgeting:setup-workspace` to run the full interactive setup wizard.
  - `/budgeting:log-transaction` or `/budgeting:process-transactions` once they have data.
  - `/budgeting:create-monthly-budget` to build the first budget.
  - `/budgeting:set-financial-goal` to record savings/debt-payoff goals.
- Reminder that the workspace is **data** — they can delete/move it freely without losing the plugin's commands.

## Notes

- The scaffold path must be resolved via `${CLAUDE_SKILL_DIR}/../../template/` (not `${CLAUDE_PLUGIN_ROOT}` — that variable isn't exported in skill bash injection, only in hooks/MCP).
- Never copy `.claude/commands/`, `.claude/agents/`, or `.claude/skills/` into the new workspace. If the user wants workspace-local overrides they can add them manually later.
- Don't hard-code personal paths, currencies, or identifiers here — everything comes from user memory or prompts.
