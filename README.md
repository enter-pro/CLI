# enter-cli Skill

A skill for operating [enter.pro](https://enter.pro) via the `enter-cli` command-line tool.

Load this skill when you want to create, build, publish, or manage Enter projects using an AI agent.

## What This Skill Covers

- Create and manage workspaces and projects
- Send chat messages to trigger AI builds, with real-time stream following
- Handle all build gate types (Supabase, Stripe, AI capability, user questions, plan mode)
- Publish projects and get shareable URLs
- Inspect thread turns, messages, and per-turn diffs
- Manage project MCP servers, AI model selection, and project-level skills
- Directly edit source files in the sandbox
- List and manage agent skills
- Manage workspace members and custom domains

## Install

```bash
npm install -g @kntech/enter-cli
enter-cli login
```

## Skill Structure

```
SKILL.md                  # Main skill file — load this in Claude Code
references/
  verified-cli.md         # Full --help reference for all commands
  known-quirks.md         # Known issues and how to handle them
  workflows.md            # End-to-end operator workflows
```

## Usage

Install via [agentskills.io](https://agentskills.io) or copy the skill directory into your agent's skills folder.

## Key Workflows

### Create a project and wait for build
```bash
WS_ID=$(enter-cli ws list -o json | jq -r '.[0].id')
PROJ=$(enter-cli proj create $WS_ID --name "My App" --prompt "Build a todo app" --wait -o json)
PROJ_ID=$(echo $PROJ | jq -r '.project_id')
enter-cli proj urls $PROJ_ID -o json | jq -r '.recommended_share_url'
```

### Send a long message (avoids shell escaping issues)
```bash
cat > /tmp/msg.txt << 'EOF'
Build a multiplayer game with the following features...
EOF
enter-cli thread chat $PROJ_ID --file /tmp/msg.txt
```

### Handle a build gate
```bash
# Check for pending gates
enter-cli thread actions $PROJ_ID --pending -o json

# Approve by type
enter-cli thread approve $PROJ_ID <action_id>                                          # supabase_enable / confirm_skill / etc.
enter-cli thread approve $PROJ_ID <action_id> --secret-name DB_URL --secret-value "…" # supabase_add_secret
enter-cli thread approve $PROJ_ID <action_id> --skip-answers                           # ask_user_question
```

### Unblock plan mode
```bash
enter-cli proj plan-mode $PROJ_ID disable
```

## All Commands

See [references/verified-cli.md](references/verified-cli.md) for the full command reference.

| Group | Commands |
|-------|----------|
| `ws` | list, get, create, delete, members (add/remove/update-role/leave), credits |
| `proj` | list, get, create, rename, delete, publish, publish-status, urls, remix, visibility, plan-mode, source-code, read-file, edit-file, env-vars, run-script, model, download, mcp, skills |
| `thread` | chat, messages, turns, diff, cancel, tasks, task-delete, task-promote, restore, actions, approve, reject |
| `domain` | list, add, remove, refresh, update |
| `skill` | list, search, get, delete, visibility, install, uninstall, agents |

## License

MIT
