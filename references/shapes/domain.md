# Domain Shapes

## `domain list <project_id> -o json`

Wrapped: `{domains: [...]}`.

```json
{
  "domains": [
    {
      "id": 6675,
      "project_id": "6c441184629d43c1ba4d9ad62ca853ee",
      "status": "ready",
      "type": "default",
      "domain": "6c441184629d43c1ba4d9ad62ca853ee.prod.enter.pro",
      "url": "https://6c441184629d43c1ba4d9ad62ca853ee.prod.enter.pro",
      "info": null,
      "created_at": 1776676679,
      "updated_at": 1776676679
    }
  ]
}
```

- `id` is an integer.
- `created_at` / `updated_at` are **unix seconds** integers, NOT ISO strings (different from every other endpoint).
- `type`: `default` (auto-assigned `*.prod.enter.pro`) | `custom` (CNAME).
- `status`: `ready` | `pending` | `dns_pending` | `ssl_pending` | `failed`.
- `info` is `null` for default domains; for custom domains contains DNS/SSL diagnostic data:
  ```json
  { "info": { "cname_target": "...", "ssl_issued": true, "last_check": "..." } }
  ```

## `domain add <project_id> --domain <name> -o json`

```json
{
  "id": 6676,
  "project_id": "...",
  "status": "dns_pending",
  "type": "custom",
  "domain": "example.com",
  "url": "https://example.com",
  "info": { "cname_target": "...prod.enter.pro" },
  "created_at": 1776683000,
  "updated_at": 1776683000
}
```

The user must add a CNAME record pointing at `info.cname_target` before the domain becomes `ready`. Use `domain refresh` after configuring DNS.

## `domain remove <project_id> --domain <name> -o json`

```json
{ "removed": true, "domain": "example.com" }
```

## `domain refresh <project_id> --domain <name> -o json`

Re-checks DNS and SSL. Returns the updated domain object (same shape as `domain add`).

## `domain update <project_id> --domain <name> [--new-domain <name>] -o json`

Renames an existing custom domain.

```json
{
  "id": 6676,
  "domain": "www.example.com",
  "updated_at": 1776683500
}
```
