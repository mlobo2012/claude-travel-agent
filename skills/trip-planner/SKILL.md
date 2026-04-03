---
name: plan-trip
description: "End-to-end trip planner combining transport intelligence, flights, trains, ferries, accommodation, local transit, and activities into a complete itinerary. Use when: user asks to plan a trip, plan a holiday, organise travel, build an itinerary, or wants a full trip planned out."
argument-hint: "<destination> [dates] [party size] [preferences]"
---

# Plan a Trip

Build a complete end-to-end trip itinerary with proactive multi-modal transport, accommodation, local transit, and activities.

## Before Planning

1. **Use the Read tool** on `${CLAUDE_PLUGIN_DATA}/travel-profile.json` to load the user's travel profile. Follow the persistent-memory fallback chain: file first, then Claude's memory, then ask the user.
2. **Also check Claude's memory** for any additional travel preferences or past feedback not in the file
3. **Gather all requirements** — confirm with the user:
   - Destination (required)
   - Travel dates or approximate timing (required)
   - Number of travellers (required)
   - **Ages of any children** (required if family trip — affects transport pricing)
   - Budget level (use profile default if not specified)
   - Must-have experiences or activities
   - Any constraints (dietary, mobility, specific requests)
4. **Detect travel context** (solo/partner/family/group/work) and apply the matching sub-profile

## Planning Sequence

When planning a full trip, invoke all three specialist agents simultaneously using Agent Teams for parallel search. This dramatically reduces wait time.

### Parallel Search Phase (Agent Teams)

Launch these three agents simultaneously:

1. **flight-searcher** agent — Search for flights from the user's home airport to the destination, for the specified dates and party size. Apply flight preferences from profile. Pass: origin, destination, dates, passengers, cabin class, direct-only preference.

2. **accommodation-searcher** agent — Search for accommodation at the destination for the trip dates and party size. Apply accommodation preferences from profile. Pass: destination, check-in, check-out, guests, budget, amenity requirements, prefer/avoid tags.

3. **activities-researcher** agent — Research activities, restaurants, day trips, and experiences at the destination. Pass: destination, dates, trip length, traveller count, any stated interests or preferences.

All three agents run in parallel and return their results independently.

### Combining Results

Once all three agents return results:

1. **Flights first** — Present the top 3 flight options sorted by price, with booking links
2. **Accommodation second** — Present the top 3 accommodation options sorted by preference fit score, with booking links
3. **Activities third** — Present recommended activities grouped by day, with practical details

Then proceed to Step 4 (Practical Information) as before.

### Fallback: Sequential Search

If Agent Teams are not available in the current environment, fall back to sequential execution:

### Step 1: Transport Intelligence

Use the transport-intelligence skill to determine the best transport mode(s):
- Read the user's profile, memory, and trip context
- Analyse the route against the knowledge base
- Determine: should we lead with train, flight, ferry, or show a comparison?
- Note child policies and luggage implications

### Step 2: Transport Search

Based on the transport intelligence recommendation:

**If train recommended:**
- Use the `/find-trains` skill to search for train routes
- Present top 3 options with the train output template
- If the user might still want flights, briefly note: "Flights are also available — want me to search those too?"

**If flight recommended:**
- Use the `/find-flights` skill to search for flights
- Present top 3 options with the flight output template
- If a competitive train route exists, note it: "FYI, [operator] does this route in [time] — want me to check train prices?"

**If ferry recommended:**
- Use the `/find-ferries` skill to search for ferry routes
- Present top 3 options with the ferry output template

**If comparison recommended:**
- Search the top 2 modes and present a side-by-side comparison table
- Make a clear recommendation with reasoning

### Step 3: Accommodation

Use the `/find-accommodation` skill to search for places to stay:
- In the destination for the trip dates
- For the specified party size
- Apply accommodation preferences from profile
- Present top 3 options with the full accommodation output template

### Step 4: Activities & Itinerary

Build a day-by-day itinerary for the trip. Use web search to research:
- Top attractions and activities at the destination
- Local restaurants and food experiences
- Weather expectations for the travel dates
- Any local events during the trip dates
- Family-friendly activities if children are travelling

### Step 5: Local Transit Connections

Use the local-transit skill to provide:
- **Arrival connection**: How to get from the arrival point (airport/station/port) to the accommodation
- **Departure connection**: How to get from the accommodation to the departure point
- **Getting around**: Overview of local transport at the destination
- **Key tips**: Payment methods, passes, child policies, apps to download

### Step 6: Practical Information

Research and include:
- Visa requirements (based on user's passport nationality from profile)
- Local currency and payment tips
- Emergency numbers and nearest hospital/embassy
- Local customs or etiquette tips
- Mobile connectivity (SIM/eSIM options)

## Output Format

Present the complete plan in this structure:

```
# Trip Plan: [Destination]
**[Start Date] — [End Date] | [N] travellers [+ N children age X, Y] | [Context: solo/partner/family/group/work]**

---

## Transport
[Transport options from Step 2 — using the appropriate output template]
[If comparison: side-by-side table with recommendation]

---

## Accommodation
[Accommodation options from Step 3 — use the accommodation output template]

---

## Getting There & Around
[Local transit from Step 5]

### Arrival: [Arrival Point] → Accommodation
[Specific connection advice]

### Getting Around [City]
[Local transport overview]

### Departure: Accommodation → [Departure Point]
[Specific connection advice]

---

## Day-by-Day Itinerary

### Day 1 — [Date] — Arrival
- **[Time]** Arrive at [station/airport/port] ([transport mode]: [operator])
- **[Time]** Transfer to accommodation ([method], ~[cost], ~[time])
- **Afternoon** [Activity suggestion]
- **Evening** [Dinner recommendation with area/restaurant name]

### Day 2 — [Date] — [Theme]
- **Morning** [Activity]
- **Lunch** [Suggestion]
- **Afternoon** [Activity]
- **Evening** [Suggestion]

[...continue for each day...]

### Day [N] — [Date] — Departure
- **[Time]** Check out
- **[Time]** Transfer to [station/airport/port] ([method], ~[cost])
- **[Time]** Depart ([transport mode]: [operator])

---

## Practical Info
- **Visa:** [Requirement based on passport]
- **Currency:** [Local currency] — [exchange/payment tips]
- **Getting around:** [Transport summary]
- **Weather:** [Expected conditions for travel dates]
- **Emergency:** [Key numbers]
- **Connectivity:** [SIM/eSIM options]

---

## Budget Summary
| Category | Estimated Cost |
|----------|---------------|
| Transport | [LIVE PRICE] per person [mode noted] |
| Accommodation | [LIVE PRICE] total for [N] nights |
| Local transit | [Estimated range — clearly marked as estimate] |
| Activities | [Estimated range — clearly marked as estimate] |
| Food | [Estimated range — clearly marked as estimate] |
| **Total** | **[Sum]** |

> Live-verified prices marked with timestamps. Estimated costs marked with "est." — verify locally.
> All prices retrieved at [HH:MM today].
```

## Important Rules

1. **Transport, flight, and accommodation prices must be live-verified** following the guardrails skill
2. **Activity and food costs can be estimated** but must be clearly marked as estimates, not live prices
3. **Never make up transport times, prices, or accommodation details** — only use what the MCP tools return
4. **Always run transport intelligence first** — don't default to flights without analysing the route
5. **If children are travelling, always highlight fare savings** — "Kids ride free on DB trains" or "Child fare saves £X vs flights"
6. **If the user said "entire trip planned out"** — include EVERYTHING: transport, accommodation, local transit, daily itinerary, practical info, and budget summary. Don't give a partial plan.
7. **Use the Write tool** to save the trip plan to `${CLAUDE_PLUGIN_DATA}/trips/[destination]-[date].json` for future reference
8. **Format all links as proper markdown** — `[Book on Ryanair](https://...)` — NOT angle brackets, NOT plain text, NOT `<Ryanair.com>`. This applies to booking links, accommodation links, and all URLs in the plan.

## Saving Trip Data

After presenting the plan, save a summary to the trip history:

```json
{
  "destination": "Paris",
  "dates": { "start": "2026-05-15", "end": "2026-05-20" },
  "travellers": 4,
  "children": [{ "age": 8 }, { "age": 5 }],
  "context": "family",
  "transport": {
    "mode": "train",
    "operator": "Eurostar",
    "price_pp": "...",
    "child_policy": "Under 4 free, 4-11 ~30% off",
    "verified_at": "..."
  },
  "accommodation": { "name": "...", "price_total": "...", "verified_at": "..." },
  "status": "planned",
  "created_at": "2026-04-03"
}
```
