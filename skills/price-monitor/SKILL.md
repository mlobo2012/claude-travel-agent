---
name: price-monitor
description: "Price monitoring for flights, hotels, and trains using the local watch store plus one Claude Desktop scheduled dispatcher task. Use when: user says 'watch this', 'monitor the price', 'alert me if price drops', or expresses interest without booking."
argument-hint: "[flight or listing reference]"
---

# Price Monitor

Track a candidate before booking, save it to the canonical watch store, and route recurring checks through the monitoring-orchestrator skill.

## Use The Canonical Monitoring Model

Always follow `monitoring-orchestrator` for:

- the local state files in `.claude/travel-monitor/`
- the single dispatcher task `ai-heroes-travel-monitor`
- scheduled-task creation via `/schedule` or the Scheduled sidebar
- actionable alert logic based on net savings or explicit thresholds
- duplicate-alert suppression through `alerts.json`

Never create one scheduled task per watched item.

## When to Activate

- User says "watch this for me", "monitor the price", "let me know if it drops"
- User says "I'm interested but not ready to book"
- User says "alert me if the price changes"
- User explicitly runs `/price-monitor`

## Registering A Watch

### Step 1: Capture The Baseline

Save the travel item in canonical watch form. Use `kind` values `flight`, `hotel`, or `train`.

Required fields:

- `id`
- `kind`
- `provider`
- `route_or_property`
- `search_params`
- `baseline_price`
- `currency`
- `threshold_type`
- `threshold_value`
- `fee_model`
- `cadence`
- `last_checked_at`
- `last_alerted_at`
- `expiry_at`
- `status`

Example:

```json
{
  "id": "watch_easyjet_lgw_cta_20260515",
  "kind": "flight",
  "provider": "easyJet",
  "route_or_property": "LGW -> CTA 2026-05-15 / 2026-05-20",
  "search_params": {
    "origin": "LGW",
    "destination": "CTA",
    "departure_date": "2026-05-15",
    "return_date": "2026-05-20",
    "adults": 4,
    "non_stop": true,
    "currency": "GBP"
  },
  "baseline_price": 509.72,
  "currency": "GBP",
  "threshold_type": "price_below",
  "threshold_value": 480.0,
  "fee_model": {
    "type": "none",
    "amount": 0
  },
  "cadence": "daily",
  "last_checked_at": null,
  "last_alerted_at": null,
  "expiry_at": "2026-05-15T06:30:00Z",
  "status": "active"
}
```

### Step 2: Save It To The Local Watch Store

Write the watch into `.claude/travel-monitor/watchlist.json`.

Also ensure `.claude/travel-monitor/history.json` and `.claude/travel-monitor/alerts.json` exist so the dispatcher has somewhere to append results.

### Step 3: Reuse Or Create The Dispatcher

If `ai-heroes-travel-monitor` already exists, do not create another task.

If it does not exist and the user wants robust recurring monitoring in Claude Desktop:

- instruct Claude to create it through `/schedule` or the Scheduled page
- set the task working folder to the project root
- keep all watches behind that one dispatcher

If Claude Desktop scheduling is not available in the current environment:

- still save the watch locally
- tell the user the recurring runtime is not active yet
- offer to re-check manually or help them create the local scheduled task later

### Step 4: Confirm To The User

If scheduling is confirmed:

```text
Watching this for you. I've saved it to the local watch store and routed it through the ai-heroes-travel-monitor dispatcher task.

Tracked item:
- [Item description], baseline: [price] as of [date/time]

You will get an alert when the threshold is actually hit under the saved rules.
```

If scheduling is not confirmed:

```text
I have saved this to the local watch store with the current baseline.

Tracked item:
- [Item description], baseline: [price] as of [date/time]

Robust recurring monitoring requires a Claude Desktop local scheduled task. I have not assumed a hidden background runtime.
```

## Alert Logic

Alert on actionable movement, not every raw change.

- `price_below`: current price is at or below the target
- `pct_drop`: percentage drop meets threshold and is still meaningful
- `net_savings_at_least`: gross movement minus fees meets the threshold

Use `alerts.json` to suppress duplicates and apply cooldowns.

## Pause, Resume, Cancel

When the user changes monitoring state, update the matching watch in `watchlist.json`:

- `paused`
- `active`
- `cancelled`
- `expired`
