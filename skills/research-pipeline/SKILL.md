---
name: research-pipeline
description: "3-stage research quality pipeline for travel searches. Ensures live retrieval, cross-checking, and quality filtering before presenting results. Automatically active during accommodation and flight searches."
user-invocable: false
---

# Research Quality Pipeline

Every travel search follows this 3-stage pipeline before results are presented to the user.

## Stage 1: Live Data Retrieval

Query MCP tools for the user's search criteria.

**For accommodation:**
1. Query the `airbnb` MCP server with location, dates, guests, and budget filters
2. For each result, capture: listing name, neighbourhood, price per night, total price, review score, review count, amenities, guest capacity, bedrooms, booking URL
3. Validate completeness — flag any result missing required fields

**For flights:**
1. Query the `google-flights` MCP server with origin, destination, dates, and preferences
2. For each result, capture: airline, departure/arrival times, duration, stops, price, booking class, booking URL
3. Validate completeness — flag any result missing required fields

**Required fields for accommodation:**
- [ ] Listing name
- [ ] Location/neighbourhood
- [ ] Price per night (exact)
- [ ] Total price including fees (exact)
- [ ] Review score and count
- [ ] Guest capacity
- [ ] Booking URL

**Required fields for flights:**
- [ ] Airline
- [ ] Departure and arrival times
- [ ] Number of stops
- [ ] Total price (exact)
- [ ] Booking URL or reference

If a result is missing more than 2 required fields, attempt to re-query for that specific listing. If still incomplete after retry, move to Stage 2 with a note.

## Stage 2: Cross-Check

For the top results from Stage 1, verify on a second source:

**Accommodation cross-check:**
- If Stage 1 used Airbnb MCP → cross-check prices via web search for the same property on Booking.com or the property's direct website
- If found on a second source:
  - Price difference ≤5%: proceed with the lower price, note both sources
  - Price difference >5%: flag both prices and let the user see both
- If no second source found: proceed but note "Single source — not cross-checked"

**Flight cross-check:**
- If Stage 1 used Google Flights MCP → cross-check via web search on Skyscanner or the airline's direct website
- Same discrepancy rules as accommodation

## Stage 3: Quality Filter

Apply these filters in order:

### 3a. Minimum Quality Standards
- Airbnb: ≥4.3/5 rating with ≥10 reviews
- Booking.com: ≥7.5/10 rating with ≥20 reviews
- Flights: prefer direct flights if within 20% price of connections

### 3b. Preference Scoring
Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel-profile.json` to load the user's profile (follow the persistent-memory fallback chain: file first, then Claude's memory, then ask the user) and score:

```
Base score = listing quality (reviews, completeness)
+ 15 for each matching "prefer" tag
- 50 for each matching "avoid" tag
```

Exclude any result with score < 0.

### 3c. Final Selection
- Sort by preference score (descending)
- Take top 3 results
- If fewer than 3 pass all filters, present what passed and offer to widen search

## Pipeline Status Reporting

While running the pipeline, keep the user informed:

```
Searching [source]... found [N] results.
Cross-checking prices... [done/skipped]
Applying your preferences... [N] listings match.
```

## Error Recovery

| Error | Recovery |
|-------|----------|
| MCP server timeout | Retry once. If still fails, note: "The [source] search timed out. Try again or I can search an alternative source." |
| MCP server not connected | Tell user: "The [source] MCP server isn't connected. Check your MCP setup or I can search via web instead." |
| Zero results | Ask user to widen search: relax dates, increase budget, expand area |
| All results below quality threshold | Show the best available with quality warnings, ask if user wants to lower standards |
