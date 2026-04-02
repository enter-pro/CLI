---
name: enter
description: >
  Use this skill to operate enter.pro through the verified enter-cli workflow.
  Trigger when the user wants to create, inspect, iterate on, publish, remix,
  or share an Enter project; inspect thread turns, messages, diffs, domains,
  workspace members, or project URLs; or recover from Enter CLI / API drift.
  Do not use for the local enter_agent_sdk or general local code execution.
metadata:
  install:
    - method: npm
      command: npm install -g @kntech/enter-cli
  bin: enter-cli
  version: "0.1.3 verified locally on 2026-04-01"
---

# Enter Skill

Use `enter-cli` first. Treat the CLI as the primary interface, but do not assume
its help text is fully accurate: some commands drift from backend behavior.

## First Checks

Run these before doing real work:

```bash
enter-cli whoami -o json
enter-cli ws list -o json
enter-cli --help
```

Do not continue until `whoami` succeeds.

## High-Confidence Capabilities

- Inspect workspaces and projects
- Read project details, including `preview_url`, `publish_url`, and `lifecycle_status`
- Create projects with `--name`, `--prompt`, and `--wait` (blocks until first build done)
- Send build instructions through project threads
- Inspect thread turns (`--wait`, `--latest`), messages (`--latest`, `--tail`, `--turn`, `--follow`), and per-turn diffs (`--out`)
- Trigger project publish with `--wait` polling
- Inspect project domains, visibility, and source code
- Remix projects
- Disable/re-enable plan mode when a project is stuck
- List, approve, and reject gate actions (Supabase, Stripe, AI capability, user questions)
- Manage workspace members (add, remove, update-role, leave)
- Refresh and update custom domains
- List, search, and inspect skills; install/uninstall skills to agents; manage skill visibility
- Directly edit project source files (`proj edit-file`)
- List sandbox env vars, run scripts in sandbox
- Manage project MCP servers (add, list, update, delete)
- Switch AI model for a project (`proj model`)
- Manage project-level skills (list, enable/disable)
- Delete and promote thread tasks

## Use This Skill Like This

1. Start with the health checks above.
2. Prefer `-o json` for any command whose output you will inspect or parse.
3. Use verified output shapes from [verified-cli.md](./references/verified-cli.md).
4. Before trusting a command, check [known-quirks.md](./references/known-quirks.md) for drift, plan gates, and fallback rules.
5. For end-to-end task patterns, use [workflows.md](./references/workflows.md).

## Decision Rules

- Prefer `publish_url` over `preview_url` when both exist.
- If only `preview_url` exists, label it clearly as preview.
- `visibility public` is not the same thing as production publish.
- After `proj publish`, confirm success via `proj get` and a production URL reachability check instead of trusting the CLI call to return quickly.
- If `proj download` is blocked by plan limits, export `thread diff` JSON instead of pretending the source was downloaded.
- If CLI help and backend behavior disagree, use a direct authenticated API fallback only after confirming `enter-cli whoami` works.

## Read Next

- [verified-cli.md](./references/verified-cli.md) for verified commands and output shapes
- [known-quirks.md](./references/known-quirks.md) for drift and failure modes
- [workflows.md](./references/workflows.md) for safer operator workflows
