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
---

# Enter Skill

Use `enter-cli` to operate [enter.pro](https://enter.pro) — an AI-native platform where agents build, deploy, and publish full-stack web apps from natural language.

## First Checks

```bash
enter-cli whoami -o json
enter-cli ws list -o json
```

Do not continue until `whoami` succeeds.

## Typical Workflow

```bash
# 1. Create
enter-cli proj create <workspace_id> --prompt "Build me a ..." -o json   # → project_id

# 2. Wait for the turn (always after create / chat)
enter-cli thread wait <project_id> -o json
#   completed → continue
#   blocked   → run the returned approve_command, then wait again
#   failed    → surface the error to the user

# 3. Iterate
enter-cli thread chat <project_id> -m "Add a login page"
#   long / quoted / multi-line message → write to a file first:
#   enter-cli thread chat <project_id> --file /tmp/msg.txt
# then thread wait again

# 4. Inspect
enter-cli proj get <project_id> -o json
enter-cli thread turns <project_id> -o json
enter-cli thread diff <project_id> <turn_number> -o json

# 5. Publish (synchronous; trust the exit code)
enter-cli proj publish <project_id>

# 6. Share / source
enter-cli proj urls <project_id> -o json
enter-cli proj download <project_id> --out /tmp/src.zip
#   VIP_REQUIRED → fall back to:
#   enter-cli proj source-code <project_id> -o json
```

When `thread wait` returns `blocked`, dispatch by `input_kind`: `none` runs the command as-is; `secret` asks the user for the value first; `questions` reads the prompt from `thread messages` and replies with `--answers '<json>'`.

## Discovering Commands

For command lists, flags, and usage, run `enter-cli <cmd> --help` directly. Top-level groups: `login`, `logout`, `whoami`, `config`, `workspace|ws`, `project|proj`, `thread`, `domain`. This skill does **not** restate `--help` output — it captures only what `--help` cannot tell you.

## Decision Rules

- Prefer `publish_url` over `preview_url` when both exist.
- If only `preview_url` exists, label it clearly as preview.
- `visibility public` is not the same as production publish.
- `proj publish` is synchronous (precheck + poll + URL probe). It exits non-zero on failure; trust its exit code.
- If `proj download` is blocked by plan limits, export `thread diff` JSON instead of pretending the source was downloaded.
- If CLI help and backend behavior disagree, check [Quirks](references/quirks.md) first.

## References

- [references/shapes.md](references/shapes.md) — index of verified JSON output shapes by command group (auth, workspace, project, thread, domain, mcp-skills). Read the relevant sub-file before parsing JSON.
- [references/quirks.md](references/quirks.md) — failure modes, async timing, gate-approval flag matrix, plan-gate fallbacks.
