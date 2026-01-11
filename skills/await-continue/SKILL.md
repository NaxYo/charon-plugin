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

3. **Start the wait command** (it will output the webhook URL to stderr):
   ```bash
   npx charon-hooks --wait <uuid> --trigger <trigger-id> &
   ```
   Output (stderr):
   ```
   [charon] Webhook URL: http://localhost:3000/api/webhook/<trigger-id>/<uuid>
   [charon] Waiting for webhook...
   ```

4. **Tell the user** the webhook URL and what event will resume execution

5. When the webhook fires, the wait command outputs the **description to stdout** and exits

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
2. Start wait: `npx charon-hooks --wait $uuid --trigger github-pr-reviewed &`
3. The command prints the webhook URL - configure GitHub to POST there
4. The wait command blocks until the webhook fires
5. When PR is reviewed, the wait command outputs the description and exits

## Key Points

- `--service status` returns JSON with `webhook_base` URL (uses tunnel if active)
- `--wait` prints the full webhook URL to stderr automatically
- No need to read config files or construct URLs manually
