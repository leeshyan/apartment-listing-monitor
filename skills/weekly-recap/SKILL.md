---
name: weekly-recap
description: >
  This skill should be used when the user says "send weekly recap", "weekly summary",
  "weekly apartment report", "send the Sunday recap", "what's the week looked like",
  "broader search summary", or when a scheduled task fires with a weekly recap intent.
  Runs a broader city-wide or expanded search, compiles the full week of matches,
  and sends a comprehensive summary with market trends.
metadata:
  version: "0.1.0"
---

# Weekly Recap

Run a broader expanded search, compile the full week of activity, and send a comprehensive weekly summary report. This is the most thorough of the three notification modes — designed to surface deals that may have been missed in filtered instant scans and give a sense of the broader market.

## Before You Start

Read `config.yaml`. If it doesn't exist, stop and tell the user to run `/configure` first.

Load `seen-listings.json`.

## Step 1 — Expanded Search Pass

Run a scan of all `listing_urls` using **relaxed filters**:

- Use the same URLs as the regular scan
- Increase `max_price` by 15% for this pass (surface nearby options slightly over budget)
- Remove the neighborhood allowlist filter for this pass (cast a wider net)
- Keep dealbreakers and bed/bath minimums in place
- Mark all results from this pass as `weekly_scan: true` in state to distinguish from instant-alert matches

If `config.weekly_recap.extra_urls` is defined, also scan those URLs. These are intended for broader city-wide searches (e.g., a city-wide Craigslist search vs. the neighborhood-specific URLs used for instant alerts).

## Step 2 — Compile Weekly Window

From `seen-listings.json`, collect all listings where:
- `first_seen` is within the past 7 days
- `matched: true` (passed all standard filters)

Also collect the expanded results from Step 1 as a separate "Over-budget / Outside area" section.

## Step 3 — Market Snapshot

Compute these metrics from all listings seen in the past 7 days (not just matches):

- Total listings seen across all sources
- Number that passed filters (conversion rate)
- Average price of all listings seen
- Average price of matched listings
- Price range of matches (min–max)
- Most common neighborhoods in results
- Most common dealbreaker hit (which filter rejected the most listings)

## Step 4 — Format the Recap

Build the recap content using the **Weekly Recap** template in `references/notification-templates.md`.

Structure:
1. **Week at a Glance** — market snapshot numbers
2. **Top Picks This Week** — top 5 matches sorted by `match_score` then price
3. **All Matches** — full list of every match this week, grouped by neighborhood
4. **Worth a Second Look** — expanded results (slightly over budget / outside area) that scored high on nice-to-haves
5. **Market Notes** — brief commentary: is inventory up or down vs. the prior week's state? Are prices trending in any direction?

## Step 5 — Send Notifications

Send via:

- **~~email**: send to `config.notifications.email.to` (always for weekly recap)
- **~~chat**: post to `config.notifications.chat.channel` if chat is enabled

## Step 6 — Mark as Notified and Prune State

After sending:
- Mark all weekly-recap listings as `notified: true`
- Prune listings older than `config.state.max_age_days` from state
- Write `seen-listings.json`

## Step 7 — Report to User

```
Weekly recap sent.
• Week: [Mon date] – [Sun date]
• Total listings seen: X across Y sources
• Matches this week: Z
• Avg match price: $X/mo ($Y/person)
• Top pick: [address] at $Z/mo
• Sent via: email [, Slack]
```
