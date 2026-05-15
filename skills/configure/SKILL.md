---
name: configure
description: >
  This skill should be used when the user says "configure", "set up my apartment search",
  "set up the monitor", "create my config", "first time setup", "I want to set up listing alerts",
  "configure apartment listing monitor", "help me set up my config.yaml", or "get started".
  Use this skill to walk the user through creating a config.yaml from scratch or editing an existing one.
metadata:
  version: "0.1.0"
---

# Configure

Walk the user through creating or updating their `config.yaml` for the apartment listing monitor. Produce a complete, valid config file they can save and use immediately.

## Check for Existing Config

First, check if a `config.yaml` (or `config.yml`) already exists in the user's workspace.

- If it exists: load it, summarize current settings, and ask if they want to review/update specific sections or start from scratch.
- If it doesn't exist: tell the user you'll walk them through creating one, then proceed with the questions below.

## Gather Configuration — Ask Section by Section

Work through each section conversationally. Ask multiple related questions together. Do not ask one question per message — group logically related fields.

### Section 1: Listing Sources

Ask: "What listing sites are you monitoring? Paste the search result URLs — the pages that show multiple listings matching your criteria. Examples: Zillow search results, Craigslist housing searches, Apartments.com filtered results, Facebook Marketplace housing, etc."

Accept a list of URLs. Validate that each looks like a real URL (starts with https://). Warn if fewer than 1 URL is provided.

### Section 2: Size and Price Filters

Ask in one message:
- How many bedrooms are you looking for? (min and max, or exact)
- Minimum bathrooms?
- Maximum monthly rent?
- How many people are splitting the rent? (used to compute price-per-person)

### Section 3: Location Filters

Ask: "Are you limiting your search to specific neighborhoods? If so, list them. Leave blank to accept any neighborhood."

### Section 4: Amenities

Ask in one message:
- "Any must-have amenities? (e.g., dishwasher, in-unit laundry, parking, elevator, gym, doorman, pet-friendly) List any that are non-negotiable."
- "Any dealbreaker phrases? (e.g., 'no pets', 'street parking only', 'heat not included') Listings containing these will be skipped automatically."
- "Nice-to-have amenities for ranking? (optional)"

### Section 5: Notifications

Ask: "How would you like to be notified about new matches?"

Present the options: email, SMS, Slack/Teams/chat. Ask which they want to enable and gather:

**Email**: recipient address, sender address (can be same)
**SMS**: phone number in E.164 format (e.g., +15555555555)
**Chat**: channel name (e.g., #apartment-hunt)

Note: connections to the actual email/SMS/chat services are set up separately — you just need the addresses here.

Also ask: should instant alerts fire every time a new match is found, or batch into the scheduled digest only?

### Section 6: Schedule

Ask: "Let's set your check schedule:
- How often should it check for new listings? (e.g., every 30 minutes, hourly)
- What time should the daily digest email go out? (e.g., 8am)
- What's your timezone?"

### Section 7: State File Location

Tell the user: "The monitor tracks which listings it's already seen in a file called `seen-listings.json`. By default it will be saved in the same folder as your config. That's fine for most setups — just confirm you're okay with this, or give me a different path."

## Generate the Config File

After gathering all answers, produce the complete `config.yaml` with:
- All user-provided values filled in
- Clear comments on each section
- Reasonable defaults for anything the user skipped
- No personal data placeholders left in (replace all with real values)

Offer to save it to the workspace as `config.yaml`. If the user confirms, write it.

Also tell the user: "You can reference `config.example.yaml` in the plugin folder any time to see the full schema with all available options."

## Next Steps

After saving config, tell the user their three options:

1. **Test it now** — "Say 'check listings' and I'll run a scan immediately using your new config."
2. **Set up scheduled tasks** — "Say 'set up my schedule' and I'll create the three scheduled tasks: hourly instant alerts, daily digest, and weekly recap."
3. **Both** — run a test scan first, then set up the schedule.

## Editing Existing Config

If the user wants to update a specific section only (e.g., "change my price limit" or "add another URL"):

1. Load `config.yaml`
2. Make the targeted change
3. Show the updated section as a diff
4. Ask for confirmation before writing
5. Save and confirm

Never overwrite the entire file when only editing one section.
