---
name: document-scanner
description: "Scan and extract travel data from photos of receipts, boarding passes, booking confirmations, and hotel bills using Claude's vision capability. Logs expenses, updates trip records, detects overcharges, and generates expense reports. Use when: user uploads a travel document image or says 'scan this', 'log this receipt', 'check my hotel bill', or runs /travel-expenses."
user-invocable: true
argument-hint: "[boarding-pass | receipt | booking | hotel-bill | expenses]"
---

# Document Scanner

Extract structured travel data from photographs and screenshots of travel documents. Uses Claude's vision capability to read and interpret receipts, boarding passes, booking confirmations, and hotel bills. All extracted data is logged to persistent storage, matched against existing trips, and surfaced to the user with actionable insights.

## When to Activate

- User uploads an image of a travel document (receipt, boarding pass, booking confirmation, hotel bill)
- User says "scan this", "read this receipt", "log this expense", "check my hotel bill"
- User pastes a screenshot of a booking confirmation email
- User runs `/travel-expenses [trip-name]` to generate an expense report
- User says "how much have I spent on this trip?"

## Document Type Detection

When the user uploads an image without specifying the type, auto-detect by examining visual cues:

| Document Type | Visual Indicators |
|---|---|
| Boarding pass | Barcode/QR code, gate, seat, boarding time, airline logo, "BOARDING PASS" header |
| Receipt | Itemised list, total amount, vendor name, date/time, payment method |
| Booking confirmation | Reservation number, check-in/out dates, property name, "confirmed" language |
| Hotel bill | Room charges, nightly rate, itemised extras, hotel letterhead, folio number |

If detection is ambiguous, ask the user: "This looks like it could be a [type A] or [type B]. Which is it?"

---

## 1. Boarding Pass Scanning

### Extraction Fields

Read the boarding pass image and extract every available field:

| Field | Example | Required |
|---|---|---|
| Passenger name | `SMITH/JOHN MR` | Yes |
| Flight number | `BA 2765` | Yes |
| Airline | `British Airways` | Yes |
| Origin airport (code + name) | `LHR` / `London Heathrow` | Yes |
| Destination airport (code + name) | `BCN` / `Barcelona El Prat` | Yes |
| Date of travel | `2026-05-15` | Yes |
| Departure time | `14:50` | Yes |
| Boarding time | `14:10` | If visible |
| Gate | `B24` | If visible |
| Seat | `14A` | If visible |
| Boarding group/zone | `Group 2` | If visible |
| Class of service | `Economy` / `Business` / `First` | If visible |
| Frequent flyer number | `BA12345678` | If visible |
| Frequent flyer tier | `Gold` / `Silver` | If visible |
| Booking reference | `ABC123` | If visible |
| Sequence number | `042` | If visible |
| Barcode format | `BCBP` / `QR` | Detected automatically |

### Processing Steps

#### Step 1: Extract All Fields

Use Claude's vision to read the boarding pass. Boarding passes use a mix of printed text, barcodes, and sometimes handwritten gate changes. Read all visible text. If the image quality is poor, report which fields could not be read and ask the user to confirm.

#### Step 2: Match to Existing Trip

1. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/booked-trips.json` (or check Claude's memory as fallback)
2. Match by flight number + date (exact match)
3. If no exact match, try matching by route + date (within +/- 1 day tolerance)
4. If no match found, create a new trip entry with status `in_progress`

#### Step 3: Update Trip Record

Update the matched trip with actual boarding pass data:

```json
{
  "flights": {
    "outbound": {
      "actual_gate": "B24",
      "actual_seat": "14A",
      "boarding_time": "14:10",
      "boarding_group": "Group 2",
      "class_of_service": "Economy",
      "sequence_number": "042",
      "boarding_pass_scanned": true,
      "scanned_at": "2026-05-15T12:30:00Z"
    }
  }
}
```

#### Step 4: Frequent Flyer Update

If a frequent flyer number is visible on the boarding pass:

1. Recall the loyalty-manager skill data
2. Check if this frequent flyer number is already on file
3. If new: add it to the loyalty profile — "I found frequent flyer number BA12345678 on your boarding pass. I've added it to your loyalty profile."
4. If existing: confirm it matches — no action needed
5. If different from what is on file: warn the user — "Your boarding pass shows BA99999999 but I have BA12345678 on file. Want me to update your loyalty profile?"

#### Step 5: Confirm to User

```
Boarding pass scanned.

Passenger: John Smith
Flight: BA 2765 — London Heathrow (LHR) to Barcelona (BCN)
Date: 15 May 2026, departing 14:50
Gate: B24, boarding at 14:10
Seat: 14A (Economy), Group 2

Trip record updated. This matches your Barcelona trip (15-20 May).
```

If critical information is missing from the scan:
```
I was able to read most of your boarding pass but couldn't make out the gate number — the image is slightly blurred in that area. Could you confirm the gate?
```

---

## 2. Receipt and Bill Scanning

### Extraction Fields

| Field | Example | Required |
|---|---|---|
| Vendor/merchant name | `The Wolseley` | Yes |
| Date | `2026-05-16` | Yes |
| Time | `19:42` | If visible |
| Total amount | `87.50` | Yes |
| Currency | `GBP` | Yes (inferred from symbol/country if not explicit) |
| Payment method | `Visa ending 4821` | If visible |
| Itemised line items | Array of `{description, qty, unit_price, line_total}` | If visible |
| VAT/tax amount | `14.58` | If visible |
| VAT rate | `20%` | If visible |
| Tax registration number | `GB 123 4567 89` | If visible |
| Tip/gratuity | `13.13` | If visible |
| Service charge | `10.50` | If visible |

### Expense Categories

Assign each receipt to one of these categories based on vendor type and item descriptions:

| Category | Key | Examples |
|---|---|---|
| Transport | `transport` | Taxi, Uber, train ticket, bus fare, fuel, parking, car rental, tolls |
| Accommodation | `accommodation` | Hotel, Airbnb, hostel (only use for incidentals not covered by the main booking) |
| Food & Drink | `food` | Restaurant, cafe, bar, supermarket groceries, room service, minibar |
| Activities & Entertainment | `activities` | Museum, tour, theatre, concert, attraction tickets, excursion |
| Shopping | `shopping` | Souvenirs, clothing, gifts, personal items |
| Communication | `communication` | SIM card, phone top-up, WiFi purchase |
| Health & Personal | `health` | Pharmacy, medical, laundry, dry cleaning |
| Other | `other` | Anything that does not fit the above categories |

If the category is ambiguous, make your best judgement and note it: "I categorised this as 'food' — let me know if you'd prefer a different category."

### Currency Detection

Determine the currency from these signals (in priority order):

1. Explicit currency code printed on receipt (GBP, EUR, USD, etc.)
2. Currency symbol: `£` = GBP, `$` = USD (unless context suggests AUD/CAD/etc.), `€` = EUR, `kr` = SEK/NOK/DKK (determine from country), `CHF`, `zl` / `PLN`
3. Country of vendor (from address, language, or known chain location)
4. Active trip destination currency

If the receipt currency differs from the user's home currency (use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel-profile.json` to check; follow the persistent-memory fallback chain: file first, then Claude's memory, then ask the user), add a conversion note using approximate current rates:

```
Note: €87.50 is approximately £74.80 at today's rate (€1 = £0.855).
```

### VAT/Tax Extraction

When VAT or tax lines are visible on the receipt:

1. Extract the gross amount, net amount, and VAT/tax amount
2. Record the VAT rate if printed (e.g., 20%, 21%, 19%)
3. If multiple VAT rates are shown (common in some countries — e.g., food at reduced rate, alcohol at standard rate), record each separately:

```json
{
  "tax_breakdown": [
    { "description": "VAT 10% (food)", "rate": 10, "net": 52.00, "tax": 5.20 },
    { "description": "VAT 21% (alcohol)", "rate": 21, "net": 18.50, "tax": 3.89 }
  ],
  "tax_total": 9.09
}
```

### Processing Steps

#### Step 1: Extract and Structure

Read the receipt image. Extract all visible fields into structured data. If the receipt is crumpled, faded, or partially obscured, extract what is possible and flag uncertain fields.

#### Step 2: Associate with Trip

1. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/booked-trips.json` (or check Claude's memory as fallback)
2. Match by date — find a trip whose date range covers the receipt date
3. If the user is currently on a trip, default to that trip
4. If multiple trips could match, ask: "Is this from your Barcelona trip (15-20 May) or your London trip (22-25 May)?"
5. If no trip matches, log as a standalone expense and ask if a new trip should be created

#### Step 3: Log to Expense File

Use the **Read tool** to load `${CLAUDE_PLUGIN_DATA}/expense-log.json`, then append the expense and use the **Write tool** to save the updated file:

```json
{
  "expenses": [
    {
      "id": "exp_20260516_001",
      "trip_id": "barcelona_may_2026",
      "date": "2026-05-16",
      "time": "19:42",
      "vendor": "The Wolseley",
      "category": "food",
      "amount": 87.50,
      "currency": "EUR",
      "home_currency_amount": 74.80,
      "home_currency": "GBP",
      "exchange_rate": 0.855,
      "items": [
        { "description": "Steak Frites", "qty": 2, "unit_price": 24.50, "line_total": 49.00 },
        { "description": "House Wine 750ml", "qty": 1, "unit_price": 18.50, "line_total": 18.50 },
        { "description": "Crema Catalana", "qty": 2, "unit_price": 7.50, "line_total": 15.00 }
      ],
      "subtotal": 82.50,
      "service_charge": 0,
      "tip": 5.00,
      "tax_breakdown": [
        { "description": "IVA 10%", "rate": 10, "net": 64.00, "tax": 6.40 },
        { "description": "IVA 21%", "rate": 21, "net": 18.50, "tax": 3.89 }
      ],
      "tax_total": 10.29,
      "payment_method": "Visa ending 4821",
      "notes": "",
      "scanned_at": "2026-05-16T20:15:00Z",
      "image_quality": "good",
      "confidence": "high"
    }
  ]
}
```

#### Step 4: Budget Tracking

After logging the expense, calculate and display the running total for the associated trip:

1. Sum all expenses for this trip from the expense log
2. Check if the user set a budget for this trip (stored in the trip record under `budget`)
3. Display the summary:

```
Receipt logged: The Wolseley — €87.50 (food)

Your Barcelona trip so far: €1,247.30 across 8 expenses.
Budget was €1,500 — 83% spent with 3 days remaining.

Breakdown:
  Accommodation: €0 (covered by booking)
  Food & Drink:  €412.80 (33%)
  Transport:     €187.50 (15%)
  Activities:    €285.00 (23%)
  Shopping:      €362.00 (29%)
```

If the trip is approaching or over budget:

```
Budget alert: You've spent €1,480 of your €1,500 budget (99%) with 2 days remaining.
At your current daily rate of €249/day, you may want to adjust your spending.
```

#### Step 5: Confirm to User

```
Expense logged.

Vendor: The Wolseley
Date: 16 May 2026, 19:42
Amount: €87.50 (approx. £74.80)
Category: Food & Drink
Trip: Barcelona (15-20 May 2026)

Items:
  2x Steak Frites          €49.00
  1x House Wine 750ml      €18.50
  2x Crema Catalana         €15.00
  Tip                        €5.00
  Total                    €87.50

VAT: €10.29 (mixed rates: 10% on food, 21% on alcohol)
```

---

## 3. Booking Confirmation Scanning

### Extraction Fields

Same fields as the booking-detection skill, but extracted from images/screenshots rather than email text:

#### Flight Booking Confirmations

| Field | Required |
|---|---|
| Booking reference | Yes |
| Airline | Yes |
| Flight number(s) | Yes |
| Route (origin + destination) | Yes |
| Departure date and time | Yes |
| Return date and time | If round trip |
| Passenger name(s) | Yes |
| Total price | Yes |
| Currency | Yes |
| Fare class / cabin | If visible |
| Baggage allowance | If visible |
| Seat selection | If visible |

#### Accommodation Booking Confirmations

| Field | Required |
|---|---|
| Booking reference | Yes |
| Property name | Yes |
| Address | If visible |
| Platform (Booking.com, Airbnb, etc.) | Yes (inferred from branding) |
| Check-in date and time | Yes |
| Check-out date and time | Yes |
| Number of guests | If visible |
| Room type | If visible |
| Rate per night | If visible |
| Total price | Yes |
| Currency | Yes |
| Cancellation policy | If visible |
| Host / contact info | If visible |

### Processing Steps

#### Step 1: Extract All Booking Details

Use Claude's vision to read the confirmation screenshot or photo. Detect the platform from logos, colour schemes, and layout (e.g., Booking.com's blue, Airbnb's coral, Ryanair's blue/yellow).

#### Step 2: Match or Create Trip

1. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/booked-trips.json` (or check Claude's memory as fallback)
2. Match by destination + dates (within +/- 1 day tolerance)
3. If match found: update the trip with booking details
4. If no match: create a new trip entry with status `booked`

#### Step 3: Save Booking Data (Dual Write)

Update `travel_past_trips` with the full booking data, using the same schema as the booking-detection skill.

#### Step 4: Trigger Price Reshop Monitoring

1. Invoke the price-reshop system to begin monitoring for better prices on the same route/property
2. Use the booked price as the baseline
3. Confirm: "I've set up price reshop monitoring. If the same flight/hotel drops below your booked price, I'll let you know so you can rebook or request a price match."

#### Step 5: Update Trip Itinerary

Add the booking to the trip's itinerary timeline:

```json
{
  "itinerary": [
    {
      "type": "flight",
      "datetime": "2026-05-15T14:50:00",
      "description": "BA 2765 LHR to BCN",
      "booking_ref": "ABC123",
      "status": "confirmed"
    },
    {
      "type": "accommodation_checkin",
      "datetime": "2026-05-15T15:00:00",
      "description": "Hotel Arts Barcelona — check-in",
      "booking_ref": "HTL987654",
      "status": "confirmed"
    }
  ]
}
```

#### Step 6: Confirm to User

```
Booking confirmation scanned.

Hotel Arts Barcelona
Check-in: 15 May 2026, 15:00
Check-out: 20 May 2026, 11:00
5 nights at €165/night — total €825
Booking ref: HTL987654 (via Booking.com)

Added to your Barcelona trip (15-20 May). Price reshop monitoring is active — I'll alert you if the rate drops.
```

---

## 4. Hotel Bill Scanning

### Extraction Fields

| Field | Example | Required |
|---|---|---|
| Hotel name | `Hotel Arts Barcelona` | Yes |
| Folio / invoice number | `F-2026-4821` | If visible |
| Guest name | `Mr John Smith` | Yes |
| Room number | `1204` | If visible |
| Check-in date | `2026-05-15` | Yes |
| Check-out date | `2026-05-20` | Yes |
| Number of nights | `5` | Yes |
| Room rate per night | `189.00` | Yes |
| Room charges total | `945.00` | Yes |
| Currency | `EUR` | Yes |
| Itemised charges | Array (see below) | Yes |
| Taxes | See tax breakdown | Yes |
| Grand total | `1,087.45` | Yes |
| Payment method | `Visa ending 4821` | If visible |
| Balance due | `0.00` | If visible |

### Itemised Charge Categories

Extract and categorise each line item on the hotel bill:

| Category | Examples |
|---|---|
| Room charges | Nightly room rate, upgrade charges |
| Food & Beverage | Restaurant charges, room service, minibar |
| Bar | Bar tabs charged to room |
| Parking | Valet parking, self-parking |
| Spa & Wellness | Spa treatments, gym day pass |
| Laundry | Laundry, dry cleaning, pressing |
| Communication | Phone calls, WiFi surcharge |
| Resort / Facility fees | Resort fee, destination fee, amenity fee |
| Taxes | City/tourist tax, VAT, occupancy tax, service charge |
| Other | Late checkout fee, damage charges, miscellaneous |

### Hotel Bill Data Structure

```json
{
  "hotel_bill": {
    "hotel_name": "Hotel Arts Barcelona",
    "folio_number": "F-2026-4821",
    "guest_name": "Mr John Smith",
    "room_number": "1204",
    "check_in": "2026-05-15",
    "check_out": "2026-05-20",
    "nights": 5,
    "currency": "EUR",
    "line_items": [
      { "date": "2026-05-15", "category": "room", "description": "Deluxe Sea View Room", "amount": 189.00 },
      { "date": "2026-05-16", "category": "room", "description": "Deluxe Sea View Room", "amount": 189.00 },
      { "date": "2026-05-17", "category": "room", "description": "Deluxe Sea View Room", "amount": 189.00 },
      { "date": "2026-05-18", "category": "room", "description": "Deluxe Sea View Room", "amount": 189.00 },
      { "date": "2026-05-19", "category": "room", "description": "Deluxe Sea View Room", "amount": 189.00 },
      { "date": "2026-05-16", "category": "minibar", "description": "Minibar — 2x beer, 1x wine", "amount": 28.50 },
      { "date": "2026-05-17", "category": "food", "description": "Room Service — Club Sandwich", "amount": 22.00 },
      { "date": "2026-05-18", "category": "parking", "description": "Valet Parking", "amount": 35.00 },
      { "date": "2026-05-15", "category": "tax", "description": "Tourist Tax (per person/night)", "amount": 6.75 },
      { "date": "2026-05-16", "category": "tax", "description": "Tourist Tax (per person/night)", "amount": 6.75 },
      { "date": "2026-05-17", "category": "tax", "description": "Tourist Tax (per person/night)", "amount": 6.75 },
      { "date": "2026-05-18", "category": "tax", "description": "Tourist Tax (per person/night)", "amount": 6.75 },
      { "date": "2026-05-19", "category": "tax", "description": "Tourist Tax (per person/night)", "amount": 6.75 },
      { "date": "2026-05-20", "category": "resort_fee", "description": "Facility Fee", "amount": 15.00 }
    ],
    "summary": {
      "room_charges": 945.00,
      "food_and_beverage": 50.50,
      "parking": 35.00,
      "taxes_and_fees": 48.75,
      "grand_total": 1079.25
    },
    "payment": {
      "method": "Visa ending 4821",
      "balance_due": 0.00
    },
    "scanned_at": "2026-05-20T12:00:00Z"
  }
}
```

### Overcharge Detection

After extracting the hotel bill, compare against the booked rate:

1. Recall the trip record from `travel_past_trips`
2. Find the accommodation booking for this hotel
3. Compare the charged nightly rate against the booked nightly rate

#### Rate Discrepancy Check

```
if charged_rate_per_night > booked_rate_per_night:
    difference = charged_rate_per_night - booked_rate_per_night
    total_overcharge = difference * nights
    flag as potential overcharge
```

#### Overcharge Alert

```
Rate discrepancy detected.

Your hotel charged €189/night but your booking confirmation shows €165/night.
Over 5 nights, that's an overcharge of €120.00.

This may be worth querying at the front desk or with the booking platform. Common causes:
- Rate changed after booking (should be honoured at booked rate)
- Room upgrade applied without consent
- Weekend/peak supplement added incorrectly
- City tax included in room rate on bill but excluded from booking rate

Would you like me to draft a message to the hotel or the booking platform?
```

#### Fee Audit

Also flag unexpected or unusually high charges:

- Resort/facility fees not mentioned in the booking confirmation
- Minibar charges the guest may want to dispute
- Charges on dates outside the stay period
- Duplicate charges (same item charged twice)

```
I noticed a few items that may be worth checking:
- Facility Fee (€15.00) — this was not mentioned in your Booking.com confirmation
- Minibar charge on 16 May (€28.50) — just flagging in case this wasn't you
```

### Processing Steps

#### Step 1: Extract and Structure

Read the hotel bill image. Hotel bills vary widely in format — look for the folio header, itemised charge list, subtotals, tax section, and grand total.

#### Step 2: Run Overcharge Detection

Compare against booked rate and flag any discrepancies.

#### Step 3: Log Expenses

Use the **Read tool** to load `${CLAUDE_PLUGIN_DATA}/expense-log.json`, then log each non-room-charge line item as an individual expense, categorised appropriately. Room charges are logged as a single accommodation expense. Use the **Write tool** to save the updated file.

#### Step 4: Confirm to User

```
Hotel bill scanned: Hotel Arts Barcelona

5 nights (15-20 May 2026), Room 1204

Room charges:    €945.00 (€189.00/night)
Food & Beverage:  €50.50
Parking:          €35.00
Taxes & Fees:     €48.75
─────────────────────────
Grand Total:   €1,079.25

Rate check: Your booking was €165/night but you were charged €189/night.
Potential overcharge of €120.00 across 5 nights. See above for details.

All charges have been logged to your Barcelona trip expenses.
```

---

## 5. Expense Report Generation

Triggered by `/travel-expenses [trip-name]` or "generate expense report for [trip]".

### Command Syntax

```
/travel-expenses [trip-name]          — Full expense report for the named trip
/travel-expenses [trip-name] --daily  — Include per-day breakdown
/travel-expenses [trip-name] --vat    — Include detailed VAT/tax breakdown
/travel-expenses                      — List all trips with expenses, prompt to select
```

### Report Generation Steps

1. Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/booked-trips.json` (or check Claude's memory as fallback) to resolve the trip name
2. Use the **Read tool** to load all expenses for that trip from `${CLAUDE_PLUGIN_DATA}/expense-log.json`
3. Group by category
4. Calculate totals, averages, and budget comparison
5. Format as markdown

### Full Expense Report Template

```markdown
# Expense Report: Barcelona Trip
**Dates:** 15-20 May 2026 (5 nights)
**Travellers:** 2

## Summary

| Category | Amount (EUR) | Amount (GBP) | % of Total |
|---|---:|---:|---:|
| Accommodation | €945.00 | £807.98 | 48.2% |
| Food & Drink | €462.80 | £395.69 | 23.6% |
| Transport | €187.50 | £160.31 | 9.6% |
| Activities | €285.00 | £243.68 | 14.5% |
| Shopping | €62.00 | £53.01 | 3.2% |
| Communication | €15.00 | £12.83 | 0.8% |
| Other | €2.50 | £2.14 | 0.1% |
| **Total** | **€1,959.80** | **£1,675.63** | **100%** |

**Budget:** €1,800.00
**Actual:** €1,959.80
**Over/Under:** €159.80 over budget (108.9%)

## Expenses by Category

### Accommodation (€945.00)
| Date | Vendor | Description | Amount |
|---|---|---|---:|
| 15-20 May | Hotel Arts Barcelona | 5 nights Deluxe Sea View | €945.00 |

### Food & Drink (€462.80)
| Date | Vendor | Description | Amount |
|---|---|---|---:|
| 15 May | La Boqueria | Market lunch | €18.50 |
| 15 May | El Xampanyet | Dinner | €67.30 |
| 16 May | Hotel Arts | Room Service — Club Sandwich | €22.00 |
| 16 May | Hotel Arts | Minibar | €28.50 |
| 16 May | The Wolseley | Dinner | €87.50 |
| 17 May | Cerveceria Catalana | Tapas lunch | €42.00 |
| 17 May | Can Culleretes | Dinner | €78.00 |
| 18 May | La Paradeta | Seafood dinner | €72.00 |
| 19 May | Tickets Bar | Dinner | €47.00 |

### Transport (€187.50)
| Date | Vendor | Description | Amount |
|---|---|---|---:|
| 15 May | Aerobús | Airport to city | €14.50 |
| 15 May | TMB | Metro day pass x2 | €22.00 |
| 16 May | Uber | City ride | €12.50 |
| 17 May | Hotel Arts | Valet Parking | €35.00 |
| 18 May | Renfe | Day trip train | €24.00 |
| 20 May | Taxi | Hotel to airport | €42.00 |
| Various | TMB | Metro rides | €37.50 |

### Activities (€285.00)
| Date | Vendor | Description | Amount |
|---|---|---|---:|
| 16 May | Sagrada Familia | Entry x2 | €52.00 |
| 17 May | Park Guell | Entry x2 | €20.00 |
| 17 May | Casa Batllo | Entry x2 | €70.00 |
| 18 May | Montserrat | Cable car + monastery | €48.00 |
| 19 May | Picasso Museum | Entry x2 | €24.00 |
| 19 May | Gothic Quarter | Walking tour x2 | €40.00 |
| 19 May | Flamenco show | Tickets x2 | €31.00 |

### Shopping (€62.00)
| Date | Vendor | Description | Amount |
|---|---|---|---:|
| 17 May | Mercat dels Encants | Souvenirs | €22.00 |
| 19 May | El Corte Ingles | Gifts | €40.00 |

## Tax / VAT Summary

| Tax Type | Net Amount | Rate | Tax Amount |
|---|---:|---|---:|
| IVA (reduced — food) | €312.40 | 10% | €31.24 |
| IVA (standard — services) | €285.00 | 21% | €59.85 |
| IVA (standard — alcohol) | €68.50 | 21% | €14.39 |
| Tourist Tax | — | — | €33.75 |
| **Total Tax** | | | **€139.23** |

## Per-Day Breakdown

| Date | Accommodation | Food | Transport | Activities | Shopping | Daily Total |
|---|---:|---:|---:|---:|---:|---:|
| 15 May | €189.00 | €85.80 | €36.50 | — | — | €311.30 |
| 16 May | €189.00 | €138.00 | €12.50 | €52.00 | — | €391.50 |
| 17 May | €189.00 | €42.00 | €35.00 | €90.00 | €22.00 | €378.00 |
| 18 May | €189.00 | €72.00 | €24.00 | €48.00 | — | €333.00 |
| 19 May | €189.00 | €47.00 | €37.50 | €95.00 | €40.00 | €408.50 |
| 20 May | — | — | €42.00 | — | — | €42.00 |
| **Total** | **€945.00** | **€384.80** | **€187.50** | **€285.00** | **€62.00** | **€1,864.30** |

*Note: Grand total includes €48.75 in hotel taxes/fees and €46.75 in other charges not shown in daily breakdown.*

## Comparison to Budget

- **Budget:** €1,800.00
- **Actual:** €1,959.80
- **Variance:** €159.80 over (8.9%)
- **Largest overspend category:** Food & Drink (€462.80 — expected ~€350 based on past trips)
- **Per-person total:** €979.90
- **Per-person per-night:** €195.98

---
*Generated by AI Heroes Travel Agent v2.1 — document-scanner skill*
*Report date: 20 May 2026*
```

### Saving the Report

After generating the report:
1. Display it in the conversation for immediate review
2. Offer to save: "Want me to save this as a file you can copy or share?"
3. If yes, use the **Write tool** to save to `${CLAUDE_PLUGIN_DATA}/reports/expense-report-[trip-name]-[date].md`

---

## 6. Gmail Integration

When the Gmail connector is available (check for `mcp__claude_ai_Gmail__gmail_search_messages` tool availability):

### Auto-Scan Offer

When the user mentions a trip or asks about bookings, proactively search Gmail for relevant confirmations:

1. Search Gmail for booking confirmation emails matching the trip dates and destination
2. Use search queries like: `subject:(booking confirmed OR reservation OR e-ticket OR itinerary) after:YYYY/MM/DD before:YYYY/MM/DD`
3. Present findings to the user:

```
I found 3 booking-related emails that might be relevant to your Barcelona trip:

1. Booking.com — Hotel Arts Barcelona, confirmed 2 Apr
2. British Airways — Flight confirmation BA 2765, confirmed 28 Mar
3. GetYourGuide — Sagrada Familia tickets, confirmed 1 Apr

Want me to scan and import these?
```

4. If the user agrees, extract booking details from each email and process them as booking confirmations (see section 3 above)

### Duplicate Detection

Before importing from Gmail, check if the booking already exists in `travel_past_trips` by matching booking references. Skip duplicates and report: "The BA flight and Hotel Arts bookings are already in your trip record. I'll add the Sagrada Familia tickets."

---

## 7. Predictive Spending Patterns

As the expense log grows across multiple trips, surface insights when relevant.

### When to Surface Patterns

- When the user is planning a new trip (budgeting phase)
- When the user asks "how much will this trip cost?"
- When the user runs `/travel-expenses` with the `--insights` flag
- When a trip's spending significantly deviates from historical patterns

### Pattern Analysis

#### Accommodation Averages

```
Based on your last [N] trips:
- European city hotels: average €[X]/night
- European vacation rentals: average €[X]/night
- UK hotels: average £[X]/night
```

#### Trip Cost Prediction

```
Based on your travel history, a [N]-night trip to [destination type] typically costs:

  Accommodation: €[X] ([Y]% of total)
  Food & Drink:  €[X] ([Y]% of total)
  Transport:     €[X] ([Y]% of total)
  Activities:    €[X] ([Y]% of total)
  Shopping:      €[X] ([Y]% of total)
  Other:         €[X] ([Y]% of total)
  ─────────────────────────────────────
  Estimated total: €[X]
  Suggested budget (with 10% buffer): €[X]
```

#### Category Insights

```
Your spending patterns across [N] trips:
- Accommodation: [X]% of total spend (trend: stable/increasing/decreasing)
- Food & Drink: [X]% of total spend
- Transport: [X]% of total spend
- Activities: [X]% of total spend
- Shopping: [X]% of total spend

Notable: Your food spending has increased ~15% over the last 3 trips.
```

#### Per-Destination Intelligence

When enough data exists for a specific destination:

```
You've visited Barcelona [N] times. Average trip cost: €[X] over [Y] nights.
Your usual spending breakdown for Barcelona: ...
```

### Data Requirements

- Minimum 3 completed trips before surfacing cross-trip patterns
- Minimum 2 visits to the same destination before surfacing destination-specific patterns
- Always caveat predictions: "Based on your past trips — actual costs will vary depending on your plans."

---

## 8. Expense Log Data Structure

The master expense log lives at `${CLAUDE_PLUGIN_DATA}/expense-log.json`. Always use the **Read tool** to load it and the **Write tool** to save updates:

```json
{
  "version": "1.0",
  "home_currency": "GBP",
  "expenses": [
    {
      "id": "exp_[YYYYMMDD]_[NNN]",
      "trip_id": "[destination]_[month]_[year]",
      "date": "YYYY-MM-DD",
      "time": "HH:MM",
      "vendor": "string",
      "category": "transport|accommodation|food|activities|shopping|communication|health|other",
      "description": "string (brief note if vendor name alone is insufficient)",
      "amount": 0.00,
      "currency": "EUR",
      "home_currency_amount": 0.00,
      "home_currency": "GBP",
      "exchange_rate": 0.855,
      "items": [
        {
          "description": "string",
          "qty": 1,
          "unit_price": 0.00,
          "line_total": 0.00
        }
      ],
      "subtotal": 0.00,
      "service_charge": 0.00,
      "tip": 0.00,
      "tax_breakdown": [
        {
          "description": "string",
          "rate": 0,
          "net": 0.00,
          "tax": 0.00
        }
      ],
      "tax_total": 0.00,
      "payment_method": "string",
      "source": "scan|manual|gmail",
      "document_type": "receipt|boarding_pass|hotel_bill|booking_confirmation",
      "notes": "string",
      "scanned_at": "ISO 8601 timestamp",
      "image_quality": "good|fair|poor",
      "confidence": "high|medium|low",
      "flagged": false,
      "flag_reason": ""
    }
  ],
  "trip_budgets": {
    "[trip_id]": {
      "budget_amount": 0.00,
      "budget_currency": "EUR",
      "set_at": "ISO 8601 timestamp"
    }
  },
  "metadata": {
    "created_at": "ISO 8601 timestamp",
    "last_updated": "ISO 8601 timestamp",
    "total_expenses": 0,
    "total_trips_with_expenses": 0
  }
}
```

---

## 9. Alert and Message Templates

### Expense Logged (Standard)

```
Expense logged: [vendor] — [currency][amount] ([category])
Trip: [trip name] | Running total: [currency][total] ([N] expenses)
```

### Expense Logged (With Budget Warning)

```
Expense logged: [vendor] — [currency][amount] ([category])

Budget warning: [trip name] is at [X]% of budget ([currency][spent] of [currency][budget]).
[N] days remaining — average daily spend so far: [currency][daily_avg].
```

### Overcharge Detected

```
Potential overcharge detected on your [hotel name] bill.

Booked rate: [currency][booked_rate]/night
Charged rate: [currency][charged_rate]/night
Difference: [currency][difference]/night x [N] nights = [currency][total_overcharge]

This may be worth raising with the hotel or booking platform.
```

### Scan Quality Warning

```
I was able to read this [document type] but some details were unclear due to [image quality issue].
Uncertain fields are marked with [?]. Please verify:
- [field]: [extracted value] [?]
```

### Trip Spending Milestone

```
Spending update for your [trip name]:
You've now logged [N] expenses totalling [currency][total].
Largest expense: [vendor] ([currency][amount])
Most frequent category: [category] ([N] expenses)
```

---

## 10. Error Handling

| Scenario | Response |
|---|---|
| Image too blurry to read | "I can't read this image clearly enough to extract reliable data. Could you take another photo with better lighting and focus?" |
| Partial extraction only | Extract what is possible, list uncertain fields with `[?]` markers, ask user to confirm or fill gaps |
| Currency cannot be determined | "I can see the amount is 87.50 but I'm not sure of the currency. Is this in EUR, GBP, or something else?" |
| No trip match found | "I can't match this to an existing trip. Want me to create a new trip, or does this belong to [closest match]?" |
| Duplicate expense detected | "This looks similar to an expense I logged on [date] — [vendor] for [amount]. Is this a separate charge or the same one?" |
| Expense log file missing | Create a new expense log with the schema defined above |
| Non-travel document uploaded | "This doesn't appear to be a travel document. I can scan receipts, boarding passes, booking confirmations, and hotel bills. Is this one of those?" |

---

## Privacy and Data Handling

- All scanned document data is stored locally in `${CLAUDE_PLUGIN_DATA}/expense-log.json` and persistent memory
- No document images are retained after scanning — only the extracted structured data is stored
- No financial data is sent to third parties
- The user can delete any expense entry: "delete expense [id]" or "remove the last expense I logged"
- The user can export and clear all expense data: "export my expenses" / "clear expense log"
- Payment card numbers are stored only as last-four-digits (e.g., "Visa ending 4821"), never in full
