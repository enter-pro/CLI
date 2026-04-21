# Workspace Shapes

## `ws list -o json`

Naked array (no wrapper).

```json
[
  {
    "id": 10000000063,
    "name": "Zhu Hang's Workspace",
    "plan_type": "pro",
    "role": "owner",
    "member_count": 1,
    "subscription_status": "active"
  }
]
```

- `id` is an integer.
- `plan_type`: `free` | `starter` | `pro` | `enterprise`.
- `role`: `owner` | `admin` | `member`.
- `subscription_status`: `active` | `cancelled` | `past_due` | etc.

## `ws get <workspace_id> -o json`

Wrapped: `{workspace: {...}}`.

```json
{
  "workspace": {
    "id": 10000000063,
    "public_id": "30424a7c-27a7-49e8-9242-f7739fbbbd15",
    "name": "Zhu Hang's Workspace",
    "image": "",
    "owner_id": 100000001,
    "auto_transfer_config": null,
    "created_at": "2025-12-17 07:36:08",
    "updated_at": "2026-02-12 08:34:48",
    "plan_type": "pro",
    "entitlement": {
      "plan_type": "pro",
      "name": "Pro",
      "tagline": "...",
      "label": "Most Popular",
      "sort_order": 2,
      "monthly_build_credits": 2000,
      "monthly_ai_credits": 500,
      "daily_credits": 100,
      "max_members": 20,
      "ai_all_trial_limit": -1,
      "ai_all_models": ["anthropic/claude-sonnet-4.5", "openai/gpt-5.2-pro", "..."],
      "granted_features": ["remove_badge", "download_code", "custom_domain", "github_sync", "purchase_credits", "credits_rollover", "enter_ai_all", "invite_members"],
      "credit_allocations": {
        "monthly": [{ "account_type": "credits_balance", "package_id": 170 }],
        "yearly": [{ "account_type": "credits_balance", "package_id": 170 }]
      }
    },
    "subscription_status": "active",
    "billing_cycle": "yearly",
    "subscribed_on": "2026-02-12T16:34:46.989084+08:00",
    "current_period_end": "2027-02-12T16:32:43+08:00",
    "cancel_at_period_end": false,
    "credits_balance": {
      "total": 846.02,
      "breakdown": { "monthly": 0, "daily": 0, "purchase": 0, "bonus": 846.02 },
      "status": "normal"
    },
    "enter_ai_balance": {
      "total": 1983.52,
      "breakdown": { "monthly": 0, "transfer": 0, "gift": 1983.52 },
      "status": "normal"
    }
  }
}
```

Notes:
- `created_at` / `updated_at` here are space-separated, not ISO. (Inconsistent with `whoami`.)
- `entitlement.ai_all_trial_limit: -1` means unlimited.
- `granted_features` enum strings; check membership before claiming a capability is available.
- `credits_balance.status`: `normal` | `low` | `exhausted`.

## `ws create <name> -o json`

Returns the new workspace object (same outer shape as `ws get`'s `workspace` value, plus `workspace_id`):

```json
{
  "workspace_id": 10000012247,
  "name": "New WS",
  "plan_type": "free",
  "...": "..."
}
```

## `ws delete <workspace_id> -o json`

```json
{ "deleted": true, "workspace_id": 10000012247 }
```

## `ws members list <workspace_id> -o json`

Naked array.

```json
[
  {
    "user_id": 100000001,
    "email": "you@example.com",
    "name": "Your Name",
    "role": "owner",
    "created_at": "2025-12-17 07:36:08"
  }
]
```

## `ws members add <workspace_id> --email <email> --role <role> -o json`

`--role` enum: `owner` | `admin` | `editor` | `viewer`.

```json
{ "invited": true, "email": "new@example.com", "role": "editor" }
```

## `ws members remove <workspace_id> [--email <email> | --user-id <id>] -o json`

Identify the target by either email or user_id (exactly one required).

```json
{ "removed": true, "user_id": 100000002 }
```

## `ws members update-role <workspace_id> [--email <email> | --user-id <id>] --role <role> -o json`

```json
{ "user_id": 100000002, "role": "admin" }
```

## `ws members leave <workspace_id> -o json`

```json
{ "left": true, "workspace_id": 10000000063 }
```

## `ws credits dashboard <workspace_id> -o json`

```json
{
  "auto_transfer": { "enabled": false, "threshold": 0, "amount": 0 },
  "credits_balance": {
    "total": 846.02,
    "breakdown": { "monthly": 0, "daily": 0, "purchase": 0, "bonus": 846.02 },
    "status": "normal"
  },
  "enter_ai_balance": {
    "total": 1983.52,
    "breakdown": { "monthly": 0, "transfer": 0, "gift": 1983.52 },
    "status": "normal"
  },
  "usage_history": {
    "projects": [
      {
        "project_id": "6c441184629d43c1ba4d9ad62ca853ee",
        "name": "2048 Game",
        "cover": "cover_6c441184629d43c1ba4d9ad62ca853ee_6126154",
        "preview_url": "https://....preview.enter.pro",
        "last_active_at": "2026-04-20T09:53:38.626329Z",
        "build_usage": 42.96,
        "enter_ai_usage": 0,
        "enter_ai_credits_status": "disabled"
      }
    ],
    "total": 65,
    "page": 1,
    "page_size": 8
  }
}
```

Notes:
- `usage_history.projects` is paginated (default 8 per page); use API directly with `?page=` for more.
- `build_usage` / `enter_ai_usage` are floats (credit cents → credits).
