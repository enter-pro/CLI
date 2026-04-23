# enter-cli Skill

[![Supported by enter.pro](https://img.shields.io/badge/Supported%20by-Enter.pro-FC5776?style=for-the-badge&labelColor=1F1F1F)](https://enter.pro)

> Build and ship full-stack web apps with AI — powered by [enter.pro](https://enter.pro)

A skill that teaches your AI agent how to drive **[enter.pro](https://enter.pro)**. You don't run these commands — your agent does.

Works with any agent that supports skills: [OpenClaw](https://openclaw.ai), [Claude Code](https://claude.ai/code), [Codex](https://platform.openai.com/docs/guides/codex), [QClaw](https://qclaw.run), and more.

---

## What this enables your agent to do

- Spin up a full-stack web app from a natural-language prompt
- Plan first, then build — review the agent's plan before any code is written
- Iterate on the app via chat (add features, fix bugs, change copy)
- Verify the live preview after each change and catch runtime errors automatically
- Publish to a real production URL on enter.pro
- Wire up integrations: Supabase, Stripe, custom domains, MCP servers
- Manage workspaces, members, credits, and project visibility

All from natural language. The agent picks the right CLI commands; you just describe what you want.

---

## How it works

Your agent uses `enter-cli` — a thin wrapper over enter.pro's API — to do everything you'd otherwise do in the web UI. The skill (`SKILL.md`) tells the agent which commands to use, in what order, and how to handle approval gates and errors.

---

## Install (one-time setup)

You need to do these three steps once. After that, your agent takes over.

**1. Install the CLI**

```bash
npm install -g @kntech/enter-cli
```

**2. Log in**

```bash
enter-cli login
```

This opens a browser for OAuth. Your credentials are stored locally so the agent can act on your behalf.

**3. Add the skill to your agent**

See [Add the skill to your agent](#add-the-skill-to-your-agent) below.

---

## Add the skill to your agent

Paste this into your agent (Claude Code, Codex, Cursor, OpenClaw, QClaw, …) and let it install the skill itself:

> Install the enter-cli skill from https://github.com/enter-pro/CLI into the appropriate skills/rules location for this environment.

The agent will figure out where skills live for its runtime and clone or copy the files in.

---

## Try it

Once installed, just ask your agent things like:

- "Build me a 2048 game and publish it"
- "Add Supabase auth to my project `abc123`"
- "Show me the latest preview URL for my project and check for errors"
- "Iterate on my landing page — add a pricing section in dark mode"

Your agent handles the rest: creating the project, reviewing the plan with you, building, verifying the preview, approving gates, and publishing.

---

## For developers / curious humans

If you want to invoke `enter-cli` directly instead of letting an agent drive it:

- Run `enter-cli --help` for the full command tree
- See [SKILL.md](SKILL.md) for the canonical workflow
- See [references/shapes.md](references/shapes.md) for verified JSON output shapes
- See [references/quirks.md](references/quirks.md) for known edge cases

---

## Links

- **Platform**: [enter.pro](https://enter.pro)
- **CLI package**: [@kntech/enter-cli](https://www.npmjs.com/package/@kntech/enter-cli)
- **Skill registry**: [agentskills.io](https://agentskills.io)

---

## License

MIT
