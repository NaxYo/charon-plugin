---
description: List and manage Charon triggers
---

Show all configured triggers.

## Steps

1. Check Charon status to get the webhook base URL:
   ```bash
   npx charon-hooks --service status
   ```
   This outputs JSON: `{"running": true/false, "port": ..., "url": ..., "webhook_base": ...}`

2. Read `~/.charon/config/triggers.yaml`
   - If file doesn't exist, report that no triggers are configured

3. Parse the YAML and display triggers in a formatted table:
   - ID
   - Name
   - Type (webhook/cron)
   - Enabled (yes/no)
   - For webhooks: full URL (`<webhook_base>/<trigger-id>`)
   - For cron: schedule expression
   - One-off status if applicable

4. Show helpful information:
   - Webhook URL format uses `webhook_base` from status (includes tunnel URL if active)
   - How to add new triggers
   - How to test triggers
