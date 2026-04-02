---
name: guardrails
description: "Anti-hallucination guardrails for travel data. Enforces live verification, price timestamping, source attribution, and cross-checking rules. Automatically active during any travel search or recommendation."
user-invocable: false
---

# Anti-Hallucination Guardrails

These rules are absolute and override all other behaviour when presenting travel information.

## Price Rules

1. **Every price must be retrieved live** in this conversation session from an MCP tool or verified web source.
2. **Every price must be timestamped**: "Price as of [HH:MM today] — verify at checkout."
3. **Never round prices.** If the MCP returns £127.43, show £127.43 — not "around £130" or "about £127".
4. **Never estimate prices** from similar listings, historical data, or general knowledge.
5. **Never convert currencies** unless the user explicitly asks. Show prices in the currency returned by the source.

## Availability Rules

1. **Only state availability if confirmed live** by an MCP tool query in this session.
2. If availability cannot be confirmed, say: "I couldn't confirm availability for these dates — check the listing directly: [link]"
3. **Never assume** a listing is available because it appeared in search results. Search results show general listings; availability must be checked for specific dates.

## Source Attribution

1. **Tag every fact** with its source:
   - "Via Airbnb MCP" — data from the airbnb MCP server
   - "Via Google Flights" — data from the google-flights MCP server
   - "Via web search" — data found through web search tools
2. When cross-checking produces **different prices** from different sources:
   - If discrepancy is ≤5%: show the lower price, note both sources
   - If discrepancy is >5%: show BOTH prices with sources and let the user decide
3. **Never merge data from different sources** without attribution. If Airbnb says £120/night and web search says £140/night, present both.

## Booking Link Rules

1. **Every recommendation must include a direct booking link.**
2. Links must be to the actual listing page, not a search results page.
3. If a link cannot be verified as working, note: "Link retrieved from [source] — verify it loads correctly."
4. **Never construct booking URLs manually.** Only use URLs returned by MCP tools or found on actual booking sites.

## Review Score Rules

1. Only show review scores retrieved from the source platform.
2. **Never average** scores across platforms. Airbnb uses /5, Booking.com uses /10 — keep them separate.
3. Minimum thresholds for recommendations:
   - Airbnb: 4.3+/5 with 10+ reviews
   - Booking.com: 7.5+/10 with 20+ reviews
4. If a listing doesn't meet thresholds, it can still be shown but must be flagged: "Note: Below usual quality threshold (X/5, Y reviews)"

## What To Do When Data Is Missing

| Situation | Action |
|-----------|--------|
| Price not available | "Price not available — check directly: [link]" |
| Availability unknown | "Availability unconfirmed for your dates — check: [link]" |
| Reviews not found | Show listing without score, note "Reviews not available" |
| Photos not accessible | Describe based on listing text, note "Photos not loaded" |
| Link broken | Exclude from recommendations, note "Listing excluded — link not working" |
| Only 1 source available | Show result, note "Single source — not cross-checked" |
| Fewer than 3 results pass quality | Show what passed, tell user: "Only [N] listings met quality standards. Want me to widen the search?" |

## Forbidden Phrases

Never say any of these:
- "Prices typically range from..."
- "You can expect to pay around..."
- "Based on similar listings..."
- "Prices are usually..."
- "I'd estimate..."
- "It should cost approximately..."
- "Comparable properties go for..."

Instead, always use:
- "The listed price is [exact amount] as of [time]"
- "I found [exact amount] via [source] at [time]"
- "I couldn't retrieve a live price — here's the link to check yourself"
