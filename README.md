# enter-cli Skill

[![Supported by enter.pro](https://img.shields.io/badge/Build%20with-Enter.pro-FC5776?style=for-the-badge&labelColor=1F1F1F)](https://enter.pro)

> Build and ship full-stack web apps with AI — powered by [enter.pro](https://enter.pro)

This skill teaches any AI agent how to operate **[enter.pro](https://enter.pro)** — an AI-native platform where you describe what you want to build and an agent writes, deploys, and publishes the entire app for you.

Works with any agent that supports skills: [OpenClaw](https://openclaw.ai), [Claude Code](https://claude.ai/code), [Codex](https://platform.openai.com/docs/guides/codex), [QClaw](https://qclaw.run), and more.

---

## What is enter.pro?

**[enter.pro](https://enter.pro)** is a platform that lets AI agents build complete web applications — frontend, backend, database, auth, payments, and deployment — all from a single chat prompt.

- **No manual setup** — AI handles the entire stack
- **Real deployments** — apps are live on a real URL, not just a preview
- **Integrations built-in** — Supabase, Stripe, custom domains, MCP servers
- **Multiplayer** — team workspaces, shared projects, role-based access

---

## What This Skill Does

Gives your agent full control over enter.pro via `enter-cli`:

| Capability | Commands |
|-----------|----------|
| Build & iterate | `thread chat`, `thread messages --follow` |
| Handle build gates | `thread actions --pending`, `thread approve` |
| Publish & share | `proj publish --wait`, `proj urls` |
| Edit code directly | `proj edit-file`, `proj run-script` |
| Manage integrations | `proj mcp`, `proj model`, `proj skills` |
| Workspaces & teams | `ws members`, `ws credits` |
| Custom domains | `domain add`, `domain refresh` |
| Agent skills | `skill install`, `skill list` |

---

## Install

```bash
npm install -g @kntech/enter-cli
enter-cli login        # OAuth — opens browser
enter-cli whoami       # verify auth
```

---

## Add to Your Agent

### Claude Code
```bash
cp -r . ~/.claude/skills/enter
```

### Cursor / Windsurf / Cline / Continue
Copy the `SKILL.md` content into your agent's system prompt or rules file, or follow your agent's skill/rule import instructions.

### agentskills.io
Available at [agentskills.io](https://agentskills.io) — search for `enter`.

---

## Quick Start

```bash
# 1. Get your workspace
WS_ID=$(enter-cli ws list -o json | jq -r '.[0].id')

# 2. Create a project and wait for the first build
PROJ=$(enter-cli proj create $WS_ID --name "My App" --prompt "Build a SaaS landing page with waitlist" --wait -o json)
PROJ_ID=$(echo $PROJ | jq -r '.project_id')

# 3. Get the shareable URL
enter-cli proj urls $PROJ_ID -o json | jq -r '.recommended_share_url'

# 4. Iterate
enter-cli thread chat $PROJ_ID -m "Add dark mode and a pricing section"
enter-cli thread messages $PROJ_ID --follow

# 5. Publish
enter-cli proj publish $PROJ_ID --wait
```

---

## Handling Build Gates

Some features require approval before the agent proceeds (Supabase setup, Stripe keys, user questions). Use:

```bash
enter-cli thread actions $PROJ_ID --pending -o json

# Approve by type:
enter-cli thread approve $PROJ_ID <action_id>                                                    # supabase_enable, confirm_skill, etc.
enter-cli thread approve $PROJ_ID <action_id> --secret-name DB_URL --secret-value "postgres://…" # supabase_add_secret
enter-cli thread approve $PROJ_ID <action_id> --secret-name STRIPE_SECRET_KEY --secret-value "sk_…" # stripe_enable
enter-cli thread approve $PROJ_ID <action_id> --skip-answers                                    # ask_user_question

# Unblock plan mode
enter-cli proj plan-mode $PROJ_ID disable
```

---

## Long Messages

For long prompts with special characters, write to a file first:

```bash
cat > /tmp/prompt.txt << 'EOF'
Build a multiplayer drawing game with:
- PeerJS for real-time sync
- 200+ word bank in Chinese
...
EOF
enter-cli thread chat $PROJ_ID --file /tmp/prompt.txt
```

---

## Full Command Reference

See [references/verified-cli.md](references/verified-cli.md) for the complete command reference with all flags.

See [references/workflows.md](references/workflows.md) for end-to-end workflow examples.

See [references/known-quirks.md](references/known-quirks.md) for known issues and how to handle them.

---

## Links

- **Platform**: [enter.pro](https://enter.pro)
- **CLI package**: [@kntech/enter-cli](https://www.npmjs.com/package/@kntech/enter-cli)
- **Skill registry**: [agentskills.io](https://agentskills.io)

---

## License

MIT
