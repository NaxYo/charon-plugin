---
name: charon-schedule
description: Schedule Claude Code tasks to run at specific times. Use when the user wants recurring or delayed task execution.
---

# Scheduled Tasks with Charon

Schedule Claude Code tasks to run at specific times, either recurring or one-off.

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

## Setting Up a Scheduled Task

Add to `~/.charon/config/triggers.yaml`:

```yaml
triggers:
  - id: <unique-trigger-id>
    name: "<Descriptive Name>"
    type: cron
    enabled: true
    schedule: "<cron expression>"
    template: "<Task description>"
    egress: cli
    context:
      cli_template: "claude --dangerously-skip-permissions --task '{description}'"
      working_dir: "/path/to/project"  # optional
```

## Cron Expression Format

```
┌─────────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌─────────── day of month (1-31)
│ │ │ ┌───────── month (1-12)
│ │ │ │ ┌─────── day of week (0-7, 0 and 7 are Sunday)
│ │ │ │ │
* * * * *
```

Common patterns:
- `0 9 * * *` - Every day at 9 AM
- `0 9 * * 1-5` - Weekdays at 9 AM
- `*/30 * * * *` - Every 30 minutes
- `0 0 1 * *` - First day of every month at midnight
- `0 14 10 1 *` - January 10th at 2 PM (one-off with `one_off: true`)

## One-Off Scheduled Tasks

For tasks that should only run once, add `one_off: true`:

```yaml
triggers:
  - id: check-deployment
    name: "Check Deployment Health"
    type: cron
    enabled: true
    schedule: "30 14 10 1 *"  # January 10th at 2:30 PM
    one_off: true  # Automatically deleted after first execution
    template: "Check if the staging deployment is healthy. Verify all endpoints respond correctly."
    egress: cli
    context:
      cli_template: "claude --dangerously-skip-permissions --task '{description}'"
      working_dir: "/home/user/myapp"
```

The trigger will automatically be removed from the config after it fires once.

## Example: Daily Security Audit

```yaml
triggers:
  - id: daily-security-audit
    name: "Daily Security Audit"
    type: cron
    enabled: true
    schedule: "0 9 * * *"  # Every day at 9 AM
    template: "Run a security audit on the codebase. Check for vulnerabilities, outdated dependencies, and security best practices."
    egress: cli
    context:
      cli_template: "claude --dangerously-skip-permissions --task '{description}'"
      working_dir: "/home/user/myapp"
```

## Example: Weekly Report

```yaml
triggers:
  - id: weekly-report
    name: "Weekly Status Report"
    type: cron
    enabled: true
    schedule: "0 17 * * 5"  # Every Friday at 5 PM
    template: "Generate a weekly status report. Summarize commits, issues closed, and pending work."
    egress: cli
    context:
      cli_template: "claude --dangerously-skip-permissions --task '{description}'"
      working_dir: "/home/user/myapp"
```

## Example: Delayed Follow-Up (One-Off)

When you want to check on something later:

```yaml
triggers:
  - id: followup-deployment
    name: "Follow-up on Deployment"
    type: cron
    enabled: true
    schedule: "0 15 * * *"  # Today at 3 PM (adjust as needed)
    one_off: true
    template: "Check if the deployment from this morning is still healthy. Look for any errors in logs."
    egress: cli
    context:
      cli_template: "claude --dangerously-skip-permissions --task '{description}'"
      working_dir: "/home/user/myapp"
```

## Context Variables

The template can use these built-in variables:
- `{trigger_time}` - ISO timestamp when the cron fired
- `{trigger_id}` - The trigger ID

Plus any static context you define in the trigger.

## Managing Scheduled Tasks

View all cron triggers via the Charon dashboard at the URL from `--service status`.

Disable a task (set enabled to false in triggers.yaml or via dashboard).

Delete a task (remove from triggers.yaml or via dashboard).

## Troubleshooting

### Service won't start?

```bash
# Run in foreground to see actual errors:
npx charon-hooks

# Check if port 3000 is already in use:
lsof -i :3000
```

### Cron job not firing?

1. Check the trigger is enabled in `~/.charon/config/triggers.yaml`
2. Verify the cron expression is valid (use https://crontab.guru to test)
3. Check service is running: `npx charon-hooks --service status`
4. View recent runs: `curl http://localhost:3000/api/runs | jq`

### Task runs but Claude doesn't execute?

1. Verify `cli_template` is correct in the trigger config
2. Check that `claude` CLI is installed and in PATH
3. Test manually: `claude --dangerously-skip-permissions --task "test"`

### Quick health check

```bash
# Check service is responding:
curl http://localhost:3000/api/triggers

# Should return JSON with your triggers, not HTML
```
