---
name: trip-reminders
description: "Pre-trip reminders routed through the local watch store and Claude Desktop local scheduled tasks. Use when: a trip is booked or the user says 'remind me before my trip'."
user-invocable: false
---

# Pre-Trip Reminders

Pre-trip reminders use the same monitoring architecture as price watches.

## Canonical Model

- save reminder watches in `.claude/travel-monitor/watchlist.json`
- route recurring delivery through the single task `ai-heroes-travel-monitor`
- create or manage that task only through `/schedule` or the Scheduled page
- do not promise reminders are active unless Claude Desktop local scheduling is actually enabled

## When To Prepare Reminders

- When a trip is confirmed as booked
- When the user says "remind me before my trip" or "set up trip reminders"
- When a booking is detected via Gmail

## Reminder Watch Shape

Represent each reminder as a watch with `threshold_type: reminder`.

```json
{
  "id": "reminder_rome_trip_tminus3",
  "kind": "flight",
  "provider": "trip-reminders",
  "route_or_property": "Rome trip 2026-05-15",
  "search_params": {
    "trip_id": "rome_2026_05_15",
    "reminder_type": "t_minus_3"
  },
  "baseline_price": 0,
  "currency": "GBP",
  "threshold_type": "reminder",
  "threshold_value": "t_minus_3",
  "fee_model": {
    "type": "none",
    "amount": 0
  },
  "cadence": "daily",
  "last_checked_at": null,
  "last_alerted_at": null,
  "expiry_at": "2026-05-15T00:00:00Z",
  "status": "active",
  "watch_reason": "trip_reminder"
}
```

## Reminder Cadence

- T-14 for longer trips
- T-7 for standard trips
- T-3 for short trips or specific ticket-printing prompts
- T-1 for departure-eve reminders

## Reminder Content Requirements

All reminders should include, where relevant:

1. Current weather for the destination
2. Flight or train details if booked
3. Accommodation details if booked
4. Packing guidance based on trip type and weather
5. Destination-specific notes such as adapter, currency, or visa reminders

## Scheduling Rules

- If Claude Desktop local scheduling is enabled, register the reminder watches and route them through `ai-heroes-travel-monitor`.
- If scheduling is not enabled, still save the reminder watches locally and tell the user recurring delivery is not active yet.
- Remote tasks are not the runtime for plugin-driven reminders.
