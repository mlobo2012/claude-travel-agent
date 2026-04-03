---
name: booking-detection
description: "Detect and parse booking confirmation emails from Gmail. Extracts booking references, dates, flight numbers, and accommodation details. Logs bookings to persistent memory and confirms via Dispatch. Use when: Gmail connector is enabled and booking emails are detected."
user-invocable: false
---

# Booking Confirmation Detection

Monitor Gmail for booking confirmation emails and automatically log them to the travel profile.

## Prerequisites

- Gmail connector must be enabled (read-only access)
- User must have granted Gmail permissions when prompted by Cowork

## Detection Triggers

Monitor for emails from these senders (check periodically or when the user mentions a booking):

### Airlines
- `noreply@ryanair.com`, `*@ryanair.com`
- `noreply@easyjet.com`, `*@easyjet.com`
- `noreply@britishairways.com`, `*@britishairways.com`
- `*@ba.com`
- `noreply@wizzair.com`
- `*@jet2.com`
- `*@tui.com`, `*@tuifly.com`
- `*@vueling.com`
- `*@emirates.com`
- `*@qatarairways.com`
- Other airline confirmation patterns: subject contains "booking confirmation", "itinerary", "e-ticket"

### Accommodation Platforms
- `*@airbnb.com`, `noreply@airbnb.com`
- `*@booking.com`, `noreply@booking.com`
- `*@vrbo.com`
- `*@hotels.com`
- `*@expedia.com`

### General Patterns
- Subject line contains: "booking confirmed", "reservation confirmed", "your booking", "your reservation", "e-ticket", "itinerary receipt"
- Look for booking reference patterns: alphanumeric codes 6-10 characters

## Extraction Rules

When a booking confirmation email is found, extract:

### From Flight Confirmations
- **Booking reference** (e.g., "ABC123", "RYRREF")
- **Flight number** (e.g., "FR1234", "U2 8765")
- **Route** — origin and destination airports
- **Departure date and time**
- **Return date and time** (if round trip)
- **Passenger names**
- **Total price paid**
- **Airline name**

### From Accommodation Confirmations
- **Booking reference / confirmation number**
- **Property name**
- **Full address**
- **Check-in date and time**
- **Check-out date and time**
- **Number of guests**
- **Total price paid**
- **Host name / contact** (if available)
- **Cancellation policy summary**

## Processing

After extracting booking details:

### Step 1: Match to Existing Trip

1. Recall `travel_past_trips` from persistent memory
2. Check if the booking dates match any existing trip (within +/- 1 day tolerance)
3. If match found → update that trip with the booking details
4. If no match → create a new trip entry

### Step 2: Save to Persistent Memory

Update the matched or new trip in `travel_past_trips`:

```json
{
  "destination": "Punta Secca, Sicily",
  "dates": { "start": "2026-05-15", "end": "2026-05-20" },
  "travellers": 4,
  "status": "booked",
  "flights": {
    "outbound": {
      "airline": "easyJet",
      "flight_number": "U2 8765",
      "route": "LGW → CTA",
      "departure": "2026-05-15T06:30:00",
      "arrival": "2026-05-15T10:45:00",
      "booking_ref": "EZY123456"
    },
    "return": {
      "airline": "easyJet",
      "flight_number": "U2 8766",
      "route": "CTA → LGW",
      "departure": "2026-05-20T11:15:00",
      "arrival": "2026-05-20T13:30:00",
      "booking_ref": "EZY123456"
    },
    "price_total": 509.72,
    "currency": "GBP"
  },
  "accommodation": {
    "name": "Beachfront Villa in Punta Secca",
    "address": "Via Example 42, Punta Secca, RG, Sicily",
    "check_in": "2026-05-15T15:00:00",
    "check_out": "2026-05-20T10:00:00",
    "booking_ref": "HM12345678",
    "platform": "airbnb",
    "price_total": 425.00,
    "currency": "GBP"
  },
  "created_at": "2026-04-03"
}
```

### Step 3: Trigger Dependent Actions

After logging a booking:
1. **Create pre-trip reminders** (invoke trip-reminders skill logic)
2. **Create calendar events** (invoke calendar-integration skill logic, if connector enabled)
3. **Cancel any active price watches** for this item (invoke price-monitor cancellation)
4. **Schedule trip pack generation** at T-2 days

### Step 4: Confirm via Dispatch

Send confirmation to the user:

```
Booking Confirmed! [flight/accommodation emoji]

[Booking type]: [Details]
Dates: [Start] to [End]
Booking ref: [Reference]
Price: [Amount]

Added to your [destination] trip.
[If new trip created: "I've created a new trip entry for this."]
[If matched existing: "Updated your existing [destination] trip."]

I'll send you reminders before departure. Your trip pack will be ready 2 days before you fly.
```

## Privacy & Security

- Gmail access is **read-only** — this skill never sends, deletes, or modifies emails
- Only booking confirmation emails are read — personal emails are never accessed
- Extracted data is stored only in persistent memory and the local plugin data directory
- No email content is sent to third parties
