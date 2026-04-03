---
name: find-accommodation
description: "Search for accommodation on Airbnb and Booking.com. Uses MCP tools for live data with anti-hallucination guardrails. Use when: user asks to find a place to stay, search Airbnb, find hotels, book accommodation, or look for rentals in any destination."
argument-hint: "<destination> [dates] [guests] [budget]"
---

# Find Accommodation

Search for accommodation using live MCP data with full quality verification.

## Before Searching

1. **Load the user's travel profile** using the persistent-memory fallback chain:
   - Try reading `${CLAUDE_PLUGIN_DATA}/travel-profile.json` first
   - If not found, check Claude's memory for travel profile facts
   - If neither exists, ask the essential questions for this search
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
- `ignoreRobotsText: true` — **ALWAYS include this parameter** to avoid robots.txt blocks that cause the first search call to fail

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
- Review score: >=4.3/5 on Airbnb (10+ reviews) OR >=7.5/10 on Booking.com (20+ reviews)
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
**Loyalty:** [If applicable: "Earns points on [programme]" for hotel chains]
**Book this property:** [DEEP BOOKING LINK — see section below]

> **Why this?** [Specific reasoning citing profile preferences, neighbourhood vibe, amenity match, or loyalty programme]
```

After all listings:

```
---
Prices may change — always verify at checkout.
[If applicable: "I couldn't cross-check these on a second source — prices shown are from [source] only."]
```

## Deep Booking Links

**ALWAYS generate deep links that take the user to the specific property with dates pre-filled, not a generic homepage or search page.**

### Airbnb
The Airbnb MCP returns listing URLs. Use those directly, but append date and guest parameters:
- `[Book this property](https://www.airbnb.com/rooms/LISTING_ID?check_in=YYYY-MM-DD&check_out=YYYY-MM-DD&adults=N&children=N)`

### Booking.com
When cross-checking or finding listings on Booking.com, construct deep links:
- `[Book on Booking.com](https://www.booking.com/hotel/COUNTRY_CODE/HOTEL_SLUG.html?checkin=YYYY-MM-DD&checkout=YYYY-MM-DD&group_adults=N&group_children=N)`

### Hotel Chains (direct booking)
When the listing is from a hotel chain, prefer the chain's direct booking link for loyalty points:
- **Marriott:** `[Book on Marriott](https://www.marriott.com/reservation/availabilitySearch.mi?propertyCode=XXXXX&fromDate=MM/DD/YYYY&toDate=MM/DD/YYYY&flexibleDates=false&clusterCode=none&numberOfRooms=1&numberOfGuests=N)`
  - If user has Marriott Bonvoy number, append: `&memberNumber=MEMBERSHIP_NUMBER`
- **Hilton:** `[Book on Hilton](https://www.hilton.com/en/book/reservation/rooms/?ctyhocn=PROPERTY_CODE&arrivalDate=YYYY-MM-DD&departureDate=YYYY-MM-DD&room1NumAdults=N)`
- **IHG:** `[Book on IHG](https://www.ihg.com/hotels/us/en/find-hotels/hotel/rooms?qDest=HOTEL_NAME&qCiD=DD&qCiMy=MMYYYY&qCoD=DD&qCoMy=MMYYYY&qAdlt=N&qRms=1)`
- **Accor:** `[Book on Accor](https://all.accor.com/ssr/app/accor/rates/HOTEL_ID/index.en.shtml?dateIn=YYYY-MM-DD&nights=N&compositions=N)`

**Format ALL booking links as proper markdown.** Show "Book this property" not "Search on Airbnb".

**Loyalty integration:** When the user has a hotel loyalty programme matching the chain, mention it prominently and include the membership number in the URL where supported. Advise logging in before booking if the URL doesn't support pre-filling the membership.

## Preference Matching

When ranking results, apply the user's profile:
- **Prefer tags** (+15 each): amenities they listed, neighbourhood vibe they described, accommodation type preference
- **Avoid tags** (-50 each): anything from past negative experiences, explicitly unwanted features
- If the user mentioned specific requirements in THIS conversation, those override profile defaults

## Special Handling

- **"Walkable to beach"** -> filter for listings within 500m of beach/coast, prioritise listings mentioning beach access
- **"Small budget"** -> set max to lowest quartile of results, emphasise value-for-money
- **"Direct flights only"** -> this is a flight filter, not accommodation — note it for the flight search
- **"Entire place"** -> exclude shared rooms and hotel rooms, only full apartments/houses
