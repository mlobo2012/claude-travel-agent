---
name: price-monitor
description: "Price monitoring for flights and accommodation. Creates scheduled tasks to check prices twice daily and alerts the user on significant changes. Use when: user says 'watch this', 'monitor the price', 'alert me if price drops', or expresses interest without booking."
argument-hint: "[flight or listing reference]"
---

# Price Monitor

Monitor prices for flights and accommodation using Cowork's scheduled tasks, alerting the user on significant changes.

## When to Activate

- User says "watch this for me", "monitor the price", "let me know if it drops"
- User says "I'm interested but not ready to book"
- User says "alert me if the price changes"
- User explicitly runs `/price-monitor`

## Setting Up a Price Watch

### Step 1: Capture Baseline

Record the current state of the item being watched:

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

### Step 2: Save to Persistent Memory

Save the watch item to `travel_watched_items` in persistent memory.

### Step 3: Create Scheduled Task

Create a scheduled task that runs **twice daily** — at 8:00 AM and 6:00 PM in the user's local timezone.

Use Cowork's scheduled task system:
- Create a task with a cron expression: `0 8,18 * * *`
- Task prompt: "Run price check for watched travel items. Recall `travel_watched_items` from persistent memory. For each active item, re-query the MCP tools with the saved search_params. Compare the current price to the baseline_price. Report any significant changes via Dispatch."

### Step 4: Confirm to User

```
Watching this for you! I'll check the price twice daily (8am and 6pm) and alert you if anything changes significantly.

Currently watching:
- [Item description] — baseline: [price] as of [date/time]

Say "stop watching" or "cancel price watch" to stop monitoring.
Say "what are you watching?" to see all active watches.
```

## Price Check Execution (Scheduled Task)

When the scheduled task fires:

1. Recall `travel_watched_items` from persistent memory
2. For each item with `status: "active"`:
   a. Re-query the same MCP tool with the saved `search_params`
   b. Record the current price
   c. Compare to baseline price
   d. Update `last_checked` and `last_price` in memory

### Alert Thresholds

| Change | Action |
|--------|--------|
| Price drops >10% | Send PRICE DROP alert via Dispatch |
| Price drops >25% | Send MAJOR PRICE DROP alert via Dispatch (urgent) |
| Price rises >20% | Send PRICE INCREASE warning via Dispatch |
| Price unchanged (within 5%) | No alert, silent update |
| Item unavailable | Send AVAILABILITY WARNING via Dispatch |

### Price Drop Alert Template

```
PRICE DROP ALERT

[listing/flight emoji] [Item name — e.g., "easyJet LGW to CTA, 15 May"]
Was: [baseline price] -> Now: [current price] (down [percentage]% / saving [amount])
Price checked at [HH:MM today]
[Book now]([booking_link])

Reply "stop watching" to cancel this alert.
```

### Price Increase Warning Template

```
PRICE INCREASE WARNING

[listing/flight emoji] [Item name]
Was: [baseline price] -> Now: [current price] (up [percentage]%)
Price checked at [HH:MM today]

Consider booking soon if you're interested — prices are trending upward.
[Book now]([booking_link])

Reply "stop watching" to cancel this alert.
```

### Availability Warning Template

```
AVAILABILITY WARNING

[Item name] is no longer showing as available for your dates.
Last known price: [last_price] at [last_checked]

This could mean it's been booked by someone else, or the listing has been temporarily removed.
[Check listing]([booking_link])

Reply "stop watching" to cancel this alert.
```

## Stopping a Watch

When the user says "stop watching", "cancel price watch", "stop monitoring":

1. Recall `travel_watched_items` from persistent memory
2. If multiple items are being watched, ask which one to cancel
3. Set `status: "cancelled"` on the selected item
4. Delete the associated scheduled task
5. Confirm: "Stopped watching [item]. You have [N] remaining active watches."

## Auto-Cancellation

A watch is automatically cancelled when:
- The travel date has passed (departure date or check-in date is in the past)
- The user books the item (detected via Gmail or manual confirmation)
- 30 days have passed since the watch was created

## Listing Active Watches

When the user asks "what are you watching?" or "show my price watches":

```
Your Active Price Watches

1. [Type emoji] [Item description]
   Baseline: [price] ([date]) | Last check: [price] ([date/time]) | Change: [+/-percentage]

2. [Type emoji] [Item description]
   Baseline: [price] ([date]) | Last check: [price] ([date/time]) | Change: [+/-percentage]

Say "stop watching [number]" to cancel a specific watch.
```
