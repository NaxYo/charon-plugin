---
name: charon-await
description: Wait for external events (webhooks) and continue execution with the result. Use when the user wants to pause and wait for something external like PR reviews, CI completion, approvals, or any webhook-based event.
---

# Await External Events with Charon

When you need to wait for an external event before continuing:

## Prerequisites - Ensure Charon is Running

**IMPORTANT:** All `npx charon-hooks` commands work from ANY directory. Do NOT cd anywhere.

1. Check if config exists at `~/.charon/config/config.yaml`
   - If no config exists, Charon is not set up. Run `npx charon-hooks` to initialize.

2. Read the port from config file

3. Check if Charon is running on that port:
   ```bash
   curl -s http://localhost:<port>/api/triggers
   ```
   - If responds, Charon is running. Proceed.
   - If connection refused, start Charon: `npx charon-hooks --service start`
   - Wait for it to be ready before proceeding

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

3. **Start background task** that waits:
   ```bash
   npx charon-hooks --wait <uuid> --trigger <trigger-id> &
   ```

4. **Configure external webhook** to point to:
   ```
   http://<charon-host>:<port>/api/webhook/<trigger-id>/<uuid>
   ```

5. **Tell the user** you're waiting and what event will resume execution

6. When the background task completes, **continue with the result** (the description output)

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
3. Configure GitHub webhook to: `http://your-server:3000/api/webhook/github-pr-reviewed/$uuid`
4. The wait command will block until the webhook fires
5. When PR is reviewed, the wait command outputs the description and exits

## Webhook URL Format

```
POST http://<host>:<port>/api/webhook/<trigger-id>/<uuid>
```

The UUID connects the specific wait operation to the incoming webhook, ensuring the right Claude session receives the event.
