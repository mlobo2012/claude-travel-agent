---
name: find-flights
description: "Search for flights using Google Flights MCP and cross-check with other sources. Live-verified prices with anti-hallucination guardrails. Use when: user asks to find flights, search for flights, book flights, check flight prices, or fly somewhere."
argument-hint: "<origin> to <destination> [dates] [preferences]"
---

# Find Flights

Search for flights using live MCP data with full quality verification.

## Before Searching

1. **Read the user's travel profile** from `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
2. **Confirm search parameters** with the user if any are missing:
   - Origin airport/city (use profile's `home_airport` as default)
   - Destination airport/city (required)
   - Departure date (required)
   - Return date (required for round trips; ask if one-way)
   - Number of passengers (required)
   - Cabin class (use profile default if not specified)
   - Direct flights only? (ask if not specified)
   - Flexible dates? (+/- days)

## Search Execution

### Primary Search: Google Flights MCP

Use the `google-flights` MCP server tools to search. Pass:
- Origin and destination airports
- Travel dates
- Number of passengers
- Cabin class preference
- Non-stop filter if "direct flights only" requested

### Cross-Check: Web Search

For top Google Flights results:
- Cross-check via web search on Skyscanner or airline direct websites
- Compare prices — flag discrepancies >5%

### Fallback

If the Google Flights MCP is not connected or returns errors:
- Use web search to find flights on Google Flights, Skyscanner, or airline websites directly
- Note that results are from web search, not MCP

## Output Template

Present each flight option in this format:

```
### [NUMBER]. [AIRLINE] — [ORIGIN] to [DESTINATION]

**Outbound:** [DATE] | Departs [TIME] → Arrives [TIME] ([DURATION])
**Return:** [DATE] | Departs [TIME] → Arrives [TIME] ([DURATION])
**Stops:** [Direct / N stop(s) via CITY]
**Price:** [EXACT PRICE] per person | [TOTAL FOR ALL PASSENGERS] total
**Verified:** [HH:MM today] via [Source]
**Class:** [CABIN CLASS]
**Booking:** [DIRECT URL or "Search on Google Flights: [search URL]"]

> [One-sentence note on why this suits them — e.g., "Direct flight, matches your aisle seat preference"]
```

After all options:

```
---
Flight prices are highly volatile — these were retrieved at [HH:MM today].
Verify current prices and availability before booking.
[If applicable: "Only [source] was available for pricing — not cross-checked."]
```

## Preference Matching

Apply the user's flight preferences:
- **Preferred airlines** → prioritise, but still show alternatives if significantly cheaper
- **Seat preference** → note in recommendation but this is chosen at booking
- **Cabin class** → filter by preference, show upgrades only if within 30% price difference
- **Direct flights preference** → if specified, exclude connections unless nothing direct exists (then note it)

## Special Handling

- **"Direct flights only from London"** → search all London airports (LHR, LGW, STN, LTN, SEN, LCY) unless a specific one is given. Note which airport each result departs from.
- **Budget-conscious** → sort by price, highlight cheapest viable option
- **Flexible dates** → if the MCP supports date range search, use it. Otherwise, search the target date +/- 2 days and present the cheapest option across those dates.
- **Multi-city** → break into separate searches, present each leg clearly
