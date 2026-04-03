---
name: local-transit
description: "Local transit and last-mile connection advice using TfL MCP for London and web search for other cities. Provides station-to-accommodation connections, airport transfers, and in-city transport tips. Automatically active during trip planning."
user-invocable: false
---

# Local Transit & Last-Mile Connections

Provides local transport advice to connect the dots between arrival points (airports, train stations, ferry ports) and accommodation, and for getting around during the trip.

## When This Skill Activates

- After a train, flight, or ferry search — to provide the last-mile connection
- During trip planning — to include local transport in the itinerary
- When the user asks about getting around at a destination
- When providing airport/station transfer advice

## London: Use TfL MCP

For any London connection, use the `tfl` MCP server for live data:

1. **Query the TfL MCP** for journey planning between two points
2. Present the recommended route with line names, interchange stations, and estimated time
3. Note Oyster/contactless pricing and any peak/off-peak differences

### Common London Connections

Provide specific advice for major arrival points:

- **St Pancras International** (Eurostar arrivals): Northern line, Victoria line, Piccadilly line, Thameslink, and many bus routes. "Your Eurostar arrives at St Pancras. For [accommodation area], take the [line] ([X] stops, ~[Y] minutes)."
- **Heathrow Airport**: Elizabeth line (fastest to central London, ~30-40 min), Piccadilly line (~50-60 min, cheaper), Heathrow Express (15 min to Paddington, most expensive)
- **Gatwick Airport**: Gatwick Express (30 min to Victoria), Thameslink (to London Bridge, City, St Pancras)
- **London Liverpool Street** (trains from Stansted): Central line, Elizabeth line, Hammersmith & City line
- **Waterloo** (trains from Southampton cruise terminal): Northern line, Bakerloo line, Jubilee line

### London Tips
- Contactless payment works on all TfL services — no Oyster card needed
- Daily and weekly caps apply automatically
- Children under 11 travel free on TfL with a paying adult
- Avoid Zone 1 Tube at 08:00-09:30 and 17:00-18:30 if possible

## Other Cities: Web Search

For non-London destinations, use web search to research:

1. **Airport/station to accommodation** — what public transport options exist, taxi/ride-share estimates
2. **Getting around the city** — metro/bus system overview, tourist passes, taxi apps
3. **Key tips** — contactless acceptance, safety notes, operating hours

### Common Transit Advice by City

**Paris:**
- RER B from CDG airport to city centre (~35 min, ~€11.80)
- Métro covers all central Paris (Navigo Easy card or contactless)
- Gare du Nord (Eurostar) connects to Métro lines 4, 5 and RER B/D/E

**Amsterdam:**
- Thalys/Eurostar arrives at Amsterdam Centraal — trams, metro, and buses from there
- GVB day pass for unlimited tram/bus/metro
- OV-chipkaart or contactless for all public transport

**Rome:**
- Leonardo Express from Fiumicino to Roma Termini (32 min, ~€14)
- Metro has 3 lines (A, B, C) — covers main tourist areas
- Roma Termini connects to both Frecciarossa trains and metro

**New York:**
- Penn Station (Amtrak/Acela) is on the A/C/E, 1/2/3 subway lines
- MetroCard or OMNY contactless ($2.90 per ride, daily cap)
- JFK: AirTrain + subway (~$10.75 total) or taxi (flat rate $70 + tip to Manhattan)

**Tokyo:**
- Shinkansen arrives at Tokyo Station — JR lines, Metro, and Marunouchi line
- Suica/Pasmo IC card works on all trains, metros, buses, and convenience stores
- Japan Rail Pass for multi-city bullet train trips

## Output Template

When providing local transit advice, format as:

```
### 🚇 Getting There: [Arrival Point] → [Accommodation Area]

**Recommended:** [Transport mode] — [Line/route name]
**Route:** [Station A] → [interchange if any] → [Station B]
**Time:** ~[X] minutes
**Cost:** ~[price] per person [+ child policy note]
**Frequency:** Every [X] minutes
**Tips:** [One practical tip — e.g., "Buy tickets at the machine in the station, not from touts outside"]
```

For city-wide transport overview:

```
### 🚇 Getting Around [City]

**Main system:** [Metro/Tram/Bus name]
**Payment:** [Contactless / Card name / Tickets]
**Tourist pass:** [Name, price, what it covers]
**Taxis:** [App name — Uber/Bolt/local alternative]
**Children:** [Free under X / discount policy]
**Key tip:** [One useful thing — e.g., "The metro closes at midnight; night buses run 00:30-05:30"]
```

## Integration

- This skill is called by `trip-planner` during Step 5 (local transit connections)
- It runs after transport searches to provide last-mile advice
- For London, always attempt the TfL MCP first; fall back to web search if unavailable
- For all other cities, use web search directly
