---
name: charon-workflow
description: Set up event-triggered Claude Code tasks. Use when the user wants automated responses to external events like "when X happens, do Y".
---

# Event-Triggered Workflows with Charon

Set up workflows where external events automatically trigger new Claude Code sessions.

## Prerequisites - Ensure Charon is Running

**IMPORTANT:** All `npx charon-hooks` commands work from ANY directory. Do NOT cd anywhere.

1. Check if Charon is running:
   ```bash
   npx charon-hooks --service status
   ```
   This outputs JSON: `{"running": true/false, "port": ..., "url": ..., "webhook_base": ...}`

2. If not running, start it:
   ```bash
   npx charon-hooks --service start
   ```

## Setting Up a Workflow

Create a trigger in `~/.charon/config/triggers.yaml`:

```yaml
triggers:
  - id: <unique-trigger-id>
    name: "<Descriptive Name>"
    type: webhook
    enabled: true
    sanitizer: passthrough  # or custom sanitizer name
    template: "<Task description template with {placeholders}>"
    egress: cli
    context:
      cli_template: "claude --dangerously-skip-permissions --task '{description}'"
      working_dir: "/path/to/project"  # optional
```

## Template Placeholders

Use `{key}` syntax to interpolate values from the webhook payload:
- `{action}` - GitHub webhook action
- `{issue.title}` - Nested values
- `{description}` - The composed task description

## Example: Deploy on CI Success

```yaml
triggers:
  - id: ci-deploy
    name: "Deploy on CI Success"
    type: webhook
    enabled: true
    sanitizer: passthrough
    template: "CI passed for branch {ref}. Deploy to staging and verify health."
    egress: cli
    context:
      cli_template: "claude --dangerously-skip-permissions --task '{description}'"
      working_dir: "/home/user/myapp"
```

Get the webhook URL from `--service status` output (uses `webhook_base` field):
```
POST <webhook_base>/ci-deploy
Content-Type: application/json

{
  "ref": "main",
  "status": "success"
}
```

## Example: Auto-Fix Failing Tests

```yaml
triggers:
  - id: fix-tests
    name: "Fix Failing Tests"
    type: webhook
    enabled: true
    sanitizer: passthrough
    template: "Tests failed in {repository.full_name}. Error: {message}. Please investigate and fix."
    egress: cli
    context:
      cli_template: "claude --dangerously-skip-permissions --task '{description}'"
      working_dir: "{repository.local_path}"
```

## Custom Sanitizers

For complex payload processing, create a sanitizer in `~/.charon/sanitizers/`:

```javascript
// ~/.charon/sanitizers/github-ci.js
export default function(payload, headers) {
  // Only process successful CI runs
  if (payload.action !== 'completed' || payload.conclusion !== 'success') {
    return null; // Skip this event
  }

  return {
    branch: payload.workflow_run.head_branch,
    repo: payload.repository.full_name,
    run_url: payload.workflow_run.html_url
  };
}
```

Then use it:
```yaml
sanitizer: github-ci
template: "Deploy {repo} branch {branch}. CI run: {run_url}"
```

## Security Note

The `--dangerously-skip-permissions` flag allows Claude to execute without confirmation prompts. Only use this in trusted environments and ensure your webhook endpoints are properly secured.

## Troubleshooting

### Service won't start?

```bash
# Run in foreground to see actual errors:
npx charon-hooks

# Check if port 3000 is already in use:
lsof -i :3000
```

### Webhook not triggering?

1. Verify the trigger exists and is enabled in `~/.charon/config/triggers.yaml`
2. Check the webhook URL is correct (get from `npx charon-hooks --service status`)
3. Test manually:
   ```bash
   curl -X POST http://localhost:3000/api/webhook/<trigger-id> \
     -H "Content-Type: application/json" \
     -d '{"test": "data"}'
   ```
4. Check recent runs: `curl http://localhost:3000/api/runs | jq`

### Template placeholders not replaced?

1. Ensure your sanitizer returns the expected keys
2. Test sanitizer locally by checking run details in the API
3. For `passthrough` sanitizer, keys come directly from the JSON payload

### Sanitizer returning null (event skipped)?

This is intentional - sanitizers return `null` to skip events. If unexpected:
1. Check your sanitizer logic
2. Verify the webhook payload matches what your sanitizer expects

### Quick health check

```bash
# Check service is responding:
curl http://localhost:3000/api/triggers

# Should return JSON with your triggers, not HTML
```
