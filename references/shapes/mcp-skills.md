# MCP & Skills Shapes

> Note: there is no top-level `enter-cli skill` group. Skills are managed per-project via `proj skills`.

## `proj mcp list <project_id> -o json`

Wrapped: `{servers: [...]}`.

```json
{ "servers": [] }
```

Server object (when present):

```json
{
  "servers": [
    {
      "id": "<uuid>",
      "name": "filesystem",
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/tmp"],
      "env": { "FOO": "bar" },
      "enabled": true,
      "created_at": "...",
      "updated_at": "..."
    }
  ]
}
```

`type` enum: `stdio` | `http` | `sse`.

## `proj mcp get <project_id> <server_id> -o json`

Returns a single server object (same shape as the array element above), naked (not wrapped in `{server: ...}`).

## `proj mcp create <project_id> --name <name> --url <url> --transport <sse|http|stdio> -o json`

```json
{ "id": "<uuid>", "name": "filesystem" }
```

`--transport` enum: `sse` | `http` | `stdio`.

## `proj mcp update <project_id> <server_id> [--display-name <name>] [--enabled <bool>] [--allowed-tools <a,b,c>] -o json`

```json
{ "id": "<uuid>", "enabled": false, "updated_at": "..." }
```

`--allowed-tools` is a comma-separated list (string, not array).

## `proj mcp delete <project_id> <server_id> -o json`

```json
{ "deleted": true, "id": "<uuid>" }
```

## `proj skills list <project_id> -o json`

Wrapped: `{skills: [...], total}`.

```json
{
  "total": 1,
  "skills": [
    {
      "name": "enter-pro-guide",
      "skill_key": "enter-pro-guide",
      "description": "Enter.pro product expert — guides users through every feature...",
      "instruction": "---\nname: enter-pro-guide\ndescription: ...\n---\n\n# Enter.pro Product Guide\n...",
      "state": "enabled",
      "type": "official",
      "version": 1
    }
  ]
}
```

- `state`: `enabled` | `disabled`.
- `type`: `official` | `custom` | `community`.
- `version` is an integer.
- `instruction` contains the full SKILL.md content (frontmatter + body) as a single string. Can be very large (10s of KB).
- `skill_key` is the stable identifier; `name` may be a display name (often identical).

## `proj skills update <project_id> --skill-key <key> --enabled <true|false> -o json`

```json
{ "skill_key": "enter-pro-guide", "enabled": false }
```
