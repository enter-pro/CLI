# Thread Shapes

## `thread chat <project_id> -m "msg" -o json`

```json
{ "task_id": "94e6ee3a-81fc-42aa-9735-a62451c0f427" }
```

Asynchronous: returns a task_id immediately. **Always follow with `thread wait`.**

For long / quoted / multi-line messages, use `--file <path>` instead of `-m` to avoid shell mangling (gives `[1001] invalid chat task input`).

`--auto-approve` skips manual gate approval for tool actions in this turn (the server auto-approves them). Useful for unattended scripts; never use it for production publishes / secret operations.

## `thread turns <project_id> -o json`

Wrapped: `{items: [...], total}`. Newest turn first.

```json
{
  "items": [
    {
      "id": "94e6ee3a-81fc-42aa-9735-a62451c0f427",
      "turn": 4,
      "turn_name": "Add another change",
      "status": "cancelled",
      "model": "auto",
      "credits_consumed": 0,
      "created_at": "2026-04-20T11:15:32Z"
    },
    {
      "id": "e68bb183-e0fa-4cef-8a1b-44be455ba61e",
      "turn": 3,
      "turn_name": "Add copyright footer",
      "status": "completed",
      "model": "auto",
      "credits_consumed": 5.14,
      "created_at": "2026-04-20T11:14:13Z"
    }
  ],
  "total": 4
}
```

- `turn` is a 1-based integer; use this for `thread diff` and `thread restore --turn`.
- `id` is the turn UUID; rarely needed by the agent.
- `status`: `pending` | `agent_start` | `running` | `completed` | `cancelled` | `error`.
- `credits_consumed` is `0` for cancelled / error turns and for `restore` turns.
- `model` may be missing on older turns.

## `thread wait <project_id> -o json`

Waits for the latest turn to reach a terminal state. Three exit shapes, all with exit code 0.

Streams progress text (`Waiting for turn to complete...`) to **stderr** while polling. The structured result goes to stdout only — pipe stdout to `jq` and ignore stderr.

### `status: "completed"`

```json
{
  "status": "completed",
  "project_id": "9573a07ff41a401f855ba87074776d53",
  "project": { /* same as `proj get`'s .project */ }
}
```

### `status: "blocked"`

The latest turn finished but emitted at least one tool action waiting for human approval.

```json
{
  "status": "blocked",
  "reason": "pending_actions",
  "project_id": "9573a07ff41a401f855ba87074776d53",
  "actions": [
    {
      "action_id": "<uuid>",
      "tool_name": "supabase_enable",
      "turn": 5,
      "input_kind": "none",
      "approve_command": "enter-cli thread approve 9573a07ff41a401f855ba87074776d53 <action_id>",
      "instructions": "No input required. Run the approve_command as-is."
    }
  ]
}
```

`input_kind` enum:
- `none` — run `approve_command` as-is.
- `secret` — ask the user for the secret value, fill placeholders in `approve_command`, then run.
- `questions` — read the question payload from `thread messages`, forward to user, pass answers via `--answers '<json>'` or `--skip-answers`.
- `confirm_plan_mode` — plan mode confirmation (a regular gate, treat as `none`).

### `status: "failed"`

```json
{
  "status": "failed",
  "project_id": "...",
  "turn": 5,
  "reason": "build_error",
  "detail": "..."
}
```

## `thread messages <project_id> --latest|--turn N|--start-turn N|--tail N|--follow -o json`

Wrapped: `{messages: [...]}`.

⚠️ Must specify exactly one of `--latest` / `--turn` / `--start-turn` / `--tail` / `--follow`.

Each message has the same envelope, with type-specific payload in `detail`:

```json
{
  "turn": 3,
  "message_type": "user_message",
  "event_id": "1776683653554-0",
  "timestamp": 1776683653554921700,
  "detail": { /* type-specific */ }
}
```

`timestamp` is unix nanoseconds (integer), not a string.

### Known `message_type` values

| `message_type` | `detail` shape |
|---|---|
| `turn_start` | `null` |
| `user_message` | `{user_message: {content, images, attachments, selected_codes, components, user_id, user_name, user_picture, target_action}}` |
| `agent_run_start` | `{stream_id, run_start: {agent_name, agent_id, agent_run_id}}` |
| `agent_run_end` | `{stream_id, agent_run_end: {agent_name, agent_id, agent_run_id}}` |
| `assistant_reasoning_start` | `{stream_id, assistant_reasoning_start: {agent_name, agent_id, message_id}}` |
| `assistant_reasoning` | `{stream_id, assistant_reasoning: {agent_name, agent_id, message_id, reasoning}}` |
| `assistant_reasoning_end` | `{stream_id, assistant_reasoning_end: {agent_name, agent_id, message_id}}` |
| `assistant_content_start` | `{stream_id, assistant_content_start: {agent_name, agent_id, message_id}}` |
| `assistant_content` | `{stream_id, assistant_content: {agent_name, agent_id, message_id, content}}` |
| `assistant_content_end` | `{stream_id, assistant_content_end: {agent_name, agent_id, message_id}}` |
| `tool_call_start` | `{stream_id, tool_call_start: {agent_name, agent_id, tool_name, tool_call_id, tool_type, tool_call_args}}` |
| `tool_call_arguments` | `{stream_id, tool_call_arguments: {agent_name, agent_id, tool_name, tool_call_id, arguments}}` (`arguments` is a JSON string) |
| `tool_call_end` | `{stream_id, tool_call_end: {agent_name, agent_id, tool_name, tool_call_id, tool_result, full_arguments}}` |
| `build_start` | `null` |
| `build_end` | `{build_end: {name, commit_id, url}}` |
| `turn_end` | `{turn_end: {model, credits_consumed}}` |
| `action_request` | (gate / approval) — `{action_request: {action_id, tool_name, ...}}` |

The visible "assistant said X" sequence is `assistant_content_start` → repeated `assistant_content` chunks → `assistant_content_end`. Concatenate `detail.assistant_content.content` across the chunks for the full message.

`tool_call_arguments.arguments` is a string of streamed JSON deltas; `tool_call_end.full_arguments` is the parsed final object.

## `thread diff <project_id> <turn_number> -o json`

Wrapped: `{diff: [...]}` — array of file changes.

```json
{
  "diff": [
    {
      "action": "edit",
      "file_path": "src/pages/Index.tsx",
      "old_content": "...",
      "new_content": "...",
      "file_type": "tsx",
      "is_image": false,
      "is_binary": false
    }
  ]
}
```

- `action` enum: `edit` | `create` | `delete`.
- For `delete`, `new_content` is absent; for `create`, `old_content` is `""`.
- For binary / image files, `old_content` / `new_content` are empty and `is_binary` / `is_image` is `true`.
- Restore turns produce a normal diff (whatever changed by going back).

Turns with no preview (e.g. cancelled turns, config-only turns) **error out** with `Error: [1001] turn has no preview` and exit non-zero. Don't treat as empty diff.

## `thread cancel <project_id> -o json`

Idempotent: checks for a running turn first.

```json
{ "cancelled": true, "turn": 4 }
```

```json
{
  "cancelled": false,
  "reason": "no_running_turn",
  "latest_turn": 4,
  "latest_status": "cancelled"
}
```

`reason` enum: `no_running_turn` | `no_turns`.

## `thread restore <project_id> --turn <N> -o json`

`--turn` is the 1-based integer from `thread turns`. Creates a new turn (type `restore`).

```json
{
  "succeed": true,
  "turn": {
    "id": "6147f307-a7fd-413e-8c61-7c144b610f89",
    "turn": 5,
    "type": "restore",
    "status": "completed",
    "turn_name": "Add copyright footer",
    "commit_id": "7c2b09dbdd24b0489b527c203f2ba4d65253518d",
    "commit_first_turn": 3,
    "start_stream_id": "1776683995263-0",
    "end_stream_id": "1776683995569-0",
    "has_action": false,
    "bookmarked": false,
    "created_at": "2026-04-20T11:19:55.261061079Z",
    "updated_at": "2026-04-20T11:19:55.587202695Z"
  }
}
```

## `thread actions <project_id> -o json`

Wrapped: `{actions: [...], total}`.

```json
{ "actions": [], "total": 0 }
```

Add `--pending` to filter to actions still waiting for response. Action object (after CLI normalizes PascalCase → snake_case):

```json
{
  "action_id": "<uuid>",
  "tool_name": "supabase_enable",
  "turn": 5,
  "status": "waiting_response",
  "created_at": "...",
  "input": { /* tool-specific */ }
}
```

`status` enum: `waiting_response` | `approved` | `rejected` | `expired`.

## `thread approve <project_id> [action_id] -o json`

`action_id` optional — auto-resolves when exactly one pending action exists. With multiple pending, must pass explicitly.

Tool-specific flags (used when the pending action's `tool_name` requires input):
- `supabase_add_secret` / `stripe_enable`: `--secret-name <name> --secret-value <value>`
- `ask_user_question`: `--answers '<json>'` or `--skip-answers`
- All others (`stripe_create_products_and_prices`, `enable_ai_capability`, `confirm_skill`, `confirm_plan_mode`): no flags needed.

Default approve (most tools — posts to `/thread/chat` with `action_response`):

```json
{ "approved": true, "action_id": "<uuid>", "turn": 5 }
```

`supabase_enable` uses a dedicated endpoint (`/entercloud/enable`) instead of `/thread/chat`. Same `enter-cli thread approve <pid> <aid>` invocation, but the response surfaces the supabase binding info and the server auto-advances the turn:

```json
{
  "approved": true,
  "action_id": "<uuid>",
  "tool_name": "supabase_enable",
  "response": {
    "binding": { "id": 7134, "instance_id": 7134, "project_id": "...", "setup_completed": true },
    "instance": {
      "id": 7134,
      "workspace_id": 10000012246,
      "cloud_ref": "spb-...",
      "provider": "aliyun_supabase",
      "status": "active",
      "api_url": "spb-....supabase.opentrust.net",
      "anon_key": "eyJ..."
    },
    "is_new_instance": true
  }
}
```

Agents don't need to know about the dispatch — calling `enter-cli thread approve` works the same way for every tool.

Idempotent no-op (exit 0) when nothing is pending:

```json
{
  "approved": false,
  "reason": "no_pending_actions",
  "project_id": "9573a07ff41a401f855ba87074776d53"
}
```

Ambiguity is a real error (exit non-zero) — when 2+ actions are pending and no `action_id` was passed:

```json
{
  "approved": false,
  "reason": "ambiguous_pending_actions",
  "project_id": "...",
  "pending": [
    { "action_id": "<uuid>", "tool_name": "supabase_enable", "turn": 5 },
    { "action_id": "<uuid>", "tool_name": "stripe_enable", "turn": 5 }
  ]
}
```

## `thread reject <project_id> [action_id] -o json`

```json
{ "rejected": true, "action_id": "<uuid>", "turn": 5 }
```

Same idempotent no-op + ambiguity shapes as `thread approve` (with `rejected` instead of `approved`).
