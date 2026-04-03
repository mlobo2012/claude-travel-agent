---
name: price-reshop
description: "Post-booking price intelligence. Monitors booked flights and accommodation for price drops after purchase, calculates net savings after cancellation fees, and proactively alerts users with rebooking guidance. Auto-activates when bookings are detected — not user-invocable."
user-invocable: false
---

# Post-Booking Price Re-Shop

Monitor booked travel items for price drops after purchase. When a lower price is found, calculate net savings after cancellation fees and proactively alert the user with actionable rebooking guidance.

This skill is DISTINCT from `price-monitor`. Price-monitor watches items the user has NOT yet booked. Price-reshop watches items the user HAS booked, factoring in cancellation policies, rebooking fees, loyalty implications, and net savings calculations.

## When This Skill Activates

This skill activates automatically. It is never invoked directly by the user.

**Activation triggers:**
- The `booking-detection` skill logs a new booking to `travel_past_trips`
- The user tells Claude they have booked something (e.g., "I just booked flights to Rome", "I've reserved that Airbnb")
- The user forwards or mentions a booking confirmation email
- The user asks "can you watch my booking for price drops?" or "let me know if my flight gets cheaper"

**This skill does NOT activate for:**
- Pre-booking price watching (that is `price-monitor`)
- Items the user is merely considering
- Past trips where the departure date has already passed

## Data Storage

### Primary Store: `${CLAUDE_PLUGIN_DATA}/booked-trips.json`

All booked items under reshop monitoring are stored in this file. Each entry represents a single bookable component (one flight booking, one accommodation booking), not an entire trip.

```json
{
  "reshop_version": "2.1",
  "alert_thresholds": {
    "percentage_drop": 10,
    "absolute_drop": 50,
    "currency": "GBP",
    "mode": "either"
  },
  "items": []
}
```

### Item Schema — Flights

```json
{
  "id": "reshop_fl_20260403_093000",
  "type": "flight",
  "status": "monitoring",
  "airline": "easyJet",
  "operator_code": "U2",
  "flight_number": "U2 8765",
  "route": {
    "origin": "LGW",
    "origin_city": "London Gatwick",
    "destination": "CTA",
    "destination_city": "Catania"
  },
  "departure_date": "2026-05-15",
  "departure_time": "06:30",
  "return_date": "2026-05-20",
  "return_time": "11:15",
  "return_flight_number": "U2 8766",
  "passengers": 4,
  "booking_ref": "EZY123456",
  "price_paid": {
    "per_person": 127.43,
    "total": 509.72,
    "currency": "GBP"
  },
  "booking_date": "2026-04-03T09:30:00Z",
  "booking_source": "easyjet.com",
  "cancellation_policy": {
    "type": "standard",
    "free_cancellation_window_hours": 24,
    "free_cancellation_deadline": "2026-04-04T09:30:00Z",
    "cancellation_fee_per_person": 45.00,
    "cancellation_fee_total": 180.00,
    "refund_method": "travel_credit",
    "refund_method_notes": "easyJet issues voucher valid 12 months, not cash refund",
    "change_fee_per_person": 30.00,
    "price_match_available": false,
    "notes": "Flexi fare customers: free changes up to 2h before departure"
  },
  "loyalty": {
    "programme": null,
    "points_earned": 0,
    "tier_credits": 0,
    "notes": null
  },
  "search_params": {
    "origin": "LGW",
    "destination": "CTA",
    "departure_date": "2026-05-15",
    "return_date": "2026-05-20",
    "adults": 4,
    "non_stop": true,
    "currency": "GBP"
  },
  "price_history": [
    {
      "timestamp": "2026-04-03T09:30:00Z",
      "price_pp": 127.43,
      "price_total": 509.72,
      "source": "booking",
      "note": "Original booking price"
    }
  ],
  "trend_analysis": {
    "direction": "stable",
    "average_price_pp": 127.43,
    "min_price_pp": 127.43,
    "max_price_pp": 127.43,
    "checks_count": 1,
    "pattern_notes": null
  },
  "alerts_sent": [],
  "created_at": "2026-04-03T09:30:00Z",
  "last_checked": null,
  "next_check": "2026-04-03T18:00:00Z",
  "monitoring_expires": "2026-05-15T06:30:00Z"
}
```

### Item Schema — Accommodation (Hotels, Airbnb, VRBO)

```json
{
  "id": "reshop_acc_20260403_093500",
  "type": "accommodation",
  "status": "monitoring",
  "platform": "airbnb",
  "listing_name": "Beachfront Villa in Punta Secca",
  "listing_id": "airbnb_12345678",
  "host_name": "Marco",
  "address": "Via Example 42, Punta Secca, RG, Sicily",
  "destination_city": "Punta Secca",
  "check_in": "2026-05-15",
  "check_in_time": "15:00",
  "check_out": "2026-05-20",
  "check_out_time": "10:00",
  "nights": 5,
  "guests": 4,
  "booking_ref": "HM12345678",
  "price_paid": {
    "per_night": 85.00,
    "subtotal": 425.00,
    "cleaning_fee": 60.00,
    "service_fee": 52.00,
    "total": 537.00,
    "currency": "GBP"
  },
  "booking_date": "2026-04-03T09:35:00Z",
  "booking_source": "airbnb.com",
  "cancellation_policy": {
    "type": "moderate",
    "free_cancellation_deadline": "2026-05-08T15:00:00Z",
    "free_cancellation_days_before": 7,
    "partial_refund_available": true,
    "partial_refund_deadline": "2026-05-14T15:00:00Z",
    "partial_refund_percentage": 50,
    "cancellation_fee": 0,
    "refund_method": "original_payment",
    "notes": "Full refund if cancelled 7+ days before check-in. 50% refund if cancelled 1-7 days before."
  },
  "loyalty": {
    "programme": null,
    "points_earned": 0,
    "tier_credits": 0,
    "notes": null
  },
  "search_params": {
    "location": "Punta Secca, Sicily",
    "checkin": "2026-05-15",
    "checkout": "2026-05-20",
    "guests": 4,
    "ignoreRobotsText": true
  },
  "price_history": [
    {
      "timestamp": "2026-04-03T09:35:00Z",
      "price_per_night": 85.00,
      "price_total": 537.00,
      "source": "booking",
      "note": "Original booking price"
    }
  ],
  "trend_analysis": {
    "direction": "stable",
    "average_price_per_night": 85.00,
    "min_price_per_night": 85.00,
    "max_price_per_night": 85.00,
    "checks_count": 1,
    "pattern_notes": null
  },
  "alternative_listings": [],
  "alerts_sent": [],
  "created_at": "2026-04-03T09:35:00Z",
  "last_checked": null,
  "next_check": "2026-04-03T18:00:00Z",
  "monitoring_expires": "2026-05-20T10:00:00Z"
}
```

### Item Status Values

| Status | Meaning |
|--------|---------|
| `monitoring` | Actively checking for price drops |
| `alert_sent` | Price drop found, user notified, awaiting response |
| `rebooked` | User confirmed rebooking; monitoring stopped |
| `expired` | Departure/check-in date has passed |
| `cancelled_by_user` | User asked to stop monitoring this item |
| `cancelled_free_window` | Free cancellation window passed without price drop; monitoring continues but alerts note fees now apply |

## Extracting Booking Details

When the user tells Claude about a booking (conversationally, not via email detection):

### Step 1: Ask Clarifying Questions If Needed

Extract as much as possible from the user's message. If critical fields are missing, ask:

**Always needed:**
- What was the total price paid?
- What are the exact dates?
- Confirmation/booking reference number?

**Ask if not obvious:**
- Which airline or platform?
- Cancellation policy type (if not inferrable from airline/platform)?
- Did you book directly or through an aggregator?

**Never ask (infer or default):**
- Currency (use user profile default from `travel_profile`)
- Passengers/guests (use user profile default or 1)
- Search parameters (reconstruct from route and dates)

### Step 2: Infer Cancellation Policy

If the user does not know their cancellation policy, apply these defaults:

**Airlines — Known Policies (as of 2026):**
| Airline | Free Cancel Window | Cancel Fee | Refund Type | Change Fee | Notes |
|---------|-------------------|------------|-------------|------------|-------|
| British Airways | 24h | Fare rules apply | Cash (flex) / Voucher (basic) | From 0 (flex) to full fare | Avios bookings: fully refundable minus taxes |
| easyJet | 24h | From 45pp | Travel voucher | From 30pp | Flexi: free changes up to 2h before |
| Ryanair | 24h (if booked direct) | Full fare minus taxes | Cash (taxes only) | From 45pp + fare diff | Value/Regular: only tax refund on cancel |
| Wizz Air | 24h | From 40pp (Wizz Flex: free) | Wizz credit | From 35pp + fare diff | Wizz Flex: unlimited free changes |
| Jet2 | 24h | Sliding scale by days-out | Cash refund minus fee | From 35pp | Package holidays: ATOL protected |
| Emirates | 24h | Fare class dependent | Cash or miles | From 75pp | Flex Plus: fully changeable |
| BA (Avios) | Until midnight before | Avios returned + taxes refunded | Avios + cash | Free date changes | Best reshop candidate |

**Accommodation Platforms — Known Policies:**
| Platform | Common Policies | Notes |
|----------|----------------|-------|
| Airbnb | Flexible (24h/48h), Moderate (5-7 days), Strict (14 days), Super Strict | Check listing-specific policy |
| Booking.com | Free cancellation (typically 24-48h before), Non-refundable (10-20% cheaper) | "Risk-free" bookings can be cancelled up to check-in |
| VRBO | Host-set: Relaxed (14d), Moderate (5d), Strict (60d), No Refund | Service fee usually non-refundable |
| Hotels.com | Varies by property; typically mirrors Booking.com | Rewards nights: separate policy |

### Step 3: Save to booked-trips.json

Create the item object using the schemas above and append to the `items` array in `${CLAUDE_PLUGIN_DATA}/booked-trips.json`. If the file does not exist, create it with the wrapper schema.

### Step 4: Register with Scheduled Task System

The price-reshop system integrates with the SAME scheduled task created by `price-monitor`. Extend the existing twice-daily task (cron `0 8,18 * * *`) to also process reshop items.

**If no scheduled task exists yet**, create one with this prompt:

> "Run price check for travel items. First, recall `travel_watched_items` from persistent memory and check pre-booking watches. Then, load `${CLAUDE_PLUGIN_DATA}/booked-trips.json` and check all items with status `monitoring`. For each reshop item, re-query the MCP tools with the saved search_params. Compare the current price to the price_paid. Append the new price to price_history. If a price drop exceeds the user's alert thresholds, send a reshop alert via Dispatch. Update trend_analysis fields."

**If the scheduled task already exists** (from price-monitor), update its prompt to include the reshop logic above.

### Step 5: Confirm to User

```
I'm now watching your booking for price drops.

[Booking type icon] [Description — e.g., "easyJet LGW to CTA, 15-20 May"]
Booked at: [price] on [date]
Cancellation policy: [summary]
Free cancellation until: [deadline, if applicable]

I'll check the price twice daily and alert you if it drops below what you paid. If the savings outweigh any cancellation fees, I'll let you know exactly how much you'd save net.
```

## Price Check Execution (Scheduled Task)

When the scheduled task fires, for each reshop item with `status: "monitoring"`:

### Step 1: Pre-Check Validation

Before querying prices, check:
1. **Has the departure/check-in date passed?** If yes, set `status: "expired"` and skip.
2. **Is the item within 24 hours of departure?** If yes, skip (prices are volatile and rebooking is impractical).
3. **Has the free cancellation window passed since last check?** If yes, update `status` to `cancelled_free_window` (keep monitoring, but all future alerts will factor in the cancellation fee).

### Step 2: Query Current Price

**For flights:**
- Use `search_flights` MCP tool with the saved `search_params`
- Find the SAME airline and route in results
- Also note the CHEAPEST overall option across all airlines (for cross-carrier alerts)
- Record both: same-airline price and best-available price

**For accommodation:**
- Use `airbnb_search` or equivalent MCP tool with saved `search_params`
- Find the SAME listing by name/ID match
- Also note any cheaper COMPARABLE listings (similar rating, location, capacity)
- Record both: same-listing price and best-alternative price

### Step 3: Update Price History

Append to the item's `price_history` array:

```json
{
  "timestamp": "2026-04-05T08:00:00Z",
  "price_pp": 112.00,
  "price_total": 448.00,
  "source": "scheduled_check",
  "same_airline": true,
  "best_alternative": {
    "airline": "Ryanair",
    "price_pp": 89.00,
    "price_total": 356.00,
    "flight_number": "FR 4521",
    "departure_time": "07:15"
  },
  "note": null
}
```

### Step 4: Update Trend Analysis

After appending the new price, recalculate:

```json
{
  "direction": "falling",
  "average_price_pp": 119.71,
  "min_price_pp": 112.00,
  "max_price_pp": 127.43,
  "checks_count": 3,
  "consecutive_drops": 2,
  "drop_velocity_per_day": -3.86,
  "pattern_notes": "Price has fallen 12.1% over 4 days. At this rate, could drop another 5-8% before departure."
}
```

**Direction values:** `stable`, `falling`, `rising`, `volatile` (changes direction between checks), `bottomed_out` (dropped then stabilised for 3+ checks).

**Pattern detection rules:**
- If price drops for 3+ consecutive checks: "Consistent downward trend — consider waiting for a further drop before rebooking"
- If price dropped sharply then stabilised: "Price may have bottomed out — this could be the best time to rebook"
- If price is volatile (up/down/up): "Prices are fluctuating — current low may not last"
- If departure is within 14 days and price is falling: "Late price drops on this route are unusual — act quickly as this may reverse"
- If historical data from `travel_past_trips` shows a pattern for this route: "This route to CTA typically drops 15% around 3 weeks before departure based on your previous trips"

### Step 5: Evaluate Alert Thresholds

Load user-specific thresholds from `${CLAUDE_PLUGIN_DATA}/booked-trips.json` (top-level `alert_thresholds`). If no custom thresholds, use defaults.

**Default thresholds:**
- Alert if price drops **more than 10%** from `price_paid`
- OR alert if price drops **more than 50 in the user's currency** (GBP/EUR/USD) from `price_paid`
- Mode `"either"` means alert triggers when EITHER condition is met (i.e., the LESS restrictive condition)

**Threshold evaluation:**
```
percentage_drop = ((price_paid.total - current_price_total) / price_paid.total) * 100
absolute_drop = price_paid.total - current_price_total

if mode == "either":
    alert = (percentage_drop > threshold.percentage_drop) OR (absolute_drop > threshold.absolute_drop)
elif mode == "both":
    alert = (percentage_drop > threshold.percentage_drop) AND (absolute_drop > threshold.absolute_drop)
```

**Per-user threshold customisation:**
Users can say "only alert me if I'd save more than 100 pounds" or "alert me on any drop over 5%". Update `alert_thresholds` accordingly.

### Step 6: Calculate Net Savings

If alert threshold is exceeded, calculate net savings:

```
gross_savings = price_paid.total - new_price_total
cancellation_fee = cancellation_policy.cancellation_fee_total (0 if within free window)
change_fee = cancellation_policy.change_fee_per_person * passengers (if route change)
rebooking_fee = 0 (if same airline, same route — just cancel + rebook)
net_savings = gross_savings - cancellation_fee - change_fee - rebooking_fee
```

**Only alert if net_savings > 0.** Never recommend rebooking at a loss.

**Special case — travel credit refunds:**
If the cancellation refund is issued as travel credit (not cash), note this prominently. Net savings calculation is the same, but the alert must clearly state: "You would receive [amount] in [airline] travel credit, not a cash refund. Net benefit depends on whether you'll use the credit."

### Step 7: Send Alert (If Warranted)

Send the appropriate alert template via Dispatch.

## Alert Templates

### Template 1: Same-Carrier Price Drop (Within Free Cancellation Window)

```
PRICE DROP on your booking — FREE to act on!

[flight/hotel icon] [Description — e.g., "easyJet LGW to CTA, 15 May"]
Booking ref: [ref]

ORIGINAL PRICE:  [price_paid.total] ([price_paid.per_person] pp)
CURRENT PRICE:   [new_total] ([new_pp] pp)
YOU SAVE:        [gross_savings] ([percentage]% less)

Cancellation fee: FREE (within your free cancellation window)
NET SAVINGS:     [gross_savings]

Your free cancellation window closes [deadline]. Act before then to lock in these savings at zero cost.

RECOMMENDED ACTION: Cancel your current booking and rebook at the lower price.

Steps:
1. Cancel existing booking [ref] via [airline/platform] — free within your window
2. Rebook the same [flight/listing] at the new price
3. [Direct booking link]

Reply "rebook" and I'll guide you through it step by step.
Reply "ignore" to dismiss this alert.
```

### Template 2: Same-Carrier Price Drop (After Free Cancellation Window)

```
PRICE DROP on your booking

[flight/hotel icon] [Description]
Booking ref: [ref]

ORIGINAL PRICE:  [price_paid.total] ([price_paid.per_person] pp)
CURRENT PRICE:   [new_total] ([new_pp] pp)
GROSS SAVINGS:   [gross_savings] ([percentage]% less)

Cancellation fee: [fee]
Refund type:      [cash/travel credit/voucher]
NET SAVINGS:      [net_savings]

[If refund is travel credit:]
Note: Your [cancellation refund amount] refund would be issued as [airline] travel credit valid for [duration], not cash. This is still worthwhile if you plan to fly [airline] again within that period.

RECOMMENDED ACTION: [one of the below]
- "Cancel and rebook — you save [net_savings] net after the [fee] cancellation fee."
- "Marginal savings — the [net_savings] net saving may not be worth the hassle of rebooking."
- "Hold off — prices are still dropping. I'll alert you again if the saving increases."

[Direct booking link]

Reply "rebook" to proceed, or "watch" to keep monitoring.
```

### Template 3: Cross-Carrier Price Drop (Different Airline Cheaper)

```
CHEAPER ALTERNATIVE found for your route

[flight icon] Your booking: [airline] [route], [date]
Booking ref: [ref] | Paid: [price_paid.total]

ALTERNATIVE: [alt_airline] [alt_flight_number]
Departs: [alt_departure_time] (vs your [departure_time])
Price:   [alt_total] ([alt_pp] pp)

GROSS SAVINGS:      [gross_savings]
Cancellation fee:   [fee] for [airline] booking
NET SAVINGS:        [net_savings]

[If loyalty implications exist:]
LOYALTY NOTE: Rebooking from [airline] to [alt_airline] means:
- You lose [points/miles] [programme] [points_type] from your current booking
- [alt_airline] [loyalty info if known]
- Net loyalty impact: [summary]

[If different departure time:]
SCHEDULE NOTE: The [alt_airline] flight departs at [time] instead of [time]. [If significantly different: "This is [N] hours earlier/later — check this works for your plans."]

RECOMMENDED ACTION: [contextual recommendation]

Reply "rebook [alt_airline]" to proceed, or "ignore" to keep your current booking.
```

### Template 4: Accommodation Price Drop (Same Listing)

```
PRICE DROP on your accommodation

[accommodation icon] [Listing name], [destination]
Booking ref: [ref]

ORIGINAL PRICE:  [price_paid.total] ([price_paid.per_night]/night)
CURRENT PRICE:   [new_total] ([new_per_night]/night)
YOU SAVE:        [gross_savings] ([percentage]% less)

Cancellation policy: [policy summary]
[If within free cancellation window:]
  FREE to cancel until [deadline] — no fees apply.
[If outside window:]
  Cancellation fee: [fee or forfeited amount]
  NET SAVINGS: [net_savings]

RECOMMENDED ACTION: [Cancel and rebook / Contact host about price match / Hold — prices may drop further]

[Direct listing link]
[Cancel current booking link if known]

Reply "rebook" to proceed.
```

### Template 5: Accommodation Alternative Found (Comparable Listing, Lower Price)

```
CHEAPER ALTERNATIVE for your stay in [destination]

Your booking: [listing_name] — [price_paid.total] total
Alternative:  [alt_listing_name] — [alt_total] total (save [savings])

Comparison:
| | Your booking | Alternative |
|---|---|---|
| Price/night | [price] | [alt_price] |
| Rating | [rating] | [alt_rating] |
| Location | [location] | [alt_location] |
| Sleeps | [capacity] | [alt_capacity] |

Cancellation cost: [fee or "free"]
NET SAVINGS: [net_savings]

[Direct link to alternative listing]

Reply "switch" to proceed, or "ignore" to keep your current booking.
```

### Template 6: Trend Insight (No Immediate Alert, But Useful Pattern)

Sent when a notable trend is detected but the threshold is not yet met:

```
PRICE TREND for your [flight/stay]

[Description]

Your booking price: [price_paid.total]
Current price:      [current_total] ([direction] [percentage]%)

Trend: [pattern_notes — e.g., "Price has dropped 7% over the past week and is still falling. If this continues, it may cross your alert threshold within 2-3 days."]

[If historical pattern detected:]
Based on your previous trips, prices on this route tend to [pattern — e.g., "bottom out around 3 weeks before departure, which is [date] for this trip"].

No action needed now — I'm keeping an eye on it.
```

### Template 7: Cancellation Window Warning

Sent 24 hours before the free cancellation deadline if a price drop is close to (within 5% of) the alert threshold:

```
CANCELLATION WINDOW CLOSING SOON

[Description]
Booking ref: [ref]

Your free cancellation window closes in [hours] hours ([deadline]).

Current price: [current_total] (down [percentage]% from your [price_paid.total])
This is below what you paid but hasn't crossed your alert threshold of [threshold]% / [threshold_amount] yet.

After the window closes, cancellation will cost [fee].

OPTIONS:
1. Cancel now and rebook at [current_total] — save [savings] with no fees
2. Keep your booking — if prices drop further, cancellation fees will apply
3. Adjust your alert threshold — say "alert me on any drop" to lower the bar

Reply with your choice.
```

## Cross-Feature Integration

### Integration with booking-detection

When `booking-detection` logs a new trip to `travel_past_trips`:
1. Automatically create reshop entries for each bookable component (flights, accommodation)
2. Begin monitoring immediately
3. Send the Step 5 confirmation to the user

### Integration with price-monitor

When a watched item from `price-monitor` is booked:
1. `price-monitor` cancels its watch (as per its existing logic)
2. `price-reshop` picks up monitoring from the booked price
3. Carry over any price history from the watch into the reshop item's `price_history` (prefix entries with `source: "pre_booking_watch"`)
4. This provides a longer trend baseline for pattern detection

### Integration with travel profile

Load the user's profile from `travel_profile` persistent memory or `${CLAUDE_PLUGIN_DATA}/travel-profile.json` to:
- Determine default currency for thresholds
- Check loyalty programme memberships for cross-carrier alerts
- Know home airport (for re-search parameters)
- Apply user's risk tolerance preferences if set

### Integration with trip-reminders

If a reshop alert is sent and the user confirms rebooking:
1. Update the trip in `travel_past_trips` with new booking details
2. Update any trip-reminders with new flight numbers/times
3. Update any calendar events via calendar-integration
4. Regenerate the trip pack if it has already been created

## Loyalty Impact Assessment

When evaluating a cross-carrier rebooking, calculate loyalty implications:

```json
{
  "current_booking_loyalty": {
    "programme": "Avios",
    "points_earned": 4500,
    "points_value_estimate": 45.00,
    "tier_points": 80,
    "status_impact": "None — you need 600 more tier points for Silver"
  },
  "alternative_loyalty": {
    "programme": null,
    "points_earned": 0,
    "notes": "Ryanair does not participate in a transferable loyalty programme"
  },
  "net_loyalty_cost": {
    "points_lost": 4500,
    "estimated_value_lost": 45.00,
    "recommendation": "The 80 net savings outweigh the 45 estimated Avios value. Rebooking is still beneficial if you value the points at the standard 1p/Avios rate."
  }
}
```

**Known programme valuations (conservative estimates):**
| Programme | Point/Mile Value | Notes |
|-----------|-----------------|-------|
| Avios (BA/Iberia) | 0.8-1.2p per Avios | Higher for premium cabin redemptions |
| Virgin Points | 0.6-1.0p per point | Best value on Upper Class upgrades |
| Emirates Skywards | 0.5-1.0p per mile | Tiered by cabin |
| SAS EuroBonus | 0.5-0.8p per point | |
| Miles & More | 0.3-0.7p per mile | Varies widely by redemption |

## User Threshold Customisation

Users can customise thresholds conversationally:

| User says | Action |
|-----------|--------|
| "Only alert me for big drops" | Set `percentage_drop: 20, absolute_drop: 100` |
| "Alert me on any price drop" | Set `percentage_drop: 1, absolute_drop: 1` |
| "Only alert if I'd save at least 100 pounds" | Set `absolute_drop: 100, mode: "both"` |
| "Stop watching my Airbnb booking" | Set that item's `status: "cancelled_by_user"` |
| "Stop all reshop monitoring" | Set all items to `status: "cancelled_by_user"` and remove from scheduled task |
| "Show my booked price watches" | List all items with `status: "monitoring"` or `status: "alert_sent"` |

When thresholds are updated, confirm:
```
Updated your reshop alert threshold:
- Alert when price drops more than [X]% OR more than [Y currency amount] (whichever happens first)
- Currently monitoring [N] booked items
```

## Auto-Expiry Rules

Monitoring automatically stops when:

| Condition | Action |
|-----------|--------|
| Departure date/check-in date has passed | Set `status: "expired"` |
| User confirms rebooking | Set `status: "rebooked"`, log new booking details |
| Item is within 24 hours of departure | Set `status: "expired"` (too late to act) |
| User cancels for reasons unrelated to reshop | Set `status: "cancelled_by_user"` |

On each scheduled check, sweep all items and expire any that match these conditions before running price queries. This avoids wasting API calls on stale bookings.

## Edge Cases

### Multiple Passengers, Mixed Cancellation
Some bookings allow cancelling individual passengers. If the airline supports partial cancellation, note this: "You could cancel and rebook 2 of 4 passengers at the lower fare, saving [amount] while keeping [passengers] on the original booking."

### Currency Fluctuations
If the user booked in a foreign currency, track prices in the BOOKING currency, not the user's home currency. Note currency movements only if they exceed 3%: "The flight is also [X]% cheaper in GBP terms due to a favourable exchange rate movement."

### Sold-Out Reshop
If the exact same flight/listing is now sold out but a price drop appeared earlier: "This flight is currently sold out at [price]. I'll keep monitoring in case availability returns."

### Price Rose, Then Dropped Below Original
If price went up after booking and then dropped: "Price has returned to below what you paid after a temporary increase. Current price [X] is [Y]% below your booking price." Do not include the peak price in the alert — it is not relevant to the savings calculation.

### Same Route, Different Time
If the same airline offers the route cheaper at a different time of day: note the time difference prominently. If it is more than 3 hours different, flag it as a schedule impact: "This cheaper option departs at [time] instead of [time] — a [N]-hour difference. Check this works with your plans before rebooking."

### Airline Schedule Change
If the airline changes the flight time on the user's existing booking, this may trigger new consumer rights (EU261 for EU flights). Note: "Your airline has changed your flight time by [N] hours. Under EU261, you may be entitled to a full refund. This makes reshop risk-free." This applies when schedule changes exceed 2 hours for EU-departing flights.

### Non-Refundable Bookings
For non-refundable bookings, still monitor. Alert only for very large drops (>30% or >150 currency units) and frame as: "Your booking is non-refundable, but the price has dropped [amount]. If you need to change dates for any reason, rebooking now would be significantly cheaper. Also worth checking: some credit cards offer price protection — check your card benefits."

### Booking Through an Aggregator
If the user booked through Skyscanner, Kayak, Google Flights, etc. rather than direct: "You booked via [aggregator]. Cancellation policies from the actual provider ([airline/hotel]) apply. Check the confirmation email from [provider] for the definitive cancellation terms."

## Monitoring Summary View

When the user asks "what are you reshop monitoring?" or "show my booked price watches":

```
Your Booked Trip Price Monitoring

1. [flight icon] easyJet LGW to CTA, 15-20 May
   Booked: 509.72 GBP | Current: 468.00 GBP | Change: -8.2%
   Trend: Falling (3 consecutive drops)
   Free cancel until: 4 Apr 09:30
   Status: Monitoring

2. [accommodation icon] Beachfront Villa, Punta Secca, 15-20 May
   Booked: 537.00 GBP | Current: 537.00 GBP | Change: 0%
   Trend: Stable
   Free cancel until: 8 May 15:00
   Status: Monitoring

Alert threshold: >10% or >50 GBP drop (whichever comes first)

Say "stop reshop [number]" to cancel monitoring for an item.
Say "set alert threshold [value]" to change your sensitivity.
```

## Privacy and Data Handling

- All booking data is stored locally in `${CLAUDE_PLUGIN_DATA}/booked-trips.json` and in Cowork persistent memory
- Booking references and personal data never leave the local environment except when querying prices via MCP tools (which only send search parameters, not booking refs)
- Price history is retained indefinitely for trend analysis but can be purged on request: "clear my reshop history"
- When a user says "forget my booking", remove the item from `booked-trips.json` AND from `travel_past_trips` if requested
