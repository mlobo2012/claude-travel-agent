---
name: plan-trip
description: "End-to-end trip planner combining flights, accommodation, and activities into a complete itinerary. Use when: user asks to plan a trip, plan a holiday, organise travel, build an itinerary, or wants a full trip planned out."
argument-hint: "<destination> [dates] [party size] [preferences]"
---

# Plan a Trip

Build a complete end-to-end trip itinerary combining flights, accommodation, and activities.

## Before Planning

1. **Read the user's travel profile** from `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
2. **Gather all requirements** — confirm with the user:
   - Destination (required)
   - Travel dates or approximate timing (required)
   - Number of travellers (required)
   - Budget level (use profile default if not specified)
   - Must-have experiences or activities
   - Any constraints (dietary, mobility, specific requests)
3. **Detect travel context** (solo/partner/group/work) and apply the matching sub-profile

## Planning Sequence

Execute these in order. Each step uses live-verified data following the research pipeline.

### Step 1: Flights

Use the `/find-flights` skill to search for flights:
- From the user's home airport to the destination
- For the specified dates and party size
- Apply flight preferences from profile
- Present top 3 options with the full flight output template

### Step 2: Accommodation

Use the `/find-accommodation` skill to search for places to stay:
- In the destination for the trip dates
- For the specified party size
- Apply accommodation preferences from profile
- Present top 3 options with the full accommodation output template

### Step 3: Activities & Itinerary

Build a day-by-day itinerary for the trip. Use web search to research:
- Top attractions and activities at the destination
- Local restaurants and food experiences
- Transport options within the destination
- Weather expectations for the travel dates
- Any local events during the trip dates

### Step 4: Practical Information

Research and include:
- Visa requirements (based on user's passport nationality from profile)
- Local currency and payment tips
- Airport transfer options to/from accommodation
- Emergency numbers and nearest hospital/embassy
- Local customs or etiquette tips
- Mobile connectivity (SIM/eSIM options)

## Output Format

Present the complete plan in this structure:

```
# Trip Plan: [Destination]
**[Start Date] — [End Date] | [N] travellers | [Context: solo/partner/group/work]**

---

## Flights
[Flight options from Step 1 — use the flight output template]

---

## Accommodation
[Accommodation options from Step 2 — use the accommodation output template]

---

## Day-by-Day Itinerary

### Day 1 — [Date] — Arrival
- **[Time]** Arrive at [airport] (Flight: [airline] [number])
- **[Time]** Transfer to accommodation ([method], approx [cost])
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
- **[Time]** Transfer to airport
- **[Time]** Depart (Flight: [airline] [number])

---

## Practical Info
- **Visa:** [Requirement based on passport]
- **Currency:** [Local currency] — [exchange/payment tips]
- **Getting around:** [Transport options]
- **Weather:** [Expected conditions for travel dates]
- **Emergency:** [Key numbers]

---

## Budget Summary
| Category | Estimated Cost |
|----------|---------------|
| Flights | [LIVE PRICE] per person |
| Accommodation | [LIVE PRICE] total for [N] nights |
| Activities | [Estimated range — clearly marked as estimate] |
| Food | [Estimated range — clearly marked as estimate] |
| Transport | [Estimated range — clearly marked as estimate] |
| **Total** | **[Sum]** |

> Live-verified prices marked with timestamps. Estimated costs marked with "est." — verify locally.
> All prices retrieved at [HH:MM today].
```

## Important Rules

1. **Flights and accommodation prices must be live-verified** following the guardrails skill
2. **Activity and food costs can be estimated** but must be clearly marked as estimates, not live prices
3. **Never make up flight times or accommodation details** — only use what the MCP tools return
4. **If the user said "entire trip planned out"** — include EVERYTHING: flights, accommodation, daily itinerary, practical info, and budget summary. Don't give a partial plan.
5. **Save the trip plan** to `${CLAUDE_PLUGIN_DATA}/trips/[destination]-[date].json` for future reference

## Saving Trip Data

After presenting the plan, save a summary to the trip history:

```json
{
  "destination": "Punta Secca, Sicily",
  "dates": { "start": "2025-05-15", "end": "2025-05-20" },
  "travellers": 4,
  "context": "group",
  "flights": { "airline": "...", "price_pp": "...", "verified_at": "..." },
  "accommodation": { "name": "...", "price_total": "...", "verified_at": "..." },
  "status": "planned",
  "created_at": "2025-04-02"
}
```
