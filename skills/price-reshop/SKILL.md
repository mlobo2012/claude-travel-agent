---
name: price-reshop
description: "Post-booking price intelligence routed through the local watch store and the single Claude Desktop monitoring dispatcher. Monitors booked travel for actionable net savings after fees. Auto-activates when bookings are detected."
user-invocable: false
---

# Post-Booking Price Re-Shop

Monitor booked travel items after purchase by registering them as watches in the canonical local watch store.

Use `monitoring-orchestrator` for the actual recurring model. This skill is only responsible for creating the correct watch data for booked items.

## Distinction From `price-monitor`

- `price-monitor` is for items not yet booked.
- `price-reshop` is for booked items where a lower replacement price may justify rebooking or refunding.
- The alert decision must be based on **net savings after fees**, not raw movement.

## When This Skill Activates

- a booking is detected or confirmed
- the user asks to watch a booked flight, hotel, or train for price drops
- the user wants rebooking guidance if a fare improves

## Canonical Watch Shape For Re-Shop

Save booked-item monitoring in `.claude/travel-monitor/watchlist.json` using the shared schema.

```json
{
  "id": "reshop_ba548_20260515",
  "kind": "flight",
  "provider": "British Airways",
  "route_or_property": "BA548 LHR -> FCO 2026-05-15 / 2026-05-20",
  "search_params": {
    "origin": "LHR",
    "destination": "FCO",
    "departure_date": "2026-05-15",
    "return_date": "2026-05-20",
    "adults": 1,
    "currency": "GBP"
  },
  "baseline_price": 335.0,
  "currency": "GBP",
  "threshold_type": "net_savings_at_least",
  "threshold_value": 20.0,
  "fee_model": {
    "type": "fixed_total",
    "amount": 45.0,
    "notes": "Only alert if the lower fare beats fees and friction"
  },
  "cadence": "daily",
  "last_checked_at": null,
  "last_alerted_at": null,
  "expiry_at": "2026-05-15T06:30:00Z",
  "status": "active",
  "watch_reason": "post_booking"
}
```

## Operating Rules

1. Save the booked item as a watch in `watchlist.json`.
2. Reuse the single dispatcher task `ai-heroes-travel-monitor`.
3. Append each recurring check to `history.json`.
4. Record sent alerts in `alerts.json`.
5. Expire the watch at departure, check-in, or when the rebooking window is no longer actionable.

## User-Facing Rule

Say that post-booking monitoring becomes robust when Claude Desktop local scheduled tasks are enabled. Do not imply remote tasks or undocumented plugin APIs are doing this work.
