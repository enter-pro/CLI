# Verified CLI Notes

Verified locally on 2026-04-02 against installed `enter-cli` 0.1.3 (post-improvement build).

## Verified Install Surface

- Package: `@kntech/enter-cli@0.1.3`
- Binary: `enter-cli`

### `enter-cli --help`

```
Commands: login, logout, whoami, config, workspace|ws, project|proj, thread, domain
Options:  -o <json|table|yaml>  -v (verbose)
```

### `enter-cli proj --help`

```
list [options] <workspace_id>             --page --page-size
get <project_id>
create [options] <workspace_id>           --name --prompt --wait --timeout
rename <project_id> <new_name>
delete <project_id>
download [options] <project_id>           --out
publish [options] <project_id>            --wait --timeout
publish-status <project_id>
urls <project_id>
remix [options] <project_id>              --workspace-id
visibility <project_id> <public|private>
plan-mode <project_id> <enable|disable>
source-code <project_id>
read-file [options] <project_id>          --path (required)
edit-file [options] <project_id>          --commit (required)  --path (required)  --code (required)
env-vars <project_id>
run-script [options] <project_id>         --script (required)  --env (JSON)
model <project_id> <model>
mcp                                       (subgroup: list / get / create / update / delete)
skills                                    (subgroup: list / update)
```

### `enter-cli proj mcp --help`

```
list <project_id>
get <project_id> <server_id>
create [options] <project_id>             --name (required)  --url (required)  --transport
update [options] <project_id> <server_id> --display-name  --enabled  --allowed-tools
delete <project_id> <server_id>
```

### `enter-cli proj skills --help`

```
list <project_id>
update [options] <project_id>             --skill-key (required)  --enabled (required: true|false)
```

### `enter-cli thread --help`

```
chat [options] <project_id>                  -m <text>  --file <path>  --auto-approve
messages [options] <project_id>              --start-turn --end-turn --turn --latest --tail --follow
turns [options] <project_id>                 --latest --wait --timeout
diff [options] <project_id> <turn_number>    --out
cancel <project_id>
tasks <project_id>
task-delete <project_id> <task_id>
task-promote <project_id> <task_id>
restore [options] <project_id>               --turn-id
actions [options] <project_id>               --pending  --ids
approve [options] <project_id> <action_id>   --secret-name --secret-value --tool-result --answers --skip-answers
reject <project_id> <action_id>
```

### `enter-cli ws --help`

```
list
get <workspace_id>
create [options] <name>       --image
delete <workspace_id>
members                       (subgroup: list / add / remove / update-role / leave)
credits                       (subgroup: dashboard)
```

### `enter-cli domain --help`

```
list <project_id>
add [options] <project_id>      --domain (required)
remove [options] <project_id>   --domain (required)
refresh [options] <project_id>  --domain (required)
update [options] <project_id>   --domain (required)  --new-domain
```

### `enter-cli skill --help`

```
list [options]                      List available skills          --role
search <keyword>                    Search skills by keyword
get <slug>                          Get skill details
delete <slug>                       Delete a skill
visibility <slug> <public|private>  Set skill visibility
install [options] <slug>            Install a skill to an agent    --agent (required)  --version
uninstall [options]                 Uninstall a skill from agent   --agent (required)  --skill (required)
agents [options]                    List skills on agents          --agent
```

Note: `skill` commands use `/work/api/v1` (separate from project `/code/api/v1`).

## Command Reference

### `proj`

```bash
enter-cli proj create <ws_id> --name "X" --prompt "..." --wait --timeout 300
enter-cli proj publish <project_id> --wait --timeout 300
enter-cli proj publish-status <project_id>          # publish_status, publish_url, lifecycle_status
enter-cli proj urls <project_id>                    # preview_url, publish_url, recommended_share_url, environment
enter-cli proj plan-mode <project_id> <enable|disable>
enter-cli proj source-code <project_id>
enter-cli proj read-file <project_id> --path src/App.tsx
```

All `proj get` responses now include `lifecycle_status` field:
- `initializing` → `building` → `ready` → `active` (published)
- `failed`, `archived` also possible

### `thread`

```bash
# Messages — --start-turn is now optional
enter-cli thread messages <project_id> --latest              # most recent turn
enter-cli thread messages <project_id> --turn 5              # specific turn
enter-cli thread messages <project_id> --tail 3              # last 3 turns
enter-cli thread messages <project_id> --start-turn 3 --end-turn 5
enter-cli thread messages <project_id> --follow              # real-time WebSocket stream

# Turns
enter-cli thread turns <project_id> --latest
enter-cli thread turns <project_id> --wait --timeout 120     # blocks until terminal status

# Diff
enter-cli thread diff <project_id> <turn_number> --out ./changes.patch

# Gate / Action approval
enter-cli thread actions <project_id> --pending              # only waiting_response
enter-cli thread actions <project_id> --ids <id1,id2>        # filter by IDs
enter-cli thread approve <project_id> <action_id>            # plain approve
enter-cli thread approve <project_id> <action_id> --secret-name <name> --secret-value <val>
enter-cli thread approve <project_id> <action_id> --answers '{"Q": {"selected_options": ["A"], "other_text": ""}}'
enter-cli thread approve <project_id> <action_id> --skip-answers
enter-cli thread reject <project_id> <action_id>
```

### `domain`

```bash
enter-cli domain refresh <project_id> --domain example.com   # re-check DNS/SSL
enter-cli domain update <project_id> --domain old.com --new-domain new.com
```

### `ws members`

```bash
enter-cli ws members update-role <ws_id> --email <email> --role <role>
enter-cli ws members leave <ws_id>
```

## Verified Output Shapes

### `enter-cli whoami -o json`

Returns a single object like:

```json
{
  "user_id": 100000104,
  "email": "user@example.com",
  "status": "active"
}
```

### `enter-cli ws list -o json`

Returns an array, not `{ data: [...] }`:

```json
[
  {
    "id": 10000000054,
    "name": "Workspace Name",
    "plan_type": "free",
    "role": "owner"
  }
]
```

### `enter-cli proj get <project_id> -o json`

Returns an object with a `project` key:

```json
{
  "project": {
    "project_id": "...",
    "name": "Coup Browser AI",
    "status": "active",
    "visibility": "public",
    "preview_url": "https://...preview.enter.pro",
    "publish_url": "https://...prod.enter.pro",
    "publish_commit": "...",
    "build_status": {
      "commit_id": "...",
      "success": true
    }
  }
}
```

### `enter-cli thread turns <project_id> -o json`

Returns an array of turns:

```json
[
  {
    "id": "...",
    "turn": 1,
    "turn_name": "Implement Coup game",
    "status": "completed",
    "credits_consumed": 78.73
  }
]
```

### `enter-cli thread diff <project_id> <turn_number> -o json`

Returns:

```json
{
  "diff": [
    {
      "action": "new",
      "file_path": "src/lib/gameLogic.ts",
      "new_content": "..."
    }
  ]
}
```

### `enter-cli domain list <project_id> -o json`

Returns:

```json
{
  "domains": [
    {
      "status": "ready",
      "type": "default",
      "domain": "...prod.enter.pro",
      "url": "https://...prod.enter.pro"
    }
  ]
}
```

## Verified Command Notes

- `enter-cli proj visibility <project_id> public` works.
- `enter-cli proj create <workspace_id> --name ... --prompt ...` is supported in 0.1.3.
- `enter-cli proj publish <project_id>` exists in 0.1.3.
- `enter-cli thread turns`, `thread diff`, and `domain list` work as expected.
- `enter-cli proj get` is the safest way to choose between preview and production URLs.

## Do Not Trust Old Examples That Assume

- `ws list` returns `.data`
- `proj get` returns `.data`
- `thread turns` returns `.data[-1].turnNumber`

Those shapes were not correct in the verified environment.
