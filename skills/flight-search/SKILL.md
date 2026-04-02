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

#### Google Flights MCP Parameter Reference
Use EXACTLY these parameter names — the MCP will reject incorrect names:
- `departure_date` (NOT `date`) — format: YYYY-MM-DD
- `return_date` — format: YYYY-MM-DD
- `origin` — airport IATA code (e.g., LTN, LGW, STN)
- `destination` — airport IATA code (e.g., CTA)
- `adults` — number of adult passengers
- `non_stop` — true/false for direct flights only
- `currency` — e.g., "GBP"

#### Date Grid Limitation
The `get_date_grid` tool may not return results for dates more than 2-3 months ahead. For dates further out, skip the date grid and use `search_flights` directly with specific dates.

#### London Airport Handling
When the user says "London" or "from London" without specifying an airport, search ALL major London airports: LHR (Heathrow), LGW (Gatwick), STN (Stansted), LTN (Luton), SEN (Southend). Present results grouped by airport with the best option highlighted.

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
**Booking:** [BOOKING LINK — see Booking Links section below]

> [One-sentence note on why this suits them — e.g., "Direct flight, matches your aisle seat preference"]
```

After all options:

```
---
Flight prices are highly volatile — these were retrieved at [HH:MM today].
Verify current prices and availability before booking.
[If applicable: "Only [source] was available for pricing — not cross-checked."]
```

## Booking Links

The Google Flights MCP returns a `booking_token`, NOT a booking URL. You MUST construct booking links manually for the user. ALWAYS include booking links — never present flights without a way to book.

For known airlines, construct direct booking links:
- **Ryanair:** `[Book on Ryanair](https://www.ryanair.com/gb/en/trip/flights/select?adults=N&dateOut=YYYY-MM-DD&origin=XXX&destination=YYY)`
- **easyJet:** `[Book on easyJet](https://www.easyjet.com/en/booking/select-flight?origin=XXX&destination=YYY&date=YYYY-MM-DD&adults=N)`
- **British Airways:** `[Book on British Airways](https://www.britishairways.com/travel/book/public/en_gb?from=XXX&to=YYY&depDate=YYYY-MM-DD&adults=N)`
- **For other airlines:** `[Book on Google Flights](https://www.google.com/travel/flights/booking?token=BOOKING_TOKEN)`

Format ALL booking links as proper markdown: `[Book on Ryanair](https://...)` — NOT angle brackets, NOT plain text, NOT `<Ryanair.com>`.

## Preference Matching

Apply the user's flight preferences:
- **Preferred airlines** → prioritise, but still show alternatives if significantly cheaper
- **Seat preference** → note in recommendation but this is chosen at booking
- **Cabin class** → filter by preference, show upgrades only if within 30% price difference
- **Direct flights preference** → if specified, exclude connections unless nothing direct exists (then note it)

## Special Handling

- **"London" or "from London"** → search ALL major London airports: LHR (Heathrow), LGW (Gatwick), STN (Stansted), LTN (Luton), SEN (Southend), LCY (City). Present results grouped by airport with the best option highlighted. Only search a single airport if the user specifies one.
- **Budget-conscious** → sort by price, highlight cheapest viable option
- **Flexible dates** → if the MCP supports date range search, use it. Otherwise, search the target date +/- 2 days and present the cheapest option across those dates.
- **Multi-city** → break into separate searches, present each leg clearly
