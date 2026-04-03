---
name: trip-reminders
description: "Dynamic pre-trip reminders relative to departure date. Creates scheduled tasks for T-14, T-7, T-3, and T-1 day reminders with weather, flight details, packing suggestions, and trip status. Use when: a trip is booked or the user says 'remind me before my trip'."
user-invocable: false
---

# Pre-Trip Reminders

Automatically create scheduled reminder tasks relative to the user's departure date. Reminders are sent via Dispatch and adapt based on trip length and destination.

## When to Create Reminders

- When a trip is confirmed as "booked" (in `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` or Claude's memory)
- When the user says "remind me before my trip" or "set up trip reminders"
- When a booking is detected via Gmail (booking-detection skill)

## Reminder Schedule

### For all trips

**T-7 days (One week before departure):**

Create a scheduled task for 9:00 AM, 7 days before departure.

Prompt: "Send the user a 1-week pre-trip reminder. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` to load trip details for [destination] (fall back to Claude's memory for `travel_past_trips` if file not found). Include: trip dates, what's booked vs still needed (flights, accommodation), any action items. Use web search to get the current weather forecast for [destination] for the travel dates. Send via Dispatch."

Template:
```
One Week to Go! Your [destination] trip is in 7 days.

Status:
- Flights: [Booked: airline, times / NOT BOOKED — book now!]
- Accommodation: [Booked: property name / NOT BOOKED — book now!]
- Activities: [Planned / Not yet planned]

Weather forecast for [destination] ([dates]):
[Weather summary from web search]

Anything still to sort? Just ask!
```

**T-1 day (Day before departure):**

Create a scheduled task for 8:00 AM, 1 day before departure.

Prompt: "Send the user a departure-eve reminder. Include flight details, weather at destination, and packing reminders based on destination and weather. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` and `${CLAUDE_PLUGIN_DATA}/travel_packing_preferences.json` to load trip details and packing preferences (fall back to Claude's memory if files not found). Send via Dispatch."

Template:
```
Flying Tomorrow!

[Airline] [Flight number/route] from [airport]
Departs: [time] — arrive at airport by [time minus 2hr domestic / 3hr international]
Terminal: [if known]

Weather in [destination]: [current forecast]
[Temperature range] — [clothing suggestion based on weather]

Don't forget:
- Passport (expires: [from profile])
- [Phone charger, travel adaptor if destination needs one]
- [Any items from user's packing preferences in memory]

Have an amazing trip!
— AI Heroes Travel Agent
```

### For trips 7+ days (extended trips)

**T-14 days (Two weeks before):**

Create a scheduled task for 9:00 AM, 14 days before departure.

Template:
```
Two Weeks to [destination]!

Quick checklist:
- Travel insurance sorted?
- Checked luggage booked with your airline?
- Any visa requirements? [check based on passport nationality]
- Currency: [destination currency] — consider ordering some or checking your travel card

Your trip: [dates], [travellers] travellers
```

### For trips 3 days or less (weekend trips)

Skip the T-14 and T-7 reminders. Instead:

**T-3 days (Combined planning + packing):**

Template:
```
Weekend Trip to [destination] in 3 days!

Your quick-trip checklist:
- [Flight details if booked]
- [Accommodation details if booked]
- Pack light: [weather-appropriate suggestions]
- [Destination-specific tips — e.g., "bring cash, small shops don't take card"]

Anything you still need to arrange?
```

**T-1 day:** Same as standard T-1 template above.

## Reminder Content Requirements

All reminders must include:
1. **Current weather forecast** for the destination (use web search at the time the reminder fires)
2. **Flight details** if booked (from `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` via the Read tool, or Claude's memory)
3. **Accommodation details** if booked (from `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` via the Read tool, or Claude's memory)
4. **Packing suggestions** appropriate to weather and destination
5. **Destination-specific items** (travel adaptor type, visa notes, currency tips)

## Packing Intelligence

Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel_packing_preferences.json` to load packing preferences (fall back to Claude's memory for `travel_packing_preferences` if file not found). If the user has saved packing preferences, include their must-haves. If not, use sensible defaults based on:

- **Beach destination:** sunscreen, swimwear, hat, sunglasses
- **City break:** comfortable walking shoes, umbrella, day bag
- **Cold destination:** layers, warm coat, thermals
- **International (non-UK):** passport, travel adaptor (type based on country), travel insurance docs
- **Domestic UK:** no passport needed, no adaptor needed

## Creating the Scheduled Tasks

When setting up reminders for a trip:

1. Calculate the reminder dates based on departure date and trip length
2. Create one scheduled task per reminder, each with the appropriate cron expression for a single fire
3. Use the **Write tool** to save the task IDs to `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` alongside the trip data AND save to Claude's memory
4. Confirm to the user: "I've set up [N] reminders for your [destination] trip. You'll get alerts at [list dates/times]."

## Cancellation

If a trip is cancelled:
1. Delete all associated reminder scheduled tasks
2. Use the **Write tool** to update the trip status to "cancelled" in `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` AND update Claude's memory
3. Confirm: "Cancelled all reminders for your [destination] trip."
