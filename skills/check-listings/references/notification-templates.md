# Notification Templates

Templates for all three notification channels (email, SMS, chat) across all three scan modes (instant alert, daily digest, weekly recap).

---

## Instant Alert — Email

**Subject**: `🏠 New Match: [address] — $[price]/mo`

**Body**:

```
New apartment match found!

📍 [address]
💰 $[price]/mo  ($[price_per_person]/person · [num_people] people)
🛏  [beds] bed · 🚿 [baths] bath[· [sqft] sqft]
📅 Available: [available_date | "Contact for availability"]
📞 [phone | omit if not available]
🔗 [url]

✅ AMENITIES FOUND
[amenities list, one per line, with checkmark prefix]

[if nice_to_have_found is non-empty:]
⭐ NICE-TO-HAVES MATCHED
[nice_to_have_found list]

[if description is available:]
📝 LISTING NOTES
[First 300 characters of description, then "…"]

---
Apartment Listing Monitor | [timestamp]
To stop these alerts, update config.yaml
```

---

## Instant Alert — SMS

Keep under 320 characters (two SMS segments).

```
🏠 New match: [address]
$[price]/mo · $[price_per_person]/person
[beds]bd/[baths]ba[· available_date if present]
[url shortened to ~30 chars if possible]
```

---

## Instant Alert — Chat (Slack/Teams)

Use markdown formatting compatible with the connected chat tool.

```
*🏠 New Apartment Match*

*[address]*
💰 $[price]/mo · $[price_per_person]/person ([num_people] people)
🛏 [beds] bed · 🚿 [baths] bath[· [sqft] sqft]
[if available_date:] 📅 Available [available_date]
[if phone:] 📞 [phone]
🔗 <[url]|View Listing>

Amenities: [amenities joined by " · "]
[if nice_to_have_found:] ⭐ Bonus: [nice_to_have_found joined by ", "]
```

---

## Daily Digest — Email

**Subject**: `🏠 Apartment Digest: [N] new match[es] — [date]`

**Body**:

```
Your daily apartment listing digest
[date range: Yesterday 8am – Today 8am]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[N] NEW MATCH[ES] FOUND
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[if match_score >= 2 for any listing:]
⭐ TOP PICK[S]
[For each top-scoring listing:]
  • [address] — $[price]/mo ($[price_per_person]/person) · [beds]bd/[baths]ba
    Matched: [nice_to_have_found joined by ", "]
    [url]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ALL MATCHES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Group by neighborhood. For each neighborhood:]

[NEIGHBORHOOD NAME]
[For each listing in neighborhood:]
  ──────────────────
  📍 [address]
  💰 $[price]/mo  ($[price_per_person]/person)
  🛏  [beds] bed · 🚿 [baths] bath[· [sqft] sqft]
  [if available_date:] 📅 Available: [available_date]
  [if phone:] 📞 [phone]
  ✅ [amenities joined by " · "]
  [if nice_to_have_found:] ⭐ [nice_to_have_found joined by ", "]
  🔗 [url]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[if zero matches:]
No new matches in the past 24 hours. [N] listings were seen and tracked.

---
Apartment Listing Monitor · Daily Digest · [date]
```

---

## Daily Digest — Chat

```
*🏠 Daily Apartment Digest — [date]*
[N] new match[es] found

[For top 3 matches:]
• *[address]* — $[price]/mo ($[price_per_person]/person) · [beds]bd/[baths]ba
  <[url]|View listing>

[if more than 3:] _+ [N-3] more — check your email for full digest_
```

---

## Weekly Recap — Email

**Subject**: `🏠 Weekly Recap: [N] matches · Avg $[avg_match_price]/mo · [date range]`

**Body**:

```
Your weekly apartment listing recap
[Mon date] – [Sun date]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
WEEK AT A GLANCE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Total listings seen:    [total_seen]
✅ Matches (passed filters): [total_matched] ([match_rate]%)
💰 Match price range:       $[min_match_price] – $[max_match_price]/mo
📈 Avg match price:         $[avg_match_price]/mo ($[avg_price_per_person]/person)
🏘  Top neighborhoods:       [top 3 neighborhoods by listing count]
🚫 Most common filter hit:  [most_common_rejection_reason]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOP PICKS THIS WEEK
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Top 5 matches sorted by match_score desc, price asc]
[For each:]
  #[rank]. [address]
  $[price]/mo · $[price_per_person]/person · [beds]bd/[baths]ba
  ⭐ [nice_to_have_found joined by ", "]
  [url]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ALL MATCHES THIS WEEK
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Same format as daily digest, grouped by neighborhood]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
WORTH A SECOND LOOK
(Slightly over budget or outside area — high amenity score)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Expanded scan results sorted by match_score, max 5]
[Same listing card format, with note: "(over budget by $X)" or "(outside filter area)"]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MARKET NOTES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2-3 sentence commentary based on metrics:
- Is inventory up or down vs last week's state count?
- Are prices tracking toward your max or staying comfortable?
- Any neighborhoods with notable new inventory?
]

---
Apartment Listing Monitor · Weekly Recap · [date]
```

---

## Formatting Notes

- All prices formatted as `$X,XXX` (commas, no cents).
- Dates formatted as `Mon Jan 6` for short form, `Monday, January 6, 2025` for full form.
- If `available_date` is "immediately" or "now", display as "Available Now".
- Phone numbers formatted as `(555) 555-5555` for US numbers; display as-is for international.
- Listing URLs: use the direct listing URL, not the search results URL.
- For chat tools that don't support markdown: strip `*` bold markers, replace bullet `•` with `-`.
