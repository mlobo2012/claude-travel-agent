---
name: find-flights
description: "Search for flights using Google Flights MCP and cross-check with other sources. Live-verified prices with anti-hallucination guardrails. Use when: user asks to find flights, search for flights, book flights, check flight prices, or fly somewhere."
argument-hint: "<origin> to <destination> [dates] [preferences]"
---

# Find Flights

Search for flights using live MCP data with full quality verification.

## Before Searching

1. **Load the user's travel profile** using the persistent-memory fallback chain:
   - Try reading `${CLAUDE_PLUGIN_DATA}/travel-profile.json` first
   - If not found, check Claude's memory for travel profile facts
   - If neither exists, ask the essential questions for this search
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

**Outbound:** [DATE] | Departs [TIME] -> Arrives [TIME] ([DURATION])
**Return:** [DATE] | Departs [TIME] -> Arrives [TIME] ([DURATION])
**Stops:** [Direct / N stop(s) via CITY]
**Price:** [EXACT PRICE] per person | [TOTAL FOR ALL PASSENGERS] total
**Verified:** [HH:MM today] via [Source]
**Class:** [CABIN CLASS]
**Loyalty:** [If applicable: "Earns [N] [points currency] on [programme]" or "Alliance partner of [programme]"]
**Book this flight:** [DEEP BOOKING LINK — see section below]

> **Why this?** [Specific reasoning citing profile preferences, loyalty tier, transport rule, or price advantage]
```

After all options:

```
---
Flight prices are highly volatile — these were retrieved at [HH:MM today].
Verify current prices and availability before booking.
[If applicable: "Only [source] was available for pricing — not cross-checked."]
```

## Deep Booking Links

**ALWAYS generate deep links that take the user to the specific flight, not a generic homepage.** The Google Flights MCP returns a `booking_token`, NOT a booking URL. Construct deep booking links manually.

For known airlines, construct deep links with dates, route, and passenger count pre-filled:

- **Ryanair:** `[Book this Ryanair flight](https://www.ryanair.com/gb/en/trip/flights/select?adults=N&teens=0&children=0&infants=0&dateOut=YYYY-MM-DD&dateIn=YYYY-MM-DD&isConnectedFlight=false&discount=0&isReturn=true&promoCode=&originIata=XXX&destinationIata=YYY)`
- **easyJet:** `[Book this easyJet flight](https://www.easyjet.com/en/booking/select-flight?pid=www.easyjet.com&origin=XXX&destination=YYY&oday=DD&omonth=MM&oyear=YYYY&rday=DD&rmonth=MM&ryear=YYYY&adults=N&children=0&infants=0)`
- **British Airways:** `[Book this BA flight](https://www.britishairways.com/travel/book/public/en_gb?from=XXX&to=YYY&depDate=YYYYMMDD&retDate=YYYYMMDD&cabin=M&adultCount=N&youngAdultCount=0&childCount=0&infantCount=0&eTicket=e)`
  - If user has BA Executive Club number, append: `&eId=MEMBERSHIP_NUMBER`
- **Wizz Air:** `[Book this Wizz Air flight](https://wizzair.com/en-gb/booking/select-flight/XXX/YYY/YYYY-MM-DD/YYYY-MM-DD/N/0/0/0)`
- **Vueling:** `[Book this Vueling flight](https://www.vueling.com/en/booking/select?origin=XXX&destination=YYY&outboundDate=YYYY-MM-DD&inboundDate=YYYY-MM-DD&adults=N)`
- **Lufthansa:** `[Book this Lufthansa flight](https://www.lufthansa.com/gb/en/flight-search?origin=XXX&destination=YYY&outDate=YYYY-MM-DD&inDate=YYYY-MM-DD&paxCombination=N-0-0-0-0)`
- **KLM:** `[Book this KLM flight](https://www.klm.com/search/result?pax=N&cabin=ECONOMY&lang=en&origins=XXX&destinations=YYY&outDate=YYYY-MM-DD&inDate=YYYY-MM-DD)`
- **For other airlines / fallback:** `[Book on Google Flights](https://www.google.com/travel/flights?q=Flights%20to%20YYY%20from%20XXX%20on%20YYYY-MM-DD)`

**Format ALL booking links as proper markdown.** Never use angle brackets or plain text URLs.

**Loyalty number inclusion:** When the user has a loyalty programme for the airline (or its alliance partner) and the booking URL supports it, include the membership number in the URL parameter. Mention it in the output: "Your [programme] number [XXXX] will be applied at booking."

## Preference Matching

Apply the user's flight preferences:
- **Preferred airlines** -> prioritise, but still show alternatives if significantly cheaper
- **Seat preference** -> note in recommendation ("matches your window seat preference")
- **Cabin class** -> filter by preference, show upgrades only if within 30% price difference
- **Direct flights preference** -> if specified, exclude connections unless nothing direct exists (then note it)

## Special Handling

- **"London" or "from London"** -> search ALL major London airports: LHR (Heathrow), LGW (Gatwick), STN (Stansted), LTN (Luton), SEN (Southend), LCY (City). Present results grouped by airport with the best option highlighted. Only search a single airport if the user specifies one.
- **Budget-conscious** -> sort by price, highlight cheapest viable option
- **Flexible dates** -> if the MCP supports date range search, use it. Otherwise, search the target date +/- 2 days and present the cheapest option across those dates.
- **Multi-city** -> break into separate searches, present each leg clearly
