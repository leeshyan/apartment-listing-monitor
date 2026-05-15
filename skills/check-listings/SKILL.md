---
name: check-listings
description: >
  This skill should be used when the user says "check listings", "check for new apartments",
  "run a listing scan", "check my apartment search", "scan for new units", "any new apartments?",
  or "run the monitor". Also trigger when a scheduled task fires with the intent of checking
  listing URLs for new matches. This is the core skill of the apartment listing monitor.
metadata:
  version: "0.1.0"
---

# Check Listings

Perform a full listing scan: fetch all configured URLs, parse listings, filter against user criteria, update state, and send notifications for new matches.

## Before You Start

Read the user's `config.yaml` (or `config.yml`) from their workspace. If it doesn't exist, tell the user to run the **configure** skill first (`/configure`) and stop.

Load `seen-listings.json` from the path specified in `config.state.file`. If the file doesn't exist, initialize it as:

```json
{
  "last_updated": "<ISO timestamp>",
  "listings": {}
}
```

Load `daily-log.json` from the same directory. If it doesn't exist, initialize it as:

```json
{
  "runs": []
}
```

Record the run start time — you'll append a log entry at the end of the scan.

## Step 1 — Fetch and Parse Each URL

For each URL in `config.listing_urls`:

1. Attempt to fetch the page using your web fetch tool. If the response is empty, contains no listing data, or appears to be a JavaScript-only shell (no prices or unit details in the HTML), mark the source as **JS-blocked** and attempt a fallback: use the Claude in Chrome browser tool if available to load the page in a real browser session. If Chrome is not connected or unavailable, log the source as skipped and continue.

   > **JS-blocking is common** on large property management sites (Irvine Company, Equity Residential, etc.). These sites require JavaScript to render availability. The web fetch tool cannot execute JS — only a real browser session can. If you find that several of your sources are consistently returning empty results, enable Claude in Chrome in Cowork and re-run the scan.
2. Parse every individual listing from the page. A listing is one rentable unit — an apartment, room, or house — with its own price and address. See `references/filter-logic.md` for parsing guidance by common site structure.
3. For each listing, extract as many of these fields as available:
   - `id` — unique identifier (extract from the listing URL path, query param, or data attribute; fall back to a stable hash of address + price)
   - `url` — direct link to the listing detail page
   - `address` — full street address and unit number
   - `neighborhood` — neighborhood or area name
   - `price` — monthly rent as a number (strip `$` and commas)
   - `beds` — number of bedrooms as a number
   - `baths` — number of bathrooms as a number (0.5 increments accepted)
   - `sqft` — square footage if listed
   - `amenities` — array of amenity strings (lowercase, normalized)
   - `phone` — contact phone number if listed
   - `available_date` — move-in date if listed
   - `description` — listing description text

If a page fetch fails, log the error, skip that URL, and continue with the rest. Do not abort the full scan for a single failed URL.

## Step 2 — Identify New Listings

A listing is **new** if its `id` is not present in `seen-listings.json`.

Do not re-process any listing whose `id` already exists in the state file, regardless of price or other changes.

## Step 3 — Apply Filters

For each new listing, apply filters from `config.filters` in this order. Reject the listing at the first failing check:

1. **Price ceiling** — `price > config.filters.max_price` → reject
2. **Bedrooms** — `beds < config.filters.beds_min` → reject; if `beds_max` is set and `beds > config.filters.beds_max` → reject
3. **Bathrooms** — `baths < config.filters.baths_min` → reject
4. **Neighborhood allowlist** — if `config.filters.neighborhoods` is non-empty and the listing's neighborhood is not in the list → reject (case-insensitive match; also check if the address contains any of the allowlisted neighborhood names)
5. **Dealbreakers** — if the listing description or amenities list contains any string from `config.filters.dealbreakers` (case-insensitive substring match) → reject
6. **Must-have amenities** — if any item in `config.filters.must_have` is not found in the listing amenities or description (case-insensitive) → reject

Listings that pass all filters are **matches**.

## Step 4 — Enrich Matches

For each match, compute:

- `price_per_person` = `price / config.filters.num_people` (round to nearest dollar)
- `match_score` — count how many `nice_to_have` amenities are present (higher = better)
- `nice_to_have_found` — array of which nice-to-have amenities were found

## Step 5 — Update State

For every listing processed (both matches and non-matches that passed parsing), add an entry to `seen-listings.json`:

```json
{
  "listing-id": {
    "first_seen": "<ISO timestamp>",
    "url": "https://...",
    "price": 2800,
    "beds": 2,
    "baths": 1,
    "address": "123 Main St, Apt 4B",
    "neighborhood": "Midtown",
    "matched": true,
    "notified": false
  }
}
```

Set `notified: false` initially. After sending notifications, update matched listings to `notified: true`.

Update `last_updated` to the current ISO timestamp.

Prune any listings from state where `first_seen` is older than `config.state.max_age_days` days.

Write the updated `seen-listings.json` back to disk.

Also append a run entry to `daily-log.json`:

```json
{
  "timestamp": "<ISO timestamp>",
  "sources_checked": 9,
  "sources_blocked": 5,
  "new_listings_seen": 3,
  "matches": 1,
  "alerts_sent": true,
  "active_listings": [
    { "id": "abc123", "address": "...", "price": 2800, "notified": true }
  ]
}
```

The `active_listings` array should include all currently tracked matched listings (not just new ones from this run) — this gives the daily digest a full picture of what's available without needing to cross-reference seen-listings.json. Write `daily-log.json` to disk. The daily digest skill clears this file after sending.

## Step 6 — Send Notifications

If there are zero matches, report to the user: "Scan complete. No new matches found. X new listings seen across Y sources." Do not send any external notifications.

If there are matches, format and send notifications per `references/notification-templates.md`.

Send in this order based on what is enabled in `config.notifications`:

1. **~~email** — if `email.enabled: true`, send via the connected email tool
2. **~~sms** — if `sms.enabled: true`, send a compact SMS via the connected SMS tool  
3. **~~chat** — if `chat.enabled: true`, post to the configured channel via the connected chat tool

After sending all notifications, mark each notified listing as `notified: true` in state and write the file again.

## Step 7 — Report to User

End with a brief in-chat summary:

```
Scan complete.
• X sources checked
• Y new listings seen (added to state)
• Z matches found and notified
• Next scheduled check: [time if known]
```

If any URLs failed to fetch or were JS-blocked, list them at the end under "⚠️ Blocked/failed sources" with a note on whether Chrome fallback was attempted.

## Edge Cases

- **No listings parsed from a page**: warn the user that the page structure may have changed. Include the URL and suggest they verify the URL still works.
- **State file corrupted**: back up the file as `seen-listings.json.bak`, reinitialize clean, and warn the user.
- **Notification tool unavailable**: log the failure, note which channel was skipped, and continue with remaining channels.
- **Duplicate IDs across sources**: use the full canonical listing URL as the tiebreaker ID.
