---
description: Show all AI Heroes Travel Agent commands and how to get started
---

# AI Heroes Travel Agent v2.2 — Quick Reference

Here are the commands available to you:

## Getting Started
- **/onboarding** — Set up your travel preferences (including transport mode, children's ages, and loyalty programmes)
- **/memory-manager** — View or update your saved travel preferences
- **/memory-manager** update — Change a specific preference
- **/memory-manager** history — See your past trips

## Searching
- **/flight-search** [origin] to [destination] [dates] — Search for flights with live-verified prices
- **/train-search** [origin] to [destination] [dates] — Search for trains with child fare policies
- **/ferry-search** [origin] to [destination] [dates] — Search for ferries with cabin options
- **/accommodation-search** [destination] [dates] [guests] — Search Airbnb and Booking.com

## Planning
- **/trip-planner** [destination] [dates] [party size] — Full end-to-end trip planning with parallel agent search (transport + accommodation + local transit + itinerary)

## Price Monitoring & Re-Shopping
- **/price-monitor** — Set up price watches on flights or accommodation (pre-booking)
- Say **"watch this for me"** or **"monitor the price"** during any search to start tracking
- Say **"what are you watching?"** to see active price watches
- Say **"stop watching"** to cancel a price watch
- **Post-booking price re-shopping** — Automatic! After you book, I monitor for price drops and alert you with savings, cancellation guidance, and rebooking links

## Loyalty Programmes
- **/loyalty-manager** — View all your loyalty programmes and status
- **/loyalty-manager** add — Add a new loyalty programme
- **/loyalty-manager** update — Update points balance or tier
- **/loyalty-manager** status — Check tier qualification progress and upcoming expirations
- Loyalty programmes are auto-applied to every search — I'll flag earning opportunities and tier implications

## Document Scanning & Expenses
- **Upload a photo** of a boarding pass, receipt, booking confirmation, or hotel bill — I'll extract all the details automatically
- **/travel-expenses** — Generate an expense report for your most recent trip
- **/travel-expenses** [trip name] — Generate an expense report for a specific trip
- **/travel-expenses** --daily — Include per-day breakdown
- **/travel-expenses** --vat — Include VAT/tax summary

## Trip Pack
- **/trip-pack** — Generate your pre-departure trip pack (all trip details in one document)
- Auto-generates 2 days before departure if a trip is booked

## After Your Trip
- **/feedback** — Record how the trip went to improve future recommendations

## How It Works
- **Proactive transport intelligence** — I analyse your route and recommend the best mode (train, flight, ferry) before you ask
- **Reasoning transparency** — Every recommendation includes a **Why this?** block explaining exactly why I chose it, based on your profile, past trips, and preferences
- **Loyalty intelligence** — I auto-apply your loyalty programmes, track tier progress, and flag earning opportunities
- **Post-booking monitoring** — After you book, I watch for price drops and alert you with net savings after cancellation fees
- **Document scanning** — Upload receipts, boarding passes, and hotel bills to track expenses and update trip records
- All prices are retrieved live from MCP servers (Airbnb, Google Flights, Public Transport, TfL, Ferryhopper)
- Prices are cross-checked across sources when possible
- Every price is timestamped — "Price as of [time] — verify at checkout"
- Results are ranked using your personal preference profile (stored in persistent memory)
- I never guess, estimate, or fabricate prices
- Trip planning uses 3 specialist agents in parallel for faster results
- **Family-aware** — I know child fare policies for major train operators and airlines

## v2.2 Features (New!)
- **Post-Booking Price Re-Shopping** — Automatic monitoring after you book. Alerts with savings, cancellation fees, net benefit, and rebooking links. Tracks price trends over time.
- **Reasoning Transparency** — Every recommendation includes a visible "Why this?" block tracing the reasoning to your profile, past feedback, loyalty status, and trip context. Challenge any recommendation and I'll explain what was filtered and why.
- **Loyalty Programme Intelligence** — Full loyalty management: auto-apply to searches, track tier progress, cross-programme optimisation (Marriott vs Hilton, BA vs budget airline). Covers airlines, hotels, and rail programmes.
- **Document Scanner** — Upload boarding passes, receipts, booking confirmations, and hotel bills. I extract all details, log expenses, flag overcharges, and update trip records. Includes budget tracking and spending pattern analysis.
- **Expense Reports** — `/travel-expenses` generates formatted reports grouped by category with totals, VAT breakdown, and comparison to your historical averages.
- **Cross-Feature Intelligence** — All new features integrate deeply: loyalty impacts price reshop decisions, document scanning feeds the loyalty tracker, expense patterns inform trip planning budgets, and reasoning transparency explains it all.

### Previously in v2.1
- **Multi-modal transport** — Trains, ferries, and local transit alongside flights
- **Proactive intelligence** — Recommends the best transport mode based on your profile
- **Family-aware** — Knows child fare policies for major train and ferry operators
- **Local transit** — Last-mile connections from stations/airports to your accommodation
- **Persistent Memory** — Your preferences persist across all sessions automatically
- **Parallel Agent Search** — Flight, accommodation, and activities agents search simultaneously
- **Price Monitoring** — Watch prices twice daily, get alerts on drops >10%
- **Pre-Trip Reminders** — Automatic reminders at T-14, T-7, T-3, and T-1 days
- **Trip Pack** — Branded pre-departure document with everything you need
- **Gmail Detection** — Auto-detect booking confirmations from your email (requires Gmail connector)
- **Calendar Integration** — Auto-create calendar events for flights, accommodation, and activities (requires Google Calendar connector)

## Optional Connectors
- **Gmail** — Enable read-only access to auto-detect booking confirmations and auto-scan them
- **Google Calendar** — Enable to auto-create trip events, flight reminders, and packing alerts

## Tips
- Run **/onboarding** first to get personalised results (including loyalty programmes!)
- Tell me your children's ages — it affects train fare calculations
- Say "direct flights only" to filter out connections
- Say "small budget" to focus on value options
- Say "watch this" after a search to monitor price changes
- Upload a photo of any travel document — I'll extract it automatically
- Say "why did you suggest this?" to see the full reasoning behind any recommendation
- Say "why not [X]?" to understand what filtered an option out
- I'll compare transport modes when multiple are competitive
- I'll always give you 3 options ranked by how well they match your preferences, with reasoning for each

Built by [AI Heroes](https://www.ai-heroes.co)
