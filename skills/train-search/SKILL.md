---
name: find-trains
description: "Search for train routes using public transport MCP and cross-check with other sources. Live-verified prices with anti-hallucination guardrails. Use when: user asks to find trains, search for rail tickets, take the train, check train prices, or travel by rail."
argument-hint: "<origin> to <destination> [dates] [preferences]"
---

# Find Trains

Search for train routes using live MCP data with full quality verification.

## Before Searching

1. **Load the user's travel profile** using the persistent-memory fallback chain:
   - Try reading `${CLAUDE_PLUGIN_DATA}/travel-profile.json` first
   - If not found, check Claude's memory for travel profile facts
   - If neither exists, ask the essential questions for this search
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

### Primary Search: Web Search

Search for trains on the operator's direct website and rail booking platforms:
- Eurostar: eurostar.com
- UK National Rail: thetrainline.com, nationalrail.co.uk
- Germany: bahn.de
- France: sncf-connect.com
- US: amtrak.com
- Cross-platform: trainline.eu, omio.com

### Cross-Check

Cross-check the top results on at least one additional source (e.g., operator site vs Trainline) — flag price discrepancies >5%.

## Output Template

Present each train option in this format:

```
### [NUMBER]. [OPERATOR] — [ORIGIN] to [DESTINATION]

**Outbound:** [DATE] | Departs [TIME] [STATION] -> Arrives [TIME] [STATION] ([DURATION])
**Return:** [DATE] | Departs [TIME] [STATION] -> Arrives [TIME] [STATION] ([DURATION])
**Stops:** [Direct / N stop(s) via CITY]
**Class:** [Standard / First]
**Price:** [EXACT PRICE] per adult | [TOTAL FOR ALL PASSENGERS] total
**Child fare:** [EXACT CHILD PRICE or "FREE (under X with adult)" or "X% off adult fare"]
**Luggage:** [Policy — usually "Unlimited free bags"]
**Verified:** [HH:MM today] via [Source]
**Loyalty:** [If applicable: "Earns [N] points on [programme]"]
**Book this train:** [DEEP BOOKING LINK — see section below]

> **Why this?** [Specific reasoning citing profile preferences, transport rule, child fare savings, or loyalty match]
```

After all options:

```
---
Train prices can vary by demand — these were retrieved at [HH:MM today].
Book direct with the operator for best prices. Advance tickets are usually cheapest.
[If applicable: "Only [source] was available for pricing — not cross-checked."]
```

## Deep Booking Links

**ALWAYS generate deep links that take the user to the specific journey, not a generic homepage.**

For known operators, construct deep links with dates, route, and passenger count pre-filled:

- **Eurostar:** `[Book this Eurostar](https://www.eurostar.com/uk-en/train/booking/options?origin=STATION_CODE&destination=STATION_CODE&outbound-date=YYYY-MM-DD&adult=N&youth=0&child=0)`
  - Station codes: London St Pancras = `7015400`, Paris Gare du Nord = `8727100`, Brussels-Midi = `8814001`, Amsterdam Centraal = `8400058`
  - If user has Club Eurostar number, mention: "Your Club Eurostar number [XXXX] — log in before booking to earn points."
- **Trainline (UK):** `[Book on Trainline](https://www.thetrainline.com/book/results?origin=STATION_ID&destination=STATION_ID&outwardDate=YYYY-MM-DDTHH%3A00%3A00&outwardDateType=departAfter&journeySearchType=single&passengers%5B%5D=YYYY-MM-DD%7Cadult&temporalDirection=departing)`
- **Trainline (EU):** `[Book on Trainline EU](https://www.trainline.eu/search/ORIGIN/DESTINATION/YYYY-MM-DD)`
- **Deutsche Bahn:** `[Book on DB](https://int.bahn.de/en/buchung/fahrplanauskunft?reise.von=ORIGIN&reise.nach=DESTINATION&reise.datum=YYYY-MM-DD&reise.zeit=HH%3AMM&reise.zeitpunktart=ABFAHRT)`
- **SNCF Connect:** `[Book on SNCF](https://www.sncf-connect.com/en-en/train/results?departure=ORIGIN&arrival=DESTINATION&date=YYYY-MM-DD&passengers=1adult)`
- **Omio:** `[Book on Omio](https://www.omio.com/search-frontend/results/ORIGIN/DESTINATION?date=YYYY-MM-DD&passengers=N)`
- **Amtrak:** `[Book on Amtrak](https://www.amtrak.com/tickets/departure.html?origin=XXX&destination=YYY&departing=MM/DD/YYYY&adults=N)`

**Format ALL booking links as proper markdown.** Show "Book this train" not "Search on Trainline".

**Loyalty integration:** When the user has a rail loyalty programme (e.g., Club Eurostar), mention it in the output and advise them to log in before booking. Include membership number where the URL supports it.

## Child & Family Fare Handling

This is a key differentiator for train travel. Always highlight child policies:

1. **Check the transport-intelligence knowledge base** for the operator's child policy
2. **Calculate the actual family cost** — show the total for adults + children, noting any free/discounted fares
3. **Compare to equivalent flight cost** if the user is deciding between modes: "Family of 4 (2 adults, 2 kids age 8 and 5): Train £180 total vs Flight £560 total (kids pay full fare + baggage)"

## Preference Matching

Apply the user's travel preferences:
- **Comfort priority** -> show first class options, highlight legroom and quiet coaches
- **Budget priority** -> sort by price, highlight advance purchase savings
- **Eco preference** -> note the CO2 comparison (trains typically emit 80-90% less CO2 than flights per passenger)
- **Speed priority** -> show fastest direct services, compare door-to-door with flights
- **Scenic preference** -> flag scenic routes from the transport-intelligence knowledge base

## Special Handling

- **UK rail** -> note railcard discounts if the user has one (1/3 off for most). Search National Rail or Trainline.
- **European rail** -> check if an Interrail/Eurail pass would be cheaper for multi-leg journeys
- **US rail** -> check if the USA Rail Pass ($499/10 segments/30 days) applies for multi-city trips
- **Overnight trains** -> present as transport + accommodation combo, note what's included (bed, breakfast, etc.)
- **Flexible dates** -> if supported by the MCP or booking platform, search +/- 2 days for cheapest fares
