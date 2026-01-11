---
description: Check Charon service status and list active triggers
---

Check if Charon is running and show configured triggers.

## Steps

1. Check Charon status:
   ```bash
   npx charon-hooks --service status
   ```
   This outputs JSON: `{"running": true/false, "port": ..., "url": ..., "webhook_base": ...}`

2. Report status based on the `running` field:
   - If `running: true`: "Charon is running on port <port>" (include `url` and `webhook_base`)
   - If `running: false`: "Charon is not running. Start with: `npx charon-hooks --service start`"

3. If running, fetch triggers from `<url>/api/triggers` and list:
   - ID
   - Name
   - Type (webhook/cron)
   - Enabled status
   - For cron: schedule expression
   - For webhooks: full URL using `webhook_base`

## Troubleshooting

If `--service status` shows `running: false` but you expected it to be running:

```bash
# Check if something else is on port 3000:
lsof -i :3000

# Try starting in foreground to see errors:
npx charon-hooks
```

If the command hangs or times out, the service may have crashed. Check by running it directly.
