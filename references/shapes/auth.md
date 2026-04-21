# Auth & Config Shapes

## `whoami -o json`

Naked object (no wrapper).

```json
{
  "user_id": 100000001,
  "external_id": "google-oauth2|...",
  "public_user_id": "183958a4-a2f3-419d-bf42-ad98de74079c",
  "email": "you@example.com",
  "name": "Your Name",
  "picture": "https://lh3.googleusercontent.com/...",
  "status": "active",
  "admin": true,
  "max_agent": 10,
  "referral_code": "r1.96ddc3eb",
  "active_code": "",
  "description": "",
  "link": "",
  "x": "",
  "preferences": { "tool_preferences": null },
  "created_at": "2025-08-24T14:17:50.317612Z",
  "updated_at": "2026-01-06T05:48:25.273761Z"
}
```

Notes:
- `user_id` is an integer, not a string.
- `picture` may be empty string for users without an avatar.
- `preferences.tool_preferences` is `null` for users that haven't customized.

## `config list -o json`

```json
{
  "api_url": "https://api.enter.pro",
  "base_path": "/code/api",
  "output": "json",
  "default_workspace": ""
}
```

`default_workspace` is `""` if not set, otherwise the workspace ID as string.

## `config get <key>`

⚠️ Ignores `-o json`. Always prints `key: value` text.

## `config set <key> <value>`

Prints `Set <key> = <value>` text. No JSON output.

## `login`

Interactive browser flow. No JSON output.

## `logout`

Prints `Logged out.` text. No JSON output.
