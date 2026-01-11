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

## Troubleshooting

### No triggers showing?

1. Check the file exists: `ls -la ~/.charon/config/triggers.yaml`
2. Verify YAML syntax is valid (use a YAML linter if unsure)
3. Ensure triggers are under the `triggers:` key as a list

### Trigger exists but webhook URL wrong?

The webhook URL uses `webhook_base` from `--service status`. If using a tunnel (ngrok), ensure:
1. Tunnel is configured in triggers.yaml
2. Service was restarted after adding tunnel config
3. Check `--service status` shows the tunnel URL

### Changes to triggers.yaml not reflected?

The service watches the file and auto-reloads. If not working:
1. Check for YAML syntax errors (service logs will show parse failures)
2. Restart service: `npx charon-hooks --service stop && npx charon-hooks --service start`
