---
name: find-trains
description: "Search for train routes using public transport MCP and cross-check with other sources. Live-verified prices with anti-hallucination guardrails. Use when: user asks to find trains, search for rail tickets, take the train, check train prices, or travel by rail."
argument-hint: "<origin> to <destination> [dates] [preferences]"
---

# Find Trains

Search for train routes using live MCP data with full quality verification.

## Before Searching

1. **Read the user's travel profile** from `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
2. **Check the transport-intelligence knowledge base** for the route — it may have operator info, child policies, and journey times
3. **Confirm search parameters** with the user if any are missing:
   - Origin city/station (use profile's `home_city` as default)
   - Destination city/station (required)
   - Departure date (required)
   - Return date (required for round trips; ask if one-way)
   - Number of passengers (required)
   - Passenger ages — especially children (required for accurate child pricing)
   - Class preference (standard/first — use profile default if not specified)
   - Flexible dates? (+/- days)

## Search Execution

### Primary Search: Public Transport MCP

Use the `public-transport` MCP server tools to search. Pass:
- Origin and destination stations/cities
- Travel dates
- Number of passengers
- Class preference

### Cross-Check: Web Search

For top results from the MCP:
- Cross-check via web search on the operator's direct website (e.g., eurostar.com, thetrainline.com, bahn.de, sncf-connect.com, amtrak.com)
- Compare prices — flag discrepancies >5%

### Fallback

If the public-transport MCP is not connected or returns errors:
- Use web search to find trains on Trainline, the operator's website, or rail booking platforms
- Note that results are from web search, not MCP

## Output Template

Present each train option in this format:

```
### [NUMBER]. [OPERATOR] — [ORIGIN] to [DESTINATION]

**Outbound:** [DATE] | Departs [TIME] [STATION] → Arrives [TIME] [STATION] ([DURATION])
**Return:** [DATE] | Departs [TIME] [STATION] → Arrives [TIME] [STATION] ([DURATION])
**Stops:** [Direct / N stop(s) via CITY]
**Class:** [Standard / First]
**Price:** [EXACT PRICE] per adult | [TOTAL FOR ALL PASSENGERS] total
**Child fare:** [EXACT CHILD PRICE or "FREE (under X with adult)" or "X% off adult fare"]
**Luggage:** [Policy — usually "Unlimited free bags"]
**Verified:** [HH:MM today] via [Source]
**Booking:** [DIRECT URL to operator or booking platform]

> [One-sentence note on why this suits them — e.g., "City centre to city centre, kids ride free, no luggage fees"]
```

After all options:

```
---
Train prices can vary by demand — these were retrieved at [HH:MM today].
Book direct with the operator for best prices. Advance tickets are usually cheapest.
[If applicable: "Only [source] was available for pricing — not cross-checked."]
```

## Child & Family Fare Handling

This is a key differentiator for train travel. Always highlight child policies:

1. **Check the transport-intelligence knowledge base** for the operator's child policy
2. **Calculate the actual family cost** — show the total for adults + children, noting any free/discounted fares
3. **Compare to equivalent flight cost** if the user is deciding between modes: "Family of 4 (2 adults, 2 kids age 8 and 5): Train £180 total vs Flight £560 total (kids pay full fare + baggage)"

## Preference Matching

Apply the user's travel preferences:
- **Comfort priority** → show first class options, highlight legroom and quiet coaches
- **Budget priority** → sort by price, highlight advance purchase savings
- **Eco preference** → note the CO2 comparison (trains typically emit 80-90% less CO2 than flights per passenger)
- **Speed priority** → show fastest direct services, compare door-to-door with flights
- **Scenic preference** → flag scenic routes from the transport-intelligence knowledge base

## Special Handling

- **UK rail** → note railcard discounts if the user has one (1/3 off for most). Search National Rail or Trainline.
- **European rail** → check if an Interrail/Eurail pass would be cheaper for multi-leg journeys
- **US rail** → check if the USA Rail Pass ($499/10 segments/30 days) applies for multi-city trips
- **Overnight trains** → present as transport + accommodation combo, note what's included (bed, breakfast, etc.)
- **Flexible dates** → if supported by the MCP or booking platform, search +/- 2 days for cheapest fares
