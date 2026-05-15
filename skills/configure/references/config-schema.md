# Config Schema Reference

Full documentation of every field in `config.yaml`.

## Top-Level Structure

```yaml
listing_urls: []        # required
filters: {}             # required
notifications: {}       # required
state: {}               # optional (has defaults)
schedule: {}            # optional (used by schedule-setup skill)
weekly_recap: {}        # optional (extra settings for weekly recap)
```

---

## `listing_urls`

**Type**: list of strings (URLs)
**Required**: yes
**Minimum**: 1 URL

Each URL should be a search results page — not an individual listing page. The monitor fetches this page and parses all individual listings from it.

```yaml
listing_urls:
  - https://craigslist.org/search/apa?...
  - https://zillow.com/homes/for_rent/...
  - https://apartments.com/city-name/...
```

**Tips**:
- Apply your filters on the site itself first (city, price range, beds) to reduce noise.
- Test each URL manually in a browser to confirm it returns listings.
- Some sites require login to view results — those URLs will not work.

---

## `filters`

### `beds_min`
**Type**: integer | Default: 0
Minimum number of bedrooms. Use `0` for studio.

### `beds_max`
**Type**: integer | Optional
Maximum number of bedrooms. Omit for no upper limit.

### `baths_min`
**Type**: number | Default: 1
Minimum number of bathrooms. Accepts `0.5` increments (e.g., `1.5`).

### `max_price`
**Type**: integer | Required
Maximum monthly rent in dollars (no `$` symbol).

### `num_people`
**Type**: integer | Default: 1
Number of people splitting rent. Used to calculate `price_per_person` in notifications.

### `neighborhoods`
**Type**: list of strings | Default: [] (empty = no neighborhood filter)
Allowlisted neighborhood names. Case-insensitive. A listing passes if its neighborhood or address contains any of these strings as a substring.

```yaml
neighborhoods:
  - Downtown
  - Midtown
  - West Village
  - Chelsea
```

Leave empty to accept listings from any neighborhood.

### `must_have`
**Type**: list of strings | Default: []
Amenities that must be present. A listing is rejected if any of these are missing. Use normalized amenity names (see filter-logic.md for normalization table).

```yaml
must_have:
  - dishwasher
  - in-unit laundry
```

### `nice_to_have`
**Type**: list of strings | Default: []
Amenities that boost the listing's `match_score` but don't disqualify if absent. Used for ranking in digest and recap.

```yaml
nice_to_have:
  - parking
  - gym
  - rooftop
  - doorman
```

### `dealbreakers`
**Type**: list of strings | Default: []
Substrings that, if found in a listing's description or amenities, cause it to be rejected. Case-insensitive.

```yaml
dealbreakers:
  - no pets
  - street parking only
  - heat not included
  - no laundry on site
```

---

## `notifications`

### `notifications.email`

```yaml
email:
  enabled: true
  to: you@example.com           # Recipient email address
  from: alerts@example.com      # Sender address (must be valid for your email connector)
  instant_alerts: true          # Send per-listing email immediately on new match
  digest_enabled: true          # Also send daily digest
  digest_empty_send: false      # Send digest even when zero new matches
```

### `notifications.sms`

```yaml
sms:
  enabled: false
  to: "+15555555555"            # E.164 format. US: +1XXXXXXXXXX
  instant_alerts: true          # SMS only used for instant alerts, not digests
```

### `notifications.chat`

```yaml
chat:
  enabled: false
  channel: "#apartment-hunt"    # Channel name (with # for Slack)
  instant_alerts: true
  digest_enabled: true          # Also post digest to channel
```

---

## `state`

```yaml
state:
  file: seen-listings.json      # Path to state file. Relative to config.yaml location.
  max_age_days: 30              # Listings older than this are pruned from state on weekly recap.
```

---

## `schedule`

Used by the `schedule-setup` skill when creating Cowork scheduled tasks. Does not affect manual scans.

```yaml
schedule:
  instant_check_interval: 60   # Minutes between instant alert scans. Min: 15.
  digest_time: "08:00"         # 24h HH:MM format.
  digest_timezone: "America/New_York"  # IANA timezone string.
  weekly_recap_day: "Sunday"   # Day name: Monday, Tuesday, ..., Sunday
  weekly_recap_time: "09:00"   # 24h HH:MM format.
```

**Common timezone strings**: `America/New_York`, `America/Chicago`, `America/Denver`, `America/Los_Angeles`, `America/Phoenix`, `Europe/London`, `Europe/Berlin`, `Asia/Tokyo`

---

## `weekly_recap`

Optional extra settings for the weekly recap scan.

```yaml
weekly_recap:
  extra_urls:                  # Additional broader-search URLs for weekly only
    - https://craigslist.org/search/apa?...  # City-wide search (no neighborhood filter)
  price_buffer_pct: 15         # % above max_price to include in "Worth a Second Look" section
```
