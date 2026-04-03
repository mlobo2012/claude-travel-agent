---
description: Show all AI Heroes Travel Agent commands and how to get started
---

# AI Heroes Travel Agent v2.0 — Quick Reference

Here are the commands available to you:

## Getting Started
- **/onboarding** — Set up your travel preferences (first-time onboarding, 10 questions)
- **/memory-manager** — View or update your saved travel preferences
- **/memory-manager** update — Change a specific preference
- **/memory-manager** history — See your past trips

## Searching
- **/flight-search** [origin] to [destination] [dates] — Search for flights with live-verified prices
- **/accommodation-search** [destination] [dates] [guests] — Search Airbnb and Booking.com

## Planning
- **/trip-planner** [destination] [dates] [party size] — Full end-to-end trip planning with parallel agent search (flights + accommodation + activities simultaneously)

## Price Monitoring
- **/price-monitor** — Set up price watches on flights or accommodation
- Say **"watch this for me"** or **"monitor the price"** during any search to start tracking
- Say **"what are you watching?"** to see active price watches
- Say **"stop watching"** to cancel a price watch

## Trip Pack
- **/trip-pack** — Generate your pre-departure trip pack (all trip details in one document)
- Auto-generates 2 days before departure if a trip is booked

## After Your Trip
- **/feedback** — Record how the trip went to improve future recommendations

## How It Works
- All prices are retrieved live from Airbnb and Google Flights MCP servers
- Prices are cross-checked across sources when possible
- Every price is timestamped — "Price as of [time] — verify at checkout"
- Results are ranked using your personal preference profile (stored in persistent memory)
- I never guess, estimate, or fabricate prices
- Trip planning uses 3 specialist agents in parallel for faster results

## V2.0 Features
- **Persistent Memory** — Your preferences persist across all sessions automatically
- **Parallel Agent Search** — Flight, accommodation, and activities agents search simultaneously
- **Price Monitoring** — Watch prices twice daily, get alerts on drops >10%
- **Pre-Trip Reminders** — Automatic reminders at T-14, T-7, T-3, and T-1 days
- **Trip Pack** — Branded pre-departure document with everything you need
- **Gmail Detection** — Auto-detect booking confirmations from your email (requires Gmail connector)
- **Calendar Integration** — Auto-create calendar events for flights, accommodation, and activities (requires Google Calendar connector)

## Optional Connectors
- **Gmail** — Enable read-only access to auto-detect booking confirmations
- **Google Calendar** — Enable to auto-create trip events, flight reminders, and packing alerts

## Tips
- Run **/onboarding** first to get personalised results
- Say "direct flights only" to filter out connections
- Say "small budget" to focus on value options
- Say "watch this" after a search to monitor price changes
- I'll always give you 3 options ranked by how well they match your preferences

Built by [AI Heroes](https://www.ai-heroes.co)
