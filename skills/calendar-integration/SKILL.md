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

## Critical Rule For Flights And Trains

When creating a calendar event for any flight or train departure:

1. Read the user's home location from their travel profile first.
2. If TfL MCP is available and the journey starts in London, use it to get a current journey estimate.
3. Otherwise, use these default travel times:
   - London Heathrow (LHR): 1h 30min from central or west London
   - London Gatwick (LGW): 1h from central London
   - London Stansted (STN): 1h 15min from central London
   - London Luton (LTN): 1h from central London
   - London City (LCY): 45min from central London
   - St Pancras International: 30 to 45min from west London
4. Add a check-in buffer before the departure event:
   - Domestic flights: 2h
   - International flights: 3h
   - Eurostar: 30min
5. Create two separate calendar events:
   - `Travel to [Airport/Station]`, from home departure time to terminal or station arrival time
   - `Flight/Train: [Route]`, from scheduled departure to scheduled arrival, including layovers where relevant

## When to Create Events

- When a flight booking is confirmed (via Gmail detection or manual confirmation)
- When a train booking is confirmed
- When an accommodation booking is confirmed
- When activities are planned as part of a trip
- When the user says "add to my calendar" or "put this in my calendar"

## Event Templates

### Travel To Terminal Events

```text
Title: Travel to [Airport/Station]
Start: [Home departure time]
End: [Terminal or station arrival time]
Location: [Airport or station name]
Description:
  Leaving from: [home area if known]
  Travel time basis: [TfL live estimate or default rule]
  Buffer before departure event: [buffer applied]
```

### Flight Or Train Events

```text
Title: Flight/Train: [ORIGIN] -> [DESTINATION]
Start: [Scheduled departure time]
End: [Scheduled arrival time]
Location: [Departure terminal or station]
Description:
  Operator: [Airline or rail operator]
  Service: [Flight number or train number]
  Booking ref: [Reference]
  Terminal/platform: [If known]
  Includes layovers: [yes/no, details if known]
```

### Accommodation Events

```text
Title: Check in - [Property name]
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

```text
Title: Check out - [Property name]
Start: [Check-out time minus 1 hour, or 09:00 if not specified]
End: [Check-out time, or 10:00 if not specified]
Location: [Full address]
Description:
  Check out by [time]
  [Any check-out instructions]
```

### Trip Block

```text
Title: [Destination] Trip
Start: [First day, all-day event]
End: [Last day, all-day event]
Description:
  [Destination] trip, [N] travellers
  Flights/Trains: [Operator] [Booking ref]
  Accommodation: [Property name] [Booking ref]
```

### Packing Reminder

```text
Title: Pack for [Destination]
Start: [Departure date minus 1 day, 19:00]
End: [Departure date minus 1 day, 20:00]
Description:
  Your [destination] trip is tomorrow.

  Packing checklist:
  - Passport
  - [Weather-appropriate items]
  - [Travel adaptor if needed]
  - [Items from packing preferences]
```

### Activity Events

```text
Title: [Activity name]
Start: [Activity time]
End: [Activity end time, or start + 2 hours]
Location: [Venue/address if known]
Description:
  [Activity details]
  [Booking ref if applicable]
  [Cost if known]
```

## Event Creation Flow

When a trip is fully booked:

1. Create the all-day trip block event.
2. Create the travel-to-terminal event for the outbound flight or train.
3. Create the outbound flight or train event.
4. Create the accommodation check-in event.
5. Create any planned activity events.
6. Create the accommodation check-out event.
7. Create the travel-to-terminal event for the return leg if relevant.
8. Create the return flight or train event.
9. Create the packing reminder.
10. Confirm to the user what was created.

## Manual Trigger

When the user says "add to my calendar":

1. Load trip data from `${CLAUDE_PLUGIN_DATA}/travel_past_trips.json` if available.
2. If needed, fall back to Claude memory for the trip details.
3. If the calendar connector is missing, explain that Google Calendar access is required.
4. If the connector is available, create all applicable events.
5. List what was created, including the separate travel-to-terminal blocks.

## Updating Events

If trip details change:

1. Find and update the relevant calendar events.
2. Notify the user what changed.

## Cancellation

If a trip is cancelled:

1. Remove all associated calendar events.
2. Confirm that the calendar was cleaned up.
