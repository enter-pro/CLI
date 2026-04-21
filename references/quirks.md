# Known Quirks

These are the behaviors most likely to break agent workflows.

## 1. Publish Can Be Slow Or Non-Responsive

`enter-cli proj publish <project_id>` is synchronous: it prechecks `build_status.commit_id`, polls until `publish_status.status == "success"` and `publish_url` is set, then verifies the URL returns `200 OK`. The command exits non-zero on failure. Tune the wait window with `--timeout <seconds>` (default 300).

## 2. Build State Can Reach Messages Before `proj get`

The thread event stream surfaces build completion slightly earlier than project metadata, so a freshly-completed turn may briefly coexist with empty `commit` / `publish_commit` in `proj get`.

Use `enter-cli thread wait <project_id>` instead of polling manually: it waits for the latest turn to reach a terminal state and returns one of three structured outcomes (`completed`, `blocked`, `failed`). It does not wait for `commit_id` to be filled — many turns (config-only changes, gate approvals) never produce a commit. Code that needs a committed build (`proj publish`) checks that itself.

`thread turns` remains a pure list and never blocks.

## 3. Gate / Action Blocking

A `completed` turn does not mean the work is done. The agent often finishes its turn by emitting a tool action that is **waiting for human approval** — the project then sits idle until you approve.

`thread wait` detects this and exits with `status: "blocked"`, listing each pending action plus the exact `approve_command` to run:

```json
{
  "status": "blocked",
  "actions": [{
    "action_id": "...",
    "tool_name": "supabase_enable",
    "input_kind": "none",
    "approve_command": "enter-cli thread approve <pid> <action_id>",
    "instructions": "No input required. Run the approve_command as-is."
  }]
}
```

Dispatch by `input_kind`:

- `none` — run `approve_command` as-is
- `secret` — ask the user for the secret name/value, fill placeholders, then run
- `questions` — read the question payload from `thread messages`, forward to user, pass answers via `--answers '<json>'` or `--skip-answers`

`thread approve <project_id>` (no action_id) auto-resolves when there is exactly one pending action. With multiple pending, pass the action_id explicitly.

After approving, run `thread wait` again. Repeat until `status: "completed"`.

## 4. `thread chat` Rejects Input — `[1001] invalid chat task input`

Most common cause: shell mangles `-m` input when the message contains quotes, newlines, or backslashes. CLI now prints this hint inline; the fix is to use `--file`:

```bash
echo "Long message with 'quotes' and ${vars}..." > /tmp/msg.txt
enter-cli thread chat <project_id> --file /tmp/msg.txt
```

Less common 1001 causes: turn still running, or project not ready — check `thread turns` / `proj get`.

## 5. Source Download May Be Blocked By Plan

On free plans, `proj download` exits with `[VIP_REQUIRED]` and CLI prints a hint pointing at the two fallbacks: `thread diff <project_id> <turn>` for per-turn changes, or `proj source-code <project_id>` for the full inline source. Use them as substitutes; don't claim the zip was produced. Tell the user the restriction is plan-based, not a local error.
