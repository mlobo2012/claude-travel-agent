---
name: price-monitor
description: "Price monitoring for flights and accommodation. Records watchlists, prepares Dispatch alerts, and supports Cowork scheduled-task workflows. Use when: user says 'watch this', 'monitor the price', 'alert me if price drops', or expresses interest without booking."
argument-hint: "[flight or listing reference]"
---

# Price Monitor

Track flights and accommodation, store the baseline, and prepare actionable price-change alerts.

## Current Reality

Be explicit about the current platform state.

- Claude Cowork has native scheduled tasks and Dispatch.
- Anthropic's public help docs describe those tasks as being created via `/schedule` or the Scheduled sidebar.
- Claude Code plugin skills do not create a system cron job or background daemon by themselves.
- This plugin can define the monitoring workflow, persist the watched item, and provide the prompt or structure for a recurring check.
- If scheduled execution is not available in the current environment, tell the user plainly that they may need to re-prompt later or set up Cowork scheduling manually.

Do not claim that a real twice-daily background task now exists unless the current Claude product confirms it.

## When to Activate

- User says "watch this for me", "monitor the price", "let me know if it drops"
- User says "I'm interested but not ready to book"
- User says "alert me if the price changes"
- User explicitly runs `/price-monitor`

## Setting Up A Price Watch

### Step 1: Capture Baseline

Record the current state of the item being watched.

**For flights:**
```json
{
  "type": "flight",
  "airline": "easyJet",
  "route": "LGW → CTA",
  "departure_date": "2026-05-15",
  "return_date": "2026-05-20",
  "passengers": 4,
  "baseline_price_pp": 127.43,
  "baseline_total": 509.72,
  "currency": "GBP",
  "search_params": {
    "origin": "LGW",
    "destination": "CTA",
    "departure_date": "2026-05-15",
    "return_date": "2026-05-20",
    "adults": 4,
    "non_stop": true,
    "currency": "GBP"
  },
  "created_at": "2026-04-03T09:00:00Z",
  "last_checked": null,
  "last_price": null,
  "status": "active"
}
```

**For accommodation:**
```json
{
  "type": "accommodation",
  "listing_name": "Beachfront Villa in Punta Secca",
  "platform": "airbnb",
  "destination": "Punta Secca, Sicily",
  "check_in": "2026-05-15",
  "check_out": "2026-05-20",
  "guests": 4,
  "baseline_price_night": 85.00,
  "baseline_total": 425.00,
  "currency": "GBP",
  "search_params": {
    "location": "Punta Secca, Sicily",
    "checkin": "2026-05-15",
    "checkout": "2026-05-20",
    "guests": 4,
    "ignoreRobotsText": true
  },
  "booking_url": "https://...",
  "created_at": "2026-04-03T09:00:00Z",
  "last_checked": null,
  "last_price": null,
  "status": "active"
}
```

### Step 2: Save The Watch Item

Save the watch item to `${CLAUDE_PLUGIN_DATA}/travel_watched_items.json`. Claude memory can be used as a secondary backup if useful.

### Step 3: Decide Whether Scheduled Execution Is Actually Available

If you are in Cowork and scheduled tasks are available:
- Tell the user this can be turned into a real recurring check.
- Instruct them to use `/schedule` or the Scheduled sidebar if Claude has not already confirmed task creation in the product UI.
- Prepare the recurring prompt and cadence for that task.

If you are in Claude Code or another environment without confirmed scheduling:
- Say that the watch has been recorded, but the recurring check is not a guaranteed background process.
- Offer a saved monitoring prompt the user can rerun.
- Do not imply that the plugin will wake itself up twice daily.

### Step 4: Confirm To The User

If scheduling is confirmed:

```text
Watching this for you. I have the baseline saved, and this can run on a twice-daily Cowork schedule.

Tracked item:
- [Item description], baseline: [price] as of [date/time]

You will get a Dispatch alert if the threshold is hit.
```

If scheduling is not confirmed:

```text
I have saved this to your watchlist with the current baseline.

Tracked item:
- [Item description], baseline: [price] as of [date/time]

I cannot honestly claim there is a native twice-daily background check running from this Claude Code plugin alone. If you want true recurring checks, set this up in Cowork scheduled tasks or ask me to re-check later.
```

## Recurring Check Workflow

When a recurring check does run, the logic is:

1. Load `${CLAUDE_PLUGIN_DATA}/travel_watched_items.json`.
2. Re-query the live tools using the stored `search_params`.
3. Compare the current price to the baseline and latest observed price.
4. Update the stored watch item.
5. Send a Dispatch alert only when the movement is meaningful.

## Alert Thresholds

| Change | Action |
|--------|--------|
| Price drops >10% | Send price-drop alert via Dispatch |
| Price drops >25% | Send major price-drop alert via Dispatch |
| Price rises >20% | Send price-increase warning |
| Price unchanged within 5% | No alert |
| Item unavailable | Send availability warning |

## Dispatch Alert Template

```text
Price Alert: [Item name]
Current: [current price] (was [baseline price])
Change: [percentage and amount]
Action: [book, rebook, or keep watching]
```

When fees or cancellation rules are relevant, include the net saving, not just the gross drop.

## Stopping A Watch

When the user says "stop watching" or "cancel price watch":

1. Load the watched items.
2. Ask which one to cancel if multiple are active.
3. Mark the selected item as `cancelled`.
4. If a Cowork scheduled task was explicitly created and identified, remove that task there as well.
5. Confirm what remains active.

## Listing Active Watches

When the user asks what is being watched, show each active item with baseline, latest observed price, and last-check time if known.
