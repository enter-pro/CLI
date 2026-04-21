# Verified Output Shapes

Real JSON shapes returned by `enter-cli <cmd> -o json`. Use these to write correct `jq` paths instead of guessing.

Split by command group:

- [shapes/auth.md](shapes/auth.md) — `whoami`, `config`, `login`, `logout`
- [shapes/workspace.md](shapes/workspace.md) — `ws list/get/create/delete`, `ws members`, `ws credits`
- [shapes/project.md](shapes/project.md) — `proj list/get/create/rename/delete/download/publish/publish-status/urls/remix/visibility/plan-mode/source-code/edit-file/model`
- [shapes/thread.md](shapes/thread.md) — `thread chat/messages/turns/wait/diff/cancel/restore/actions/approve/reject`
- [shapes/domain.md](shapes/domain.md) — `domain list/add/remove/refresh/update`
- [shapes/mcp-skills.md](shapes/mcp-skills.md) — `proj mcp`, `proj skills`

## Shared Conventions

- The CLI strips the API envelope (`{code, message, detail, data}`) and prints only the inner payload. You will not see `code` / `data` in CLI output.
- Some commands return naked top-level objects (`whoami`, `config list`). Others wrap in a one-key object (`proj get → {project: {...}}`). The relevant sub-file always notes the outer shape.
- Timestamps come in two flavors: ISO strings (`2026-04-20T11:09:43.364566Z`) for most fields, and unix seconds (integer) for `domain.created_at` / `domain.updated_at`. Don't assume one or the other.
- Boolean-shaped flags use real booleans, not 0/1.
