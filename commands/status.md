---
description: Check Charon service status and list active triggers
---

Check if Charon is running and show configured triggers.

## Steps

1. Read `~/.charon/config/config.yaml` for the port configuration
   - If file doesn't exist, report that Charon is not configured

2. Try to reach Charon at `http://localhost:<port>/api/triggers`

3. Report status:
   - If responds: "Charon is running on port <port>"
   - If fails: "Charon is not running"

4. If running, list the configured triggers with their:
   - ID
   - Name
   - Type (webhook/cron)
   - Enabled status
   - For cron: schedule expression
