---
name: find-accommodation
description: "Search for accommodation on Airbnb and Booking.com. Uses MCP tools for live data with anti-hallucination guardrails. Use when: user asks to find a place to stay, search Airbnb, find hotels, book accommodation, or look for rentals in any destination."
argument-hint: "<destination> [dates] [guests] [budget]"
---

# Find Accommodation

Search for accommodation using live MCP data with full quality verification.

## Before Searching

1. **Read the user's travel profile** from `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
2. **Detect context** from the conversation — solo, partner, group, or work trip
3. **Confirm search parameters** with the user if any are missing:
   - Destination (required)
   - Check-in and check-out dates (required)
   - Number of guests (required)
   - Budget per night (use profile default if not specified)
   - Must-have amenities (use profile default if not specified)

## Search Execution

### Primary Search: Airbnb MCP

Use the `airbnb` MCP server tools to search. Pass:
- Location/destination
- Check-in and check-out dates
- Number of guests
- Price range filters if available

### Cross-Check: Web Search

For top Airbnb results, cross-check via web search:
- Search for the same property or similar ones on Booking.com
- Compare prices — flag discrepancies >5%

### Fallback

If the Airbnb MCP is not connected or returns errors:
- Use web search to find listings on Airbnb.com and Booking.com directly
- Note that results are from web search, not MCP

## Listing Eligibility

A listing must meet ALL of these to be recommended:
- Availability confirmed for the requested dates (live check)
- Price retrieved live in this session
- Review score: ≥4.3/5 on Airbnb (10+ reviews) OR ≥7.5/10 on Booking.com (20+ reviews)
- Direct booking URL verified
- Guest capacity meets or exceeds party size

If fewer than 3 listings pass: tell the user and offer to widen the search (expand area, adjust budget, relax dates).

## Output Template

Present each recommendation in this format:

```
### [NUMBER]. [LISTING NAME]

**Location:** [Neighbourhood], [City]
**Price:** [EXACT PRICE]/night | [TOTAL FOR STAY] total (inc. fees & taxes)
**Verified:** [HH:MM today] via [Source]
**Rating:** [SCORE]/5 ([NUMBER] reviews) — [or /10 for Booking.com]
**Details:** [BEDROOMS] bedroom(s) | Sleeps [GUESTS]
**Amenities:** [TOP AMENITIES] (+ [N] more)
**Booking:** [DIRECT URL]

> [One-sentence summary of why this matches their preferences]
```

After all listings:

```
---
Prices may change — always verify at checkout.
[If applicable: "I couldn't cross-check these on a second source — prices shown are from [source] only."]
```

## Preference Matching

When ranking results, apply the user's profile:
- **Prefer tags** (+15 each): amenities they listed, neighbourhood vibe they described, accommodation type preference
- **Avoid tags** (-50 each): anything from past negative experiences, explicitly unwanted features
- If the user mentioned specific requirements in THIS conversation, those override profile defaults

## Special Handling

- **"Walkable to beach"** → filter for listings within 500m of beach/coast, prioritise listings mentioning beach access
- **"Small budget"** → set max to lowest quartile of results, emphasise value-for-money
- **"Direct flights only"** → this is a flight filter, not accommodation — note it for the flight search
- **"Entire place"** → exclude shared rooms and hotel rooms, only full apartments/houses
