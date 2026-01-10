---
description: List and manage Charon triggers
---

Show all configured triggers from `~/.charon/config/triggers.yaml`.

## Steps

1. Read `~/.charon/config/triggers.yaml`
   - If file doesn't exist, report that no triggers are configured

2. Parse the YAML and display triggers in a formatted table:
   - ID
   - Name
   - Type (webhook/cron)
   - Enabled (yes/no)
   - For webhooks: endpoint URL format
   - For cron: schedule expression
   - One-off status if applicable

3. Show helpful information:
   - Webhook URL format: `http://<host>:<port>/api/webhook/<trigger-id>`
   - How to add new triggers
   - How to test triggers
