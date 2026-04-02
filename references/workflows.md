# Safer Workflows

## 1. Create and Build a Project

Start with:

```bash
enter-cli whoami -o json
enter-cli ws list -o json
```

Preferred path (with polling until build completes):

```bash
enter-cli proj create <workspace_id> --name "My App" --prompt "Build ..." --wait -o json
```

Without `--wait`, the CLI returns immediately and you must poll manually:

```bash
enter-cli proj get <project_id> -o json | jq '{status, lifecycle_status}'
```

## 2. Monitor a Build

Block until a turn completes:

```bash
enter-cli thread turns <project_id> --wait --timeout 120 -o json
```

Or poll manually:

```bash
enter-cli thread turns <project_id> --latest -o json
enter-cli proj get <project_id> -o json | jq '{status, lifecycle_status}'
```

Interpretation:

- turn still running + preview 404: keep waiting
- turn complete + preview 200: preview is ready
- `publish_url` exists + production 200: share production link

If the turn never advances, check for gate actions (see Workflow 7).

## 3. Publish a Project

With wait:

```bash
enter-cli proj publish <project_id> --wait --timeout 180 -o json
```

Then verify:

```bash
enter-cli proj urls <project_id> -o json | jq '{share: .recommended_share_url, env: .environment}'
```

Treat `publish_url` plus `200 OK` as the real publish confirmation.

## 4. Inspect What the Agent Changed

Use:

```bash
enter-cli thread turns <project_id> -o json
enter-cli thread diff <project_id> <turn_number> -o json
```

This is the safest fallback when source download is blocked.

## 5. Choose the Right Link for the User

Fetch:

```bash
enter-cli proj get <project_id> -o json
```

Share links in this order:

1. `publish_url` if non-empty
2. otherwise `preview_url`, labeled clearly as preview

## 6. Source Export Fallback on Free Plan

If `proj download` fails with `VIP_REQUIRED`, use free alternatives:

```bash
# Option A: full source tree
enter-cli proj source-code <project_id> -o json

# Option B: per-turn diff
enter-cli thread diff <project_id> <turn_number> --out ./changes.patch

# Summarize changed files
enter-cli thread diff <project_id> <turn_number> -o json | jq -r '.diff[] | [.action, .file_path] | @tsv'
```

## 7. Handle Gate (Project Stuck Waiting for Approval)

```bash
# Step 1: find pending gates
enter-cli thread actions <project_id> --pending -o json

# Step 2: check tool_name field in the action, then:

# supabase_enable / enable_ai_capability / confirm_skill / stripe_create_products_and_prices
enter-cli thread approve <project_id> <action_id>

# supabase_add_secret
enter-cli thread approve <project_id> <action_id> \
  --secret-name DB_URL --secret-value "postgres://user:pass@host/db"

# stripe_enable
enter-cli thread approve <project_id> <action_id> \
  --secret-name STRIPE_SECRET_KEY --secret-value "sk_live_..."

# ask_user_question — answer the questions
enter-cli thread approve <project_id> <action_id> \
  --answers '{"Which database?": {"selected_options": ["Supabase"], "other_text": ""}}'
# or skip them all
enter-cli thread approve <project_id> <action_id> --skip-answers

# Step 3: if stuck in plan mode (no pending actions but still building)
enter-cli proj plan-mode <project_id> disable
```
