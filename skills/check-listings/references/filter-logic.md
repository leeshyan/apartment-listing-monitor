# Filter Logic Reference

Detailed guidance on parsing listing pages and applying filters.

## Parsing Strategy by Site Type

Different listing sites expose data in different ways. Use these strategies in priority order:

### 1. Structured Data (preferred)

Look for `<script type="application/ld+json">` tags containing `schema.org/Apartment` or `schema.org/RealEstateListing` data. This is the cleanest source — extract fields directly from the JSON.

Also check for JSON blobs embedded in `window.__INITIAL_STATE__`, `window.__NEXT_DATA__`, or similar global variables in `<script>` tags.

### 2. HTML Attributes

Many listing sites embed data in `data-*` attributes:
- `data-listing-id`, `data-price`, `data-beds`, `data-baths`
- `data-lat`, `data-lng` (can indicate neighborhood)
- `itemscope`/`itemprop` microdata attributes

### 3. HTML Text Parsing (fallback)

Parse listing cards by identifying the repeating element pattern (often `<article>`, `<li>`, or `<div>` with a listing-related class like `.listing-card`, `.result-item`, `.property-card`).

For each card, extract:
- **Price**: Look for `$` followed by digits. Strip commas. Common patterns: `$2,400/mo`, `$2400 per month`, `$2,400`.
- **Beds**: Look for patterns like `2 bd`, `2 bed`, `2 BR`, `2 bedrooms`, `Studio` (= 0 beds).
- **Baths**: Look for `1 ba`, `1 bath`, `1.5 baths`, `1 bathroom`.
- **Address**: Look for street address patterns (number + street name) or a clearly labeled address element.
- **Listing URL**: Find the `<a>` tag wrapping or inside the card with `href` pointing to a listing detail page.

### 4. Common Site Patterns

**Craigslist**: Listings are in `<li class="cl-search-result">` elements. Price is in `.price`. Beds in `.housing`. Link in `<a class="cl-app-anchor">`.

**Zillow**: Data is in `__NEXT_DATA__` JSON. Look for `searchResults.mapResults` or `cat1.searchResults.listResults` array.

**Apartments.com**: Listing cards are `<article class="placard">`. Price in `.rentLabel`, beds in `.detailsTextWrapper`.

**Facebook Marketplace**: Data often in `__bbox` JSON blob. May require browser rendering — note a warning if the page appears empty (JS-rendered).

**Trulia**: Similar to Zillow (same parent company). Check `__NEXT_DATA__`.

**HotPads**: Check for JSON in `<script>` tags with `listing` arrays.

## Listing ID Extraction

Priority order for generating a stable listing ID:

1. Extract from URL path: `/rental/12345678` → `12345678`
2. Extract from URL query params: `?listingId=abc123` → `abc123`
3. Extract from `data-listing-id` or similar attribute
4. Hash of (address + price): `sha256(address.lower() + str(price))[:12]`

Use the full canonical listing URL as the ID if all else fails.

## Amenity Normalization

Normalize all amenity strings to lowercase. Map common variants:

| Raw text (examples) | Normalized |
|---|---|
| W/D in unit, washer/dryer, in-unit laundry | `in-unit laundry` |
| DW, dishwasher | `dishwasher` |
| A/C, central air, central AC, air conditioning | `air conditioning` |
| Parking, garage, parking included | `parking` |
| Elevator, lift | `elevator` |
| Gym, fitness center, fitness room | `gym` |
| Doorman, concierge, door attendant | `doorman` |
| Pets OK, pet friendly, cats ok, dogs ok | `pet-friendly` |
| Roof deck, rooftop, roof access | `rooftop` |
| Hardwood, hardwood floors | `hardwood floors` |
| Balcony, deck, patio | `outdoor space` |

## Dealbreaker Detection

Check for dealbreaker strings across:
1. `amenities` array (normalized)
2. `description` text (raw, case-insensitive)
3. `address` text (for things like "no pets" in address notes)

A match is a case-insensitive substring match. For example, dealbreaker `"no pets"` matches `"Sorry, no pets allowed"`.

## Price Parsing Edge Cases

- **"Call for pricing"** or **"Price upon request"**: set price to `null`. These listings will fail the price filter and be skipped (they cannot be verified against `max_price`). Log them separately.
- **Ranges like "$2,000–$2,400"**: use the higher end of the range (pessimistic).
- **Weekly pricing**: convert to monthly by multiplying by 4.33.
- **Price includes utilities**: note in the listing card that utilities are included, but use the listed price as-is.

## Neighborhood Detection

Try these sources in order:

1. Explicit neighborhood field in structured data
2. `data-neighborhood` attribute
3. Listing page title or h1 tag (often contains "2BR in [Neighborhood]")
4. Cross-reference listing address against `config.filters.neighborhoods` using substring match

If no neighborhood can be determined and `config.filters.neighborhoods` is non-empty, log a warning and skip neighborhood filtering for that listing (do not reject it solely because neighborhood is unknown).
