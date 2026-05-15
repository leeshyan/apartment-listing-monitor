---
name: daily-digest
description: >
  This skill should be used when the user says "send daily digest", "send the digest",
  "morning summary", "daily apartment summary", "what matched yesterday", "send digest email",
  or when a scheduled task fires with a daily digest intent. Compiles all new listing matches
  from the past 24 hours and sends a single formatted summary notification — does not send
  per-listing instant alerts.
metadata:
  version: "0.1.0"
---

# Daily Digest

Compile all new apartment listing matches from the past 24 hours into a single formatted digest notification. This is the batch counterpart to instant alerts — one message summarizing everything new.

## Before You Start

Read `config.yaml`. If it doesn't exist, stop and tell the user to run `/configure` first.

Check that `config.notifications.email.digest_enabled` is `true`. If digest notifications are disabled, tell the user and stop.

Load `seen-listings.json`. If it doesn't exist or is empty, report: "No listings in state yet — run a listing scan first with 'check listings'."

## Step 1 — Run a Fresh Scan (Optional but Recommended)

Before compiling the digest, run a fresh check of all listing URLs (same logic as the `check-listings` skill, Steps 1–5) to catch any listings added since the last scheduled check.

Skip this step only if the user explicitly says to send the digest from existing state without rescanning.

## Step 2 — Collect Digest Matches

From `seen-listings.json`, collect all listings where:
- `matched: true`
- `notified: false` (unseen in a digest)
- `first_seen` is within the past 24 hours (use `config.schedule.digest_timezone` for cutoff)

If zero matches qualify, send a "nothing new" digest if `config.notifications.email.digest_empty_send` is `true`, otherwise skip sending and report to user in chat.

## Step 3 — Sort and Rank

Sort matches by `match_score` descending (most nice-to-haves first), then by `price` ascending as a tiebreaker.

## Step 4 — Format the Digest

Build the digest content using the **Digest Email** template from `references/notification-templates.md`.

The digest groups listings by neighborhood (or by source URL if neighborhood is unknown) and includes:
- Count of new matches
- Time window covered
- Each listing card with full details
- A ranked "top picks" callout for listings with `match_score >= 2`

## Step 5 — Send Notifications

Send via all enabled digest channels:

- **~~email**: send to `config.notifications.email.to`
- **~~chat**: if `config.notifications.chat.enabled` and `chat.digest_enabled` is not explicitly `false`, post to `config.notifications.chat.channel`

Do not send SMS for digests (SMS is instant-alert only by convention).

## Step 6 — Mark as Notified

After sending, update all included listings in `seen-listings.json` to `notified: true`. Write the file.

## Step 7 — Report to User

```
Daily digest sent.
• Time window: [yesterday 8am – today 8am]
• New matches: X
• Top pick: [address] at $Y/mo ($Z/person)
• Sent via: email [, Slack]
```
