---
name: transport-intelligence
description: "Proactive multi-modal transport mode selection engine. Reads user profile, Claude memory, and trip context to recommend the best transport mode (train, flight, ferry, coach) before any search. Automatically active before transport searches."
user-invocable: false
---

# Transport Intelligence

The proactive brain behind multi-modal transport recommendations. Before any transport search, this skill analyses the route, user profile, and trip context to recommend the best mode of travel — without the user needing to ask.

## Before Any Transport Search

1. **Read the user's travel profile** from `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
2. **Read Claude's memory** for any travel preferences or past feedback
3. **Detect trip context** from the conversation:
   - Solo / partner / family / group / work
   - Children travelling? What ages?
   - Luggage situation (light carry-on vs heavy bags)
4. **Analyse the route** against the embedded knowledge base below

## Proactive Mode Selection Logic

Apply these rules in order. The FIRST matching rule determines the lead recommendation:

### Rule 1: Train-Competitive Short Routes
- Route < 4 hours by train AND travelling with children → **Lead with train**, explain child benefits (free/discounted fares, no luggage fees, no airport hassle)
- Route < 3 hours by train AND city-centre to city-centre departure → **Lead with train**, highlight door-to-door time advantage

### Rule 2: Profile-Driven Selection
- User has "eco preference" or "comfort priority" or "environmental impact" in profile → **Lead with train/ferry** when available
- User has "budget priority" in profile → **Show cheapest mode first** (often coach/bus for short routes, budget airlines for long)
- User has "speed priority" in profile → **Show fastest door-to-door mode first**

### Rule 3: Known Train-Beating Routes
- Cross-channel routes (any UK ↔ EU city) → **Always show Eurostar** alongside flights
- Northeast Corridor US routes (NYC-DC, Boston-NYC, NYC-Philadelphia) → **Always show Acela** alongside flights
- Any route in the knowledge base below where train time ≤ flight door-to-door time → **Lead with train**

### Rule 4: Ferry Routes
- Greek island routes → **Show ferry** (often the only practical option)
- Baltic routes with overnight option → **Show overnight ferry** as accommodation+transport combo
- UK ↔ Netherlands/France with car → **Show ferry** (DFDS, P&O, Stena Line)

### Rule 5: No Strong Signal
- If no rule above matches → **Show top 2 modes side-by-side** with a comparison table
- Include: journey time, price range, luggage policy, child policy, door-to-door estimate

### Rule 6: Explicit User Request
- If the user explicitly asks for a specific mode (e.g., "find me flights") → **Search that mode only**
- Still mention alternatives briefly: "I've searched flights as requested. FYI, Eurostar does this route in 2h17 if you'd like me to check train prices too."

## Embedded Knowledge Base

### European Train-Competitive Routes

| Route | Operator | Train Time | Kids Policy | Luggage | Notes |
|---|---|---|---|---|---|
| London ↔ Paris | Eurostar | 2h 17m | Under 4 free (on lap) | Unlimited free | City centre to city centre, no luggage fees |
| London ↔ Brussels | Eurostar | 2h 01m | Under 4 free (on lap) | Unlimited free | |
| London ↔ Amsterdam | Eurostar | 3h 52m | Under 4 free (on lap) | Unlimited free | |
| Paris ↔ Lyon | TGV (SNCF) | 2h 00m | Under 4 free | Unlimited free | |
| Paris ↔ Marseille | TGV (SNCF) | 3h 20m | Under 4 free | Unlimited free | |
| Paris ↔ Brussels | Thalys/Eurostar | 1h 22m | Under 4 free | Unlimited free | |
| Berlin ↔ Munich | ICE (DB) | 4h 00m | Under 6 free; 6-14 FREE with adult | Unlimited free | |
| Berlin ↔ Hamburg | ICE (DB) | 1h 45m | Under 6 free; 6-14 FREE with adult | Unlimited free | |
| Milan ↔ Rome | Frecciarossa (Trenitalia) | 2h 55m | Under 4 free | Unlimited free | |
| Milan ↔ Venice | Frecciarossa (Trenitalia) | 2h 25m | Under 4 free | Unlimited free | |
| Madrid ↔ Barcelona | AVE (Renfe) | 2h 30m | Under 4 free | Unlimited free | |
| Zurich ↔ Milan | EC (SBB) | 3h 20m | Under 6 free; 6-16 half fare | Unlimited free | Scenic Gotthard route |
| Stockholm ↔ Gothenburg | SJ X2000 | 3h 00m | Under 6 free | Unlimited free | |
| Vienna ↔ Budapest | Railjet (ÖBB) | 2h 30m | Under 6 free | Unlimited free | |

### US Train-Competitive Routes

| Route | Operator | Train Time | Price Range | vs Flight (door-to-door) |
|---|---|---|---|---|
| NYC ↔ Washington DC | Amtrak Acela | 2h 45m | $40–250 | Flight $150–475 + 2h airport time. Train wins on total time. |
| Boston ↔ NYC | Amtrak Acela | 3h 25m | $49–250 | Nearly identical total door-to-door time |
| NYC ↔ Philadelphia | Amtrak Acela/NE Regional | 1h 10m | $20–150 | No flight market — train is the default |
| LA ↔ San Diego | Amtrak Pacific Surfliner | 2h 47m | $10–35 | Flight $80–200, but airport adds 2h+ |
| Miami ↔ Orlando | Brightline | 3h 30m | $24–80 | Flight $80–200, similar total time |
| Chicago ↔ Milwaukee | Amtrak Hiawatha | 1h 30m | $19–37 | No meaningful flight market |
| San Jose ↔ San Francisco | Caltrain | 1h 10m | $7–13 | No flight market |

### Child & Family Policies Comparison

| Operator | Free Under Age | Discount Ages | Luggage Policy |
|---|---|---|---|
| Eurostar | 4 (on lap) | 4–11: ~30% off | Unlimited free bags |
| UK National Rail | 5 | 5–15: 50% off (with adult) | Unlimited free |
| Deutsche Bahn (DB) | 6 | 6–14: FREE when travelling with parent/grandparent | Unlimited free |
| SNCF (TGV) | 4 | 4–11: ~30% off | Unlimited free |
| Trenitalia | 4 | 4–11: ~50% off | Unlimited free |
| Renfe (AVE) | 4 | 4–13: ~40% off | Unlimited free |
| Amtrak | 2 (on lap) | 2–12: 50% off | 5 free items (2 carry-on + 2 checked + 1 personal) |
| Most airlines | 2 (on lap, ~10% fare) | 2+: full fare | £25–45 per checked bag |

### Scenic & Experience Routes

Suggest these when the trip context allows time and the user values experiences:

| Route | Operator | Duration | Highlight |
|---|---|---|---|
| California Zephyr | Amtrak | 52.5h (Chicago → SF) | Rocky Mountains, Sierra Nevada |
| Empire Builder | Amtrak | 46h (Chicago → Seattle) | Glacier National Park views |
| Coast Starlight | Amtrak | 35h (LA → Seattle) | Pacific coastline |
| Auto Train | Amtrak | 17.5h (Lorton VA → Sanford FL) | Bring your car, overnight |
| Glacier Express | SBB/MGB | 8h (Zermatt → St. Moritz) | Swiss Alps, 291 bridges, 91 tunnels |
| Bergen Railway | Vy | 7h (Oslo → Bergen) | Norwegian fjords and mountains |
| West Highland Line | ScotRail | 5h 10m (Glasgow → Mallaig) | Scottish Highlands, Glenfinnan Viaduct |
| Bernina Express | RhB | 4h (Chur → Tirano) | UNESCO World Heritage route |

**USA Rail Pass:** $499 for 10 segments in 30 days — mention when users are doing multi-city US trips.

### Ferry Routes Worth Knowing

| Route | Operator(s) | Duration | Notes |
|---|---|---|---|
| Greek islands | Multiple (Blue Star, Hellenic Seaways) | Varies | Often the only practical option |
| Stockholm ↔ Helsinki | Viking Line, Tallink Silja | 16h overnight | Combines transport + accommodation |
| Copenhagen ↔ Oslo | DFDS | 17h overnight | Combines transport + accommodation |
| Harwich ↔ Hook of Holland | Stena Line | 6h 45m / overnight | Car-friendly UK ↔ Netherlands |
| Dover ↔ Calais | P&O, DFDS | 1h 30m | Cheapest UK ↔ France with car |
| Portsmouth ↔ Bilbao | Brittany Ferries | 24–32h | UK ↔ Spain with car |
| Split ↔ Hvar/Brač | Jadrolinija, Krilo | 1–2h | Croatian island hopping |
| Naples ↔ Capri | Caremar, SNAV | 50m–1h 20m | |
| Barcelona ↔ Mallorca | Baleària, Trasmed | 5–7.5h | |

## Recommendation Template

When presenting a train or alternative transport recommendation, use this format:

```
### 🚂 Recommended: [Operator] [Route]

**Journey:** [time] | [departure station] → [arrival station] (city centre to city centre)
**Child policy:** [Child name/ages] rides free / [X]% off (vs full fare on airlines)
**Luggage:** Unlimited free bags (vs £[X] per bag on airlines)
**Door-to-door:** ~[total time including getting to/from station] vs ~[flight door-to-door including airport]
**Price range:** [range] per person
**Booking:** [operator website or search link]

> [One sentence on why this suits them based on profile/context]
```

For side-by-side comparisons:

```
### Transport Comparison: [Origin] → [Destination]

| | ✈️ Flight | 🚂 Train | 🚢 Ferry |
|---|---|---|---|
| Journey time | [X]h | [X]h | [X]h |
| Door-to-door | ~[X]h | ~[X]h | ~[X]h |
| Price range | [range] | [range] | [range] |
| Child (age X) | Full fare | [Free/discount] | [Free/discount] |
| Luggage | £[X]/bag | Unlimited free | Unlimited free |
| Departs from | [Airport] | [Station] | [Port] |
| Arrives at | [Airport] | [Station] | [Port] |

**Recommended:** [Mode] — [one sentence why]
```

## Integration with Other Skills

- This skill runs BEFORE `train-search`, `flight-search`, or `ferry-search`
- It decides which search skill(s) to invoke and in what order
- The `trip-planner` skill calls this as its first transport step
- Results feed into the `research-pipeline` for live verification
