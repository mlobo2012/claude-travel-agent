---
name: trip-pack
description: "Generate a branded pre-departure trip pack document with all trip details, itinerary, practical info, and packing suggestions. Auto-generates T-2 days before departure or on-demand. Use when: user says 'send me my trip pack', 'trip summary', or 'departure briefing'."
argument-hint: "[destination or trip reference]"
---

# Trip Pack Generator

Generate a comprehensive, branded pre-departure document compiling all trip information into a single reference.

## When to Generate

- **Automatic:** Via scheduled task, T-2 days before departure (created when a trip is booked)
- **On-demand:** User says "send me my trip pack", "trip summary", "departure briefing", or runs `/trip-pack`

## Data Sources

Compile data from:
1. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` for trip details, booking references, dates (fall back to Claude's memory for `travel_past_trips` if file not found)
2. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel_profile.json` for passport, home airport, preferences (fall back to Claude's memory for `travel_profile` if file not found)
3. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel_packing_preferences.json` for personal packing must-haves (fall back to Claude's memory for `travel_packing_preferences` if file not found)
4. **Web search** — current weather forecast for destination during travel dates
5. **Web search** — practical info (currency exchange rates, emergency numbers, local transport)

## Trip Pack Template

Generate the following document and send via Dispatch:

```
================================================
       AI HEROES TRAVEL AGENT
       Your Trip Pack
================================================

TRIP SUMMARY
Destination:    [City, Country]
Dates:          [Start Date] to [End Date] ([N] nights)
Travellers:     [Number] ([names if known])
Trip type:      [Solo / Partner / Group / Work]

------------------------------------------------
FLIGHTS
------------------------------------------------

OUTBOUND
  [Airline] [Flight number if known]
  [Origin airport] -> [Destination airport]
  [Date] | Departs [Time] | Arrives [Time]
  Terminal: [if known]
  Booking ref: [if saved]
  Arrive at airport by: [departure minus 2hr domestic / 3hr international]

RETURN
  [Airline] [Flight number if known]
  [Destination airport] -> [Origin airport]
  [Date] | Departs [Time] | Arrives [Time]
  Terminal: [if known]
  Booking ref: [if saved]

------------------------------------------------
ACCOMMODATION
------------------------------------------------

  [Property name]
  [Full address]
  Check-in:  [Date] from [Time]
  Check-out: [Date] by [Time]
  Booking ref: [if saved]
  Host contact: [if available]

  Key info: [WiFi password, door code, parking — if saved]

------------------------------------------------
DAY-BY-DAY ITINERARY
------------------------------------------------

Day 1 — [Date] — Arrival
  [Time] Arrive at [airport]
  [Time] Transfer to accommodation ([method])
  Afternoon: [Activity suggestion]
  Evening: [Dinner recommendation]

Day 2 — [Date] — [Theme]
  Morning: [Activity]
  Lunch: [Suggestion]
  Afternoon: [Activity]
  Evening: [Suggestion]

[...continue for each day...]

Day [N] — [Date] — Departure
  [Time] Check out
  [Time] Transfer to airport
  [Time] Depart

------------------------------------------------
WEATHER FORECAST
------------------------------------------------

  [Destination] ([Date range]):
  [Day-by-day forecast summary from web search]
  Temperature range: [Low]C to [High]C
  Conditions: [Summary]

  Suggested clothing: [Based on forecast]

------------------------------------------------
PRACTICAL INFORMATION
------------------------------------------------

  Currency:     [Local currency] ([current exchange rate vs GBP])
  Language:     [Local language] — [key phrases if non-English]
  Time zone:    [Timezone] ([+/- vs UK])
  Electricity:  [Plug type] — [adaptor needed? Y/N]
  Tipping:      [Local custom]
  Water:        [Safe to drink from tap? Y/N]
  Transport:    [Best way to get around locally]
  Emergency:    [Local emergency number]
  UK Embassy:   [Nearest UK embassy/consulate address + phone]

  Visa:         [Required? Based on passport nationality]
  Passport:     Expires [date from profile] — [OK / WARNING if <6 months]

------------------------------------------------
PACKING CHECKLIST
------------------------------------------------

  Essentials:
  [ ] Passport (expires [date])
  [ ] Boarding passes / booking confirmations
  [ ] Travel insurance documents
  [ ] Phone charger
  [ ] [Travel adaptor if needed for destination]

  Clothing (based on weather):
  [ ] [Weather-appropriate items]

  Personal must-haves (from your preferences):
  [ ] [Items from travel_packing_preferences]

================================================
Prepared by your AI Heroes Travel Agent
ai-heroes.co
================================================
```

## Scheduled Task Setup

When a trip is confirmed as "booked":

1. Calculate T-2 date (2 days before departure)
2. Create a scheduled task for 9:00 AM on T-2
3. Task prompt: "Generate and send the trip pack for the user's [destination] trip on [dates]. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json`, `${CLAUDE_PLUGIN_DATA}/travel_profile.json`, and `${CLAUDE_PLUGIN_DATA}/travel_packing_preferences.json` to load trip data (fall back to Claude's memory if files not found). Use web search for current weather forecast and practical info. Format using the trip pack template. Send via Dispatch."

## On-Demand Generation

When the user requests a trip pack:

1. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` to check for trips (fall back to Claude's memory for `travel_past_trips` if file not found)
2. If multiple upcoming trips, ask which one
3. If one upcoming trip, generate for that
4. If no upcoming trips, say "No upcoming trips found. Plan a trip first with /plan-trip."

## Important Rules

1. **Weather must be current** — always use web search at generation time, never use cached weather
2. **Exchange rates must be current** — use web search for today's rate
3. **Never fabricate booking references** — only include if saved in memory
4. **If any section has no data**, include the section header with "Not yet confirmed — [action needed]"
5. **Passport expiry warning** — if passport expires within 6 months of travel date, add a prominent warning
