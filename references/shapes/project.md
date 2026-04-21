# Project Shapes

## `proj list <workspace_id> -o json`

Wrapped: `{items: [...], total, page, page_size}`.

```json
{
  "items": [
    {
      "project_id": "6c441184629d43c1ba4d9ad62ca853ee",
      "name": "2048 Game",
      "status": "active",
      "visibility": "private",
      "preview_url": "https://....preview.enter.pro",
      "updated_at": "2026-04-20T11:05:34.661932Z"
    }
  ],
  "total": 65,
  "page": 1,
  "page_size": 20
}
```

- `status`: `active` | `initializing` | `archived` | etc.
- `visibility`: `public` | `private`.
- This is a **summary** shape — many fields available in `proj get` are absent here.

## `proj get <project_id> -o json`

Wrapped: `{project: {...}, lifecycle_status: "..."}`.

```json
{
  "project": {
    "project_id": "9573a07ff41a401f855ba87074776d53",
    "name": "Test Game",
    "status": "active",
    "visibility": "private",
    "release_id": "",
    "remixed": 0,
    "parent_project_id": "",
    "parent_release_id": "",
    "template_hash": "76ea49dc69a3c1051e71062567d0eab0ad656070",
    "user_id": 100000001,
    "cover": "cover_9573a07ff41a401f855ba87074776d53_5178df0",
    "category": "",
    "commit": "7c2b09dbdd24b0489b527c203f2ba4d65253518d",
    "commit_turn": 3,
    "commit_first_turn": 3,
    "publish_commit": "",
    "publish_turn": 0,
    "publish_url": "",
    "thread_id": "157a6d92-1ebb-470b-a218-cb7671b9efb2",
    "agent_config_hash": "97f97c74...",
    "github_repo_full_name": "",
    "is_template": false,
    "is_community": false,
    "feature": {
      "has_supabase_integration": false,
      "secret_keys": null,
      "selected_model": "auto"
    },
    "auto_task_queue": false,
    "plan_mode": false,
    "workspace_id": 10000012246,
    "ai_capability_enabled": false,
    "ai_connection_state": "never_enabled",
    "enter_ai_credits_status": "disabled",
    "created_at": "2026-04-20T10:38:27.813694Z",
    "updated_at": "2026-04-20T11:15:34.928566Z",
    "last_active_at": "2026-04-20T11:09:43.364566Z",
    "template_updated_at": "0001-01-01T00:00:00Z",
    "build_usage_credit_cents": 1169,
    "enter_ai_usage_credit_cents": 0,
    "supabase": {},
    "google_analysis_connected": false,
    "stripe_connected": false,
    "stripe_secret_key_type": "",
    "preview_url": "https://....preview.enter.pro",
    "github_url": "",
    "user_name": "Zhu Hang",
    "user_photo_url": "https://lh3.googleusercontent.com/...",
    "public_user_id": "183958a4-...",
    "is_favorited": false,
    "github_branch": "",
    "github_sync_status": "",
    "zero_one_thread_id": "",
    "zero_one_agent_config_hash": "",
    "ai_all_status": "never",
    "publish_status": {
      "unpublished_changes": 2,
      "unpublished_commit_ids": ["..."],
      "last_published_commit_id": ""
    },
    "build_status": {
      "commit_id": "7c2b09dbdd24b0489b527c203f2ba4d65253518d",
      "success": true
    },
    "project_role": "owner"
  },
  "lifecycle_status": "unknown"
}
```

Critical fields:
- `commit` — current build commit (latest turn that produced code).
- `commit_turn` — turn number that produced `commit`.
- `publish_commit` — last successfully published commit. Empty `""` if never published.
- `publish_status.last_published_commit_id` — same info, current source of truth (older `publish_commit` field can lag).
- `publish_status.unpublished_changes` — number of commits ahead of last publish.
- `publish_url` — empty string if not published; otherwise the production URL.
- `preview_url` — always present (sandbox preview).
- `lifecycle_status` (top-level): `unknown` | `ready` | `active` | `initializing` | `failed` | etc.
- `build_status.success` — last build outcome.
- `template_updated_at: "0001-01-01T00:00:00Z"` is the zero value, treat as "never".
- `feature.selected_model` — only appears after `proj model` is set.

## `proj create <workspace_id> --prompt <text> [--name <name>] [--wait] -o json`

```json
{
  "project_id": "9573a07ff41a401f855ba87074776d53",
  "task_id": "..."
}
```

Asynchronous by default: returns `task_id` immediately. **Always follow with `thread wait <project_id>`.**

`--wait` makes the command block until the first build completes (with `--timeout <seconds>`, default 300). Without `--wait`, the project is still being initialized when the call returns.

## `proj rename <project_id> <new_name> -o json`

```json
{ "project_id": "9573a07ff41a401f855ba87074776d53", "name": "Test Game" }
```

## `proj visibility <project_id> <public|private> -o json`

```json
{ "project_id": "9573a07ff41a401f855ba87074776d53", "visibility": "public" }
```

## `proj plan-mode <project_id> <enable|disable> -o json`

Returns the **full project object** (same shape as `proj get`'s `.project`, but flattened — no outer `{project: ...}` wrapper). Inspect `plan_mode: true|false` to confirm.

## `proj model <project_id> <model> -o json`

Wrapped: `{project: {...}, selected_model: "<model>"}`. Inspect `selected_model` to confirm.

## `proj delete <project_id> -o json`

```json
{ "deleted": true, "project_id": "9573a07ff41a401f855ba87074776d53" }
```

## `proj urls <project_id> -o json`

```json
{
  "preview_url": "https://....preview.enter.pro",
  "preview_reachable": true,
  "publish_url": "https://....prod.enter.pro",
  "publish_reachable": true,
  "last_published_commit_id": "17bf3e53...",
  "last_published_at": "2026-04-20T09:21:13.758545Z",
  "recommended_share_url": "https://....prod.enter.pro",
  "environment": "production",
  "visibility": "private"
}
```

If never published:

```json
{
  "preview_url": "https://....preview.enter.pro",
  "preview_reachable": true,
  "publish_url": null,
  "publish_reachable": false,
  "last_published_commit_id": null,
  "last_published_at": null,
  "recommended_share_url": "https://....preview.enter.pro",
  "environment": "preview",
  "visibility": "private"
}
```

- `recommended_share_url` always picks the best available URL — use this when sharing with users.
- `*_reachable` is a real `200 OK` HTTP probe done by the CLI.
- `environment`: `production` | `preview`.

## `proj publish-status <project_id> -o json`

```json
{
  "status": "active",
  "lifecycle_status": "active",
  "published": true,
  "unpublished_changes": 0,
  "last_published_commit_id": "17bf3e53...",
  "last_published_at": "2026-04-20T09:21:13.758545Z",
  "publish_url": "https://....prod.enter.pro",
  "preview_url": "https://....preview.enter.pro",
  "publish_commit": "17bf3e53...",
  "updated_at": "2026-04-20T11:05:34.661932Z"
}
```

- `published: true` means at least one commit has been published.
- `unpublished_changes: 0` means current build commit matches last published commit.
- `lifecycle_status` enum: `ready` | `active` | `initializing` | `failed`.
- For an **unpublished** project: `published: false`, `last_published_commit_id: null`, `last_published_at: null`, `publish_url: ""`, `publish_commit: ""`, `lifecycle_status: "ready"`.

## `proj publish <project_id>`

Synchronous. Streams progress text to stderr; on success prints the same `{project: ..., lifecycle_status: ...}` shape as `proj get` to stdout.

```
Waiting for publish to complete...

Published commit 17bf3e5 at 2026-04-20T11:20:42.679586Z. URL: https://....prod.enter.pro (200 OK)
{ "project": {...}, "lifecycle_status": "active" }
```

Exit codes:
- `0` — published, URL probe returned 200.
- non-zero — failed (timeout, build error, URL probe failed).

## `proj source-code <project_id> -o json`

```json
{
  "commit": "7c2b09dbdd24b0489b527c203f2ba4d65253518d",
  "source_code": [
    {
      "name": "package.json",
      "content": "{\n  \"name\": \"thread\",\n  ...}",
      "is_binary": false,
      "is_oversized": false
    }
  ]
}
```

- `name` is the path relative to project root.
- `content` is the raw file content (string). For binary files, `content` is empty and `is_binary: true`.
- `is_oversized: true` for files exceeding the inline size limit (content omitted).

## `proj download <project_id> --out <output.zip>`

Streams a ZIP file to disk. No JSON output. Exits non-zero on plan limits with `[VIP_REQUIRED]` and a hint pointing at `proj source-code` as fallback.

## `proj edit-file <project_id> --path <path> --code <content> --commit <commit_id> -o json`

Creates a new turn that directly writes the given content to the given path. `--commit` should be the current `commit` from `proj get` (used as the parent).

Returns the same wrapped shape as `proj get` after the edit lands.

## `proj remix <project_id> --workspace-id <ws_id> -o json`

```json
{
  "project_id": "<new_project_id>",
  "task_id": "..."
}
```

Same async behavior as `proj create` — follow with `thread wait`.
