---
name: calendar-integration
description: "Create Google Calendar events for flights, accommodation, and trip activities. Adds airport buffer times, packing reminders, and blocks trip dates. Use when: a booking is confirmed or the user says 'add to my calendar'."
user-invocable: false
---

# Google Calendar Integration

Automatically create calendar events for all trip components when bookings are confirmed.

## Prerequisites

- Google Calendar connector must be enabled (read-write access)
- User must have granted Calendar permissions when prompted by Cowork

## When to Create Events

- When a flight booking is confirmed (via Gmail detection or manual confirmation)
- When an accommodation booking is confirmed
- When activities are planned as part of a trip
- When the user says "add to my calendar" or "put this in my calendar"

## Event Templates

### Flight Events

**Outbound flight:**
```
Title: [plane emoji] [ORIGIN] -> [DESTINATION] — [Trip Name]
Start: [Departure time minus buffer]
End: [Arrival time]
Location: [Airport name], [Terminal if known]
Description:
  Flight: [Airline] [Flight number]
  Departs: [Time] from [Airport] [Terminal]
  Arrives: [Time] at [Destination airport]
  Booking ref: [Reference]

  Arrive at airport by [buffer time]
  Check-in opens [time if known]
```

Buffer times:
- **Domestic UK flights:** 2 hours before departure
- **European flights:** 2.5 hours before departure
- **International (long-haul):** 3 hours before departure

**Return flight:** Same format, reversed airports.

### Accommodation Events

**Check-in event:**
```
Title: [key emoji] Check in — [Property name]
Start: [Check-in time, or 15:00 if not specified]
End: [Check-in time + 1 hour]
Location: [Full address]
Description:
  Property: [Property name]
  Address: [Full address]
  Booking ref: [Reference]
  Host: [Name/contact if available]

  [Any check-in instructions from booking]
```

**Check-out event:**
```
Title: [key emoji] Check out — [Property name]
Start: [Check-out time minus 1 hour, or 09:00 if not specified]
End: [Check-out time, or 10:00 if not specified]
Location: [Full address]
Description:
  Check out by [time]
  [Any check-out instructions]
```

### Trip Block (All-day events)

```
Title: [palm tree emoji] [Destination] Trip
Start: [First day, all-day event]
End: [Last day, all-day event]
Description:
  [Destination] trip — [N] travellers
  Flights: [Airline] [Booking ref]
  Accommodation: [Property name] [Booking ref]
```

### Packing Reminder

```
Title: [luggage emoji] Pack for [Destination]
Start: [Departure date minus 1 day, 19:00]
End: [Departure date minus 1 day, 20:00]
Description:
  Your [destination] trip is tomorrow!

  Packing checklist:
  - Passport
  - [Weather-appropriate items]
  - [Travel adaptor if needed]
  - [Items from packing preferences]

  Don't forget to check in online if you haven't already.
```

### Activity Events

When activities are planned:
```
Title: [activity emoji] [Activity name]
Start: [Activity time]
End: [Activity end time, or start + 2 hours]
Location: [Venue/address if known]
Description:
  [Activity details]
  [Booking ref if applicable]
  [Cost if known]
```

Activity emoji mapping:
- Restaurant/food: fork and knife emoji
- Tour/sightseeing: camera emoji
- Beach: beach emoji
- Hiking: boot emoji
- Museum/culture: building emoji
- Boat/water: boat emoji
- Default: pin emoji

## Event Creation Flow

When a trip is fully booked (flights + accommodation):

1. Create the all-day trip block event
2. Create outbound flight event (with buffer)
3. Create accommodation check-in event
4. Create any planned activity events
5. Create accommodation check-out event
6. Create return flight event (with buffer)
7. Create packing reminder (T-1 day, 7 PM)
8. Confirm to user: "Added [N] events to your Google Calendar for your [destination] trip."

## Manual Trigger

When the user says "add to my calendar":
1. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` to load trip data. If the file doesn't exist, fall back to Claude's memory for trip details.
2. If no calendar connector, explain: "I need Google Calendar access to create events. You can enable the Google Calendar connector in your Cowork settings."
3. If connector available, create all applicable events
4. List what was created

## Updating Events

If trip details change (new flight, different accommodation):
1. Find and update the relevant calendar events
2. Notify the user: "Updated your calendar — [what changed]"

## Cancellation

If a trip is cancelled:
1. Remove all associated calendar events
2. Confirm: "Removed all calendar events for your [destination] trip."
