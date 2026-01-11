---
name: charon-await
description: Wait for external events (webhooks) and continue execution with the result. Use when the user wants to pause and wait for something external like PR reviews, CI completion, approvals, or any webhook-based event.
---

# Await External Events with Charon

When you need to wait for an external event before continuing:

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

## Steps

1. **Generate a UUID** for this wait operation (use `uuidgen` or generate one)

2. **Create/update trigger** in `~/.charon/config/triggers.yaml`:
   ```yaml
   triggers:
     - id: <descriptive-trigger-id>
       name: "Wait for <event>"
       type: webhook
       enabled: true
       sanitizer: passthrough  # or custom sanitizer
       template: "<template using webhook payload>"
       egress: cli
       context:
         cli_template: "npx charon-hooks --resolve {promise_uuid} --description '{description}'"
   ```

3. **Start the wait command as a background task** using Bash tool's `run_in_background: true`:
   ```bash
   npx charon-hooks --wait <uuid> --trigger <trigger-id>
   ```

   **CRITICAL:**
   - Use the Bash tool with `run_in_background: true` parameter - do NOT use shell `&`
   - Do NOT add any sleep commands or additional waiting logic after this command
   - The subprocess will appear in Claude Code's footer immediately when run correctly

   Output (stderr):
   ```
   [charon] Webhook URL: http://localhost:3000/api/webhook/<trigger-id>/<uuid>
   [charon] Waiting for webhook...
   ```

4. **Tell the user** the webhook URL and what event will resume execution

5. **Stop and wait for the user** - do not continue working until the webhook fires

6. When the webhook fires, the wait command outputs the **description to stdout** and exits

## Cleanup

- If the operation is cancelled, kill the `--wait` process (promise auto-cleans)
- Optionally remove the trigger from triggers.yaml if no longer needed

## Example: Wait for PR Review

```yaml
# Add to ~/.charon/config/triggers.yaml
triggers:
  - id: github-pr-reviewed
    name: "PR Review Received"
    type: webhook
    enabled: true
    sanitizer: passthrough
    template: "PR review: {action} by {review.user.login} - {review.state}"
    egress: cli
    context:
      cli_template: "npx charon-hooks --resolve {promise_uuid} --description '{description}'"
```

Then:
1. Generate UUID: `uuid=$(uuidgen)`
2. Start wait with `run_in_background: true`: `npx charon-hooks --wait $uuid --trigger github-pr-reviewed`
3. The command prints the webhook URL - configure GitHub to POST there
4. Tell the user the URL and stop - do NOT add sleep commands or continue working
5. When PR is reviewed, the wait command outputs the description and exits

## Key Points

- `--service status` returns JSON with `webhook_base` URL (uses tunnel if active)
- `--wait` prints the full webhook URL to stderr automatically
- No need to read config files or construct URLs manually

## Troubleshooting

### Subprocess not appearing in footer immediately?

If the wait command doesn't show as a subprocess in Claude Code's footer:
- You MUST use Bash tool's `run_in_background: true` parameter
- Do NOT use shell `&` to background the command
- Do NOT add `sleep` commands after launching the wait command
- The subprocess should appear immediately - if it doesn't, re-run with correct parameters

### Service won't start?

```bash
# Run in foreground to see actual errors:
npx charon-hooks

# Check if port 3000 is already in use:
lsof -i :3000
```

### "Connection refused" or "fetch failed"?

The `--wait` command automatically retries for ~5 seconds if the service isn't ready yet.
If it still fails:

```bash
# Verify service is running:
npx charon-hooks --service status

# Should return JSON like: {"running": true, "port": 3000, ...}
# If running is false, start the service first.
```

### Quick health check

```bash
# This should return JSON, not HTML:
curl http://localhost:3000/api/promise

# Expected: {"promises": [...]}
```

### Service starts but webhook doesn't work?

1. Check the trigger exists in `~/.charon/config/triggers.yaml`
2. Verify the trigger ID matches what you're using
3. Check service logs by running in foreground: `npx charon-hooks`
