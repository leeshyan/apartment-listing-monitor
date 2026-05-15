---
name: schedule-setup
description: >
  This skill should be used when the user says "set up my schedule", "create scheduled tasks",
  "set up the automatic checks", "schedule the monitor", "automate my apartment search",
  "set up recurring scans", "create the three tasks", or "how do I make this run automatically".
  Guides the user through creating the three standard scheduled tasks: hourly instant alerts,
  daily digest, and weekly recap.
metadata:
  version: "0.1.0"
---

# Schedule Setup

Create the three standard Cowork scheduled tasks that automate the apartment listing monitor. Each task maps to one of the plugin's scan skills.

## Before You Start

Read `config.yaml` to pull schedule settings. If config doesn't exist, tell the user to run `/configure` first.

Confirm the user wants to create all three tasks, or ask which ones they want:

1. **Instant Alert scan** — runs every N minutes, sends immediate notifications for new matches
2. **Daily Digest** — runs once per day at a configured time, sends a compiled summary
3. **Weekly Recap** — runs once per week, broader search + full summary

## Task Definitions

### Task 1: Instant Alert Scan

- **Name**: "Apartment Listing Monitor — Instant Alerts"
- **Trigger**: Every `config.schedule.instant_check_interval` minutes (default: 60)
- **Instruction to Claude**: "Run the apartment listing monitor check-listings skill. Check all listing URLs in config.yaml for new matches under the configured price threshold. Send instant notifications for any new matches found. Update seen-listings.json with all new listings seen."
- **Mode**: Automated (runs in background)

### Task 2: Daily Digest

- **Name**: "Apartment Listing Monitor — Daily Digest"
- **Trigger**: Daily at `config.schedule.digest_time` in `config.schedule.digest_timezone`
- **Instruction to Claude**: "Run the apartment listing monitor daily digest. Compile all new listing matches from the past 24 hours and send a single formatted digest email. Update seen-listings.json to mark notified listings."
- **Mode**: Automated

### Task 3: Weekly Recap

- **Name**: "Apartment Listing Monitor — Weekly Recap"
- **Trigger**: Weekly on `config.schedule.weekly_recap_day` at `config.schedule.weekly_recap_time`
- **Instruction to Claude**: "Run the apartment listing monitor weekly recap. Perform an expanded broader search, compile all matches from the past 7 days, generate a market snapshot, and send the full weekly summary email. Prune old state entries."
- **Mode**: Automated

## Create the Tasks

Use the Cowork scheduled task tool to create each task. After creating, confirm with the user by showing:

```
Created scheduled tasks:
✅ Instant Alerts — every 60 min
✅ Daily Digest — 8:00am ET daily
✅ Weekly Recap — Sundays at 9:00am ET
```

## Adjustments

If the user wants different intervals or times than what's in config:
- Accept their input
- Update `config.yaml` `schedule` section to match
- Create tasks with the new values

## Pausing and Resuming

Tell the user: "You can pause or delete any of these tasks from the Cowork scheduled tasks panel. To update settings (like the check interval), edit `config.yaml` and then re-run `/schedule-setup` — it will update the existing tasks."
