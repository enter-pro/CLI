# Known Quirks

These are the behaviors most likely to break agent workflows.

## CLI / API Drift

### `proj create` was fixed in 0.1.3

Earlier CLI builds could fail with:

```text
Error: [400] prompt is required
```

In 0.1.3, `proj create --help` includes `--prompt <text>`, so the preferred path is now:

```bash
enter-cli proj create <workspace_id> --name "My App" --prompt "Build ..."
```

Keep the old direct API fallback in mind only for older CLI versions or future regressions.

## Publish Can Be Slow Or Non-Responsive

`enter-cli proj publish <project_id>` now exists, but the publish endpoint may not return promptly.

Safe rule:

1. Confirm `project.commit` or `build_status.commit_id` is non-empty in `proj get`
2. Trigger `proj publish`
3. Poll `proj get`
4. Confirm `publish_url` exists
5. Confirm the production URL returns `200 OK`

Do not assume that a hanging or slow publish call means the publish failed.

## Build State Can Reach Messages Before `proj get`

The thread event stream can surface build completion slightly earlier than project metadata.

Observed pattern:

- `thread messages` already contains `build_end`
- `build_end.commit_id` is visible in the thread
- `proj get` still briefly shows empty `commit` / `publish_commit`

Safe rule:

- treat `thread messages` as an early signal
- use `proj get` as the publish gate
- if `build_end` exists but `proj get` still has empty `commit`, wait and retry instead of declaring failure

## Thread Message Parameter Drift (fixed post-0.1.3)

`--start-turn` is no longer required. Use shorthand flags instead:

```bash
enter-cli thread messages <project_id> --latest -o json     # most recent turn
enter-cli thread messages <project_id> --turn 5 -o json     # specific turn
enter-cli thread messages <project_id> --tail 3 -o json     # last 3 turns
```

If you must use raw parameters, both `--start-turn` and `--end-turn` are still supported.

## Gate / Action Blocking

Projects can stall waiting for human approval of tool actions. Symptoms:
- Thread turn stuck in `running` state
- No new messages after agent announces a feature (e.g. "I'll set up Supabase")

**Resolution workflow:**

```bash
# 1. Check for pending gates
enter-cli thread actions <project_id> --pending -o json

# 2. Inspect the action's tool_name field, then approve accordingly:

# supabase_enable / stripe_create_products_and_prices / enable_ai_capability / confirm_skill
enter-cli thread approve <project_id> <action_id>

# supabase_add_secret (supply the secret)
enter-cli thread approve <project_id> <action_id> --secret-name DB_URL --secret-value "postgres://..."

# stripe_enable (supply Stripe key)
enter-cli thread approve <project_id> <action_id> --secret-name STRIPE_SECRET_KEY --secret-value "sk_..."

# ask_user_question (answer or skip)
enter-cli thread approve <project_id> <action_id> --answers '{"Which DB?": {"selected_options": ["Supabase"], "other_text": ""}}'
enter-cli thread approve <project_id> <action_id> --skip-answers
```

## Plan Mode Lock

If `proj get` shows `lifecycle_status: building` but no progress for several minutes:

```bash
enter-cli proj plan-mode <project_id> disable
```

This unblocks agent-side plan mode gates. Re-enable with `enable` if needed.

## `thread chat` Can Reject Follow-Up Input

Observed failure:

```text
Error: [1001] invalid chat task input
```

**Most common cause:** Long messages or messages with special characters (quotes, newlines, backslashes) passed directly via `-m` get mangled by the shell before reaching the CLI.

**Fix — write message to a temp file and use `--file`:**

```bash
cat > /tmp/msg.txt << 'EOF'
Build a full game with PeerJS for multiplayer...
Include all these features:
1. Landing page
2. Lobby
...
EOF
enter-cli thread chat <project_id> --file /tmp/msg.txt
```

`--file` reads the content directly from disk, bypassing all shell quoting issues. Use it for any message longer than ~200 characters or containing special characters.

Other causes (less common):
- Turn is still running — inspect `thread turns` and wait for completion
- Project not in a ready state — inspect `proj get`

## Preview vs Production

- `preview_url` may exist before the artifact is ready
- preview can return `404 Not Found` while the build is still running
- `publish_url` may appear later than `preview_url`
- `visibility public` does not guarantee `publish_url` is already live

Safe rule:

- poll `proj get`
- prefer `publish_url`
- use `preview_url` only as fallback, labeled as preview
- after `proj publish`, verify the production host directly

## Plan Gates

### Source download may be blocked

Observed API response for project download on free plan:

```json
{
  "code": "VIP_REQUIRED",
  "data": {
    "current_plan": "free",
    "feature": "download_code"
  }
}
```

If this happens:

- do not claim the source was downloaded
- export `thread diff` JSON instead
- tell the user the restriction is plan-based, not a local error

## Sparse Task Payloads

`enter-cli thread tasks -o json` may return very little metadata. Do not assume `status` or `content` will always be populated.

Use `thread turns`, `thread messages`, and URL reachability checks together when monitoring progress.
