---
name: find-ferries
description: "Search for ferry routes using Ferryhopper MCP and cross-check with other sources. Live-verified prices with anti-hallucination guardrails. Use when: user asks to find ferries, take a boat, island hopping, or travel by sea."
argument-hint: "<origin> to <destination> [dates] [preferences]"
---

# Find Ferries

Search for ferry routes using live MCP data with full quality verification.

## Before Searching

1. **Use the Read tool** on `${CLAUDE_PLUGIN_DATA}/travel-profile.json` to load the user's travel profile. Follow the persistent-memory fallback chain: file first, then Claude's memory, then ask the user.
2. **Check the transport-intelligence knowledge base** for the route — it may have operator info and journey times
3. **Confirm search parameters** with the user if any are missing:
   - Origin port/island/city (required)
   - Destination port/island/city (required)
   - Departure date (required)
   - Return date (if round trip)
   - Number of passengers (required)
   - Passenger ages — especially children
   - Vehicle? (car, motorbike, bicycle, foot passenger)
   - Cabin preference for overnight routes (deck seat / cabin / en-suite cabin)

## Search Execution

### Primary Search: Ferryhopper MCP

Use the `ferryhopper` MCP server tools to search. Pass:
- Origin and destination ports
- Travel dates
- Number of passengers (with ages)
- Vehicle type if applicable

### Cross-Check: Web Search

For top results from the MCP:
- Cross-check via web search on the operator's direct website (e.g., bluestarferries.com, dfds.com, stenaline.co.uk, brittanyferries.com)
- Compare prices — flag discrepancies >5%

### Fallback

If the Ferryhopper MCP is not connected or returns errors:
- Use web search to find ferries on DirectFerries.com, the operator's website, or FerryScanner
- Note that results are from web search, not MCP

## Output Template

Present each ferry option in this format:

```
### [NUMBER]. [OPERATOR] — [ORIGIN] to [DESTINATION]

**Outbound:** [DATE] | Departs [TIME] [PORT] → Arrives [TIME] [PORT] ([DURATION])
**Return:** [DATE] | Departs [TIME] [PORT] → Arrives [TIME] [PORT] ([DURATION])
**Vessel:** [Ship name if available] | [Type: high-speed / conventional]
**Accommodation:** [Deck seat / Cabin type] — [what's included]
**Price:** [EXACT PRICE] per person | [TOTAL] total [+ vehicle £X if applicable]
**Child fare:** [EXACT CHILD PRICE or "FREE (under X)" or "X% off"]
**Luggage:** [Policy]
**Verified:** [HH:MM today] via [Source]
**Book this ferry:** [DEEP BOOKING LINK — see section below]

> [One-sentence note — e.g., "Overnight sailing doubles as accommodation — saves a hotel night"]
```

After all options:

```
---
Ferry prices vary by season and vehicle type — these were retrieved at [HH:MM today].
Book direct with the operator for best availability, especially in peak season.
[If applicable: "Only [source] was available for pricing — not cross-checked."]
```

## Deep Booking Links

**ALWAYS generate deep links that take the user to the specific route search, not a generic homepage.** Construct deep booking links with route, dates, and passenger count pre-filled where possible.

For known ferry platforms and operators, construct deep links:

- **Ferryhopper:** `[Book this ferry on Ferryhopper](https://www.ferryhopper.com/en/ferry-routes/ORIGIN-DESTINATION?departureDate=YYYY-MM-DD&passengers=N)`
- **Direct Ferries:** `[Book this ferry on Direct Ferries](https://www.directferries.com/routes/ORIGIN_DESTINATION.htm?departureDate=YYYY-MM-DD&adults=N)`
- **DFDS:** `[Book this ferry](https://www.dfds.com/en/passenger-ferries/ORIGIN-DESTINATION?outbound=YYYY-MM-DD&adults=N)`
- **Stena Line:** `[Book this ferry](https://www.stenaline.co.uk/routes/ORIGIN-DESTINATION?date=YYYY-MM-DD&adults=N)`
- **P&O Ferries:** `[Book this ferry](https://www.poferries.com/en/routes/ORIGIN-DESTINATION?departureDate=YYYY-MM-DD&adults=N)`
- **Brittany Ferries:** `[Book this ferry](https://www.brittany-ferries.co.uk/ferry-routes/ORIGIN-DESTINATION?departureDate=YYYY-MM-DD&adults=N)`
- **Blue Star Ferries:** `[Book this ferry](https://www.bluestarferries.com/en/booking?from=ORIGIN&to=DESTINATION&date=YYYY-MM-DD&passengers=N)`
- **Viking Line:** `[Book this ferry](https://www.vikingline.com/find-a-cruise-or-ferry/?route=ORIGIN-DESTINATION&date=YYYY-MM-DD&adults=N)`
- **Jadrolinija:** `[Book this ferry](https://www.jadrolinija.hr/en/ferry-croatia?from=ORIGIN&to=DESTINATION&date=YYYY-MM-DD&adults=N)`
- **For other operators / fallback:** `[Book on Ferryhopper](https://www.ferryhopper.com/en/ferry-routes/ORIGIN-DESTINATION)` or `[Book on Direct Ferries](https://www.directferries.com/routes/ORIGIN_DESTINATION.htm)`

**Link text MUST name the specific option.** Write "Book DFDS Dover→Calais 14:30 →" NOT "Search on Ferryhopper →" or "Book this ferry". Every link label must include the operator, route, and departure time. Format ALL booking links as proper markdown.

**Children in URLs:** When children are in the party, include child parameters:
- Ferryhopper: `&children=N`
- Direct Ferries: `&children=N&childAges=AGE1,AGE2`
- DFDS: `&children=N`
- Stena Line: `&children=N`
- P&O: `&children=N`
- Brittany Ferries: `&children=N&childAges=AGE1`

**Operator direct vs aggregator:** When an operator's direct booking link is available, prefer it. Use Ferryhopper or Direct Ferries as a fallback or for price comparison. If showing both, label clearly: "Book direct" vs "Compare on Ferryhopper".

**Link verification (optional):** If a browser MCP or agent-browser tool is available, verify that the constructed booking URL loads the correct route before presenting it to the user.

## Overnight Ferry Handling

For routes over 6 hours, overnight ferries combine transport and accommodation:

1. **Calculate the combined value**: "This overnight ferry replaces one night's accommodation. Ferry cabin £85 vs flight £120 + hotel £95 = £130 saved."
2. **Present cabin options**: deck seat (cheapest), inside cabin, outside cabin, en-suite
3. **Note onboard facilities**: restaurants, bars, duty-free, children's play areas, Wi-Fi
4. **Highlight the time advantage**: "Departs 17:00, arrive 09:00 — you travel overnight and wake up at your destination"

## Preference Matching

Apply the user's travel preferences:
- **Budget priority** → show deck/seat options first, note any early booking discounts
- **Comfort priority** → show cabin options, highlight en-suite and premium lounges
- **Family travel** → note children's facilities, play areas, family cabins
- **Eco preference** → note that modern ferries are more efficient than flights for the same route
- **Car travel** → always include vehicle pricing, note check-in times (usually 1-2h before)

## Special Handling

- **Greek island hopping** → suggest multi-island itineraries with connecting ferries. Note that high-speed catamarans are faster but more expensive and more affected by weather.
- **UK ↔ Continental Europe** → compare with Eurostar/flights. For car travel, ferry is often the only practical option.
- **Peak season (June-September)** → warn about availability: "Peak season — book early, especially for vehicle crossings."
- **Weather sensitivity** → high-speed ferries may be cancelled in rough seas. Note: "High-speed service — may be affected by weather. Conventional ferry is a slower but more reliable backup."
- **Multi-leg ferry journeys** → check if island-hopping passes are available (e.g., Greek island passes)
