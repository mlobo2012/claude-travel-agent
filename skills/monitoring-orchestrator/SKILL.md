---
name: monitoring-orchestrator
description: "Canonical monitoring architecture for the AI Heroes Travel Agent. Defines the local watch store, the single Claude Desktop scheduled dispatcher task, watch lifecycle rules, and actionable alert logic. Automatically active for price watches, re-shop monitoring, and trip reminders."
user-invocable: false
---

# Monitoring Orchestrator

This skill is the source of truth for recurring monitoring in the AI Heroes Travel Agent.

## Official-Docs Ground Rules

1. Robust recurring monitoring for this plugin uses **Claude Desktop local scheduled tasks**.
2. Scheduled task creation and management must happen through Claude's supported scheduling flows such as `/schedule` or the Scheduled sidebar. Do not assume a hidden plugin API for task creation.
3. Do not route plugin-driven monitoring to remote tasks. The documented remote runtime is Claude Code on the web, which runs in a GitHub-backed remote VM. Plugin-based monitoring is grounded here only in the local Cowork scheduled-task docs, which explicitly mention access to installed plugins.
4. Every monitoring claim made to the user must stay inside those product limits.

## Canonical Local State Store

Recurring monitoring state lives in the project so the local scheduled task and the plugin conversation can share it:

- `.claude/travel-monitor/watchlist.json`
- `.claude/travel-monitor/history.json`
- `.claude/travel-monitor/alerts.json`

If the directory or files do not exist, create them before saving the first watch.

### `watchlist.json`

This is the authoritative watch registry.

```json
{
  "version": "1.0",
  "dispatcher_task_name": "ai-heroes-travel-monitor",
  "updated_at": "2026-04-03T16:00:00Z",
  "watches": [
    {
      "id": "watch_ba548_20260515",
      "kind": "flight",
      "provider": "British Airways",
      "route_or_property": "LHR -> FCO 2026-05-15 / 2026-05-20",
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
      "threshold_value": 25.0,
      "fee_model": {
        "type": "fixed_total",
        "amount": 45.0,
        "notes": "BA cancellation/change friction used for actionable savings"
      },
      "cadence": "daily",
      "last_checked_at": null,
      "last_alerted_at": null,
      "expiry_at": "2026-05-15T06:30:00Z",
      "status": "active",
      "watch_reason": "pre_booking",
      "cooldown_hours": 24,
      "notes": "Alert only if the change is actionable after fees"
    }
  ]
}
```

### `history.json`

Append one record per completed check.

```json
{
  "version": "1.0",
  "checks": [
    {
      "watch_id": "watch_ba548_20260515",
      "checked_at": "2026-04-03T16:00:00Z",
      "observed_price": 298.0,
      "currency": "GBP",
      "observed_status": "available",
      "gross_delta": -37.0,
      "gross_delta_pct": -11.04,
      "fee_cost": 45.0,
      "net_savings": -8.0,
      "triggered_alert": false,
      "alert_reason": null,
      "source": "google-flights",
      "notes": "Price moved but did not clear the actionable threshold"
    }
  ]
}
```

### `alerts.json`

Track sent alerts so duplicate notifications are suppressed.

```json
{
  "version": "1.0",
  "alerts": [
    {
      "watch_id": "watch_ba548_20260515",
      "alerted_at": "2026-04-04T09:00:00Z",
      "alert_type": "net_savings_hit",
      "observed_price": 270.0,
      "net_savings": 20.0,
      "dedupe_key": "watch_ba548_20260515:net_savings_hit:270.0"
    }
  ]
}
```

## Single-Dispatcher Pattern

Use **one** local scheduled task named `ai-heroes-travel-monitor`.

Do not create one scheduled task per watch.

The dispatcher task should:

1. Open the travel project folder as its working folder.
2. Read `.claude/travel-monitor/watchlist.json`.
3. Select watches where:
   - `status` is `active`
   - `expiry_at` is in the future
   - the watch is due based on `cadence` and `last_checked_at`
4. Re-run the relevant live search for each due watch.
5. Append results to `history.json`.
6. Evaluate whether an alert is actionable.
7. If an alert is sent, append to `alerts.json` and update `last_alerted_at`.
8. Mark expired watches as `expired`.
9. Save `last_checked_at` for every processed watch.

## Recurring Monitor Task Prompt

When Claude needs to create or refresh the dispatcher task, use a prompt in this shape:

```text
Create or update a local scheduled task named ai-heroes-travel-monitor.

Working folder: this travel plugin project root.
Cadence: [user-selected cadence, usually daily or twice daily].

Instructions:
- Read .claude/travel-monitor/watchlist.json, .claude/travel-monitor/history.json, and .claude/travel-monitor/alerts.json.
- Process only watches with status=active that are due based on cadence and not past expiry_at.
- Re-run the live lookup for each watch using its saved search_params.
- Calculate actionable net savings after fee_model, not just raw price movement.
- Append one record per check to history.json.
- Suppress duplicate alerts using alerts.json and each watch's cooldown_hours.
- Update last_checked_at for every processed watch, last_alerted_at when an alert is sent, and status=expired once expiry_at has passed.
- If an item is unavailable, record that in history and only alert if the unavailability itself is actionable.
- Produce a concise report for this run.
```

Never claim the task has been created until Claude's scheduling UI confirms it.

## Watch Lifecycle

### Register

When the user asks to monitor something:

1. Normalize the watch into the canonical schema.
2. Save it to `watchlist.json`.
3. Reuse the dispatcher task if it already exists.
4. If the dispatcher does not exist yet, instruct Claude to create it through `/schedule` or the Scheduled page.

### Update

When the user changes threshold, cadence, dates, provider, or fee assumptions:

1. Find the existing watch by `id` or the clearest matching travel item.
2. Update only the changed fields.
3. Keep the same dispatcher task.
4. Save `updated_at` in the file-level metadata.

### Pause

Set `status` to `paused`. Do not delete history.

### Resume

Set `status` back to `active` and keep prior history and alerts.

### Expire

Set `status` to `expired` when `expiry_at` is in the past or when the watch is no longer actionable, for example once travel has started or the decision window has closed.

### Cancel

Set `status` to `cancelled` if the user explicitly stops monitoring.

## Actionable Alert Rules

Alert on decisions, not noise.

### Net-savings-first rules

Use `fee_model` to convert gross movement into actionable movement.

- `threshold_type: net_savings_at_least`
  Alert only when `observed_savings_after_fees >= threshold_value`.
- `threshold_type: price_below`
  Alert when the current all-in price is below the target price.
- `threshold_type: pct_drop`
  Alert when the percentage drop meets the threshold and the move is still actionable after fees.
- `threshold_type: reminder`
  Alert when the time window is reached, such as T-7, T-3, T-1, or booking-window expiry.

### Dedupe and cooldown

Default `cooldown_hours` is `24` unless the user asks for something else.

Suppress an alert when:

- the last alert for the same watch was within `cooldown_hours`
- the dedupe key already exists in `alerts.json`
- the new observed price is not materially different from the last alerted price

Materially different means:

- at least 3% additional movement, or
- at least 10 units of the watch currency additional net benefit, or
- a different alert type such as `availability_change` or `expiry_warning`

## Worktree Recommendation

Local scheduled tasks can be configured with an optional working folder, and plugin-shipped agents can use `isolation: "worktree"` in Claude Code. For this monitoring system, the safe recommendation is:

1. Prefer the dispatcher task **without worktree isolation**.
2. Set the task's working folder to the real project root so `.claude/travel-monitor/` is always reachable.
3. Only use worktree isolation if you have explicitly verified that `.claude/travel-monitor/` is available inside that worktree path and the task writes back to the shared state you intend.

If that visibility is uncertain, do not use worktree isolation for monitoring.

## User-Facing Honesty Rules

- Tell the user robust recurring monitoring depends on Claude Desktop local scheduled tasks.
- Tell the user those tasks run only while the desktop app is open and the machine is awake.
- Do not describe remote tasks as the monitoring runtime for this plugin.
- Do not imply the plugin itself can silently create schedules through an undocumented API.
