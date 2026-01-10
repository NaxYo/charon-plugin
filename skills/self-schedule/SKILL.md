---
name: charon-schedule
description: Schedule Claude Code tasks to run at specific times. Use when the user wants recurring or delayed task execution.
---

# Scheduled Tasks with Charon

Schedule Claude Code tasks to run at specific times, either recurring or one-off.

## Prerequisites - Ensure Charon is Running

1. Check if config exists at `~/.charon/config/config.yaml`
   - If no config exists, Charon is not set up. Run `npx charon-hooks` to initialize.

2. Read the port from config file

3. Check if Charon is running on that port:
   ```bash
   curl -s http://localhost:<port>/api/triggers
   ```
   - If responds, Charon is running. Proceed.
   - If connection refused, start Charon: `npx charon-hooks --service start`

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

View all scheduled tasks:
```bash
curl -s http://localhost:3000/api/triggers | jq '.triggers[] | select(.type == "cron")'
```

Disable a task (set enabled to false in triggers.yaml or via API).

Delete a task (remove from triggers.yaml).
