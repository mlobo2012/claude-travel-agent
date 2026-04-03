# AI Heroes Travel Agent

An intelligent travel plugin for **Claude Code** and **Claude Cowork** that plans trips end-to-end — flights, trains, ferries, accommodation, and activities — with live-verified prices, proactive transport recommendations, loyalty programme intelligence, and reasoning transparency on every suggestion. It learns your preferences over time and applies them to every search.

Built by [AI Heroes](https://www.ai-heroes.co).

---

## Key Features

- **Proactive transport intelligence** — Recommends trains, flights, or ferries based on route analysis, your profile, and trip context. Doesn't default to flights.
- **Deep booking links** — Every result links directly to the specific option with dates, passengers, and route pre-filled. Covers 15+ airlines, 8+ rail operators, 10+ ferry operators, and major hotel chains.
- **Reasoning transparency** — Every recommendation includes a "Why this?" block explaining exactly which preferences, past trips, and loyalty factors influenced the choice.
- **Loyalty programme tracking** — Auto-applies membership numbers, tracks tier progress, flags earning opportunities, and factors loyalty into every tradeoff.
- **Live-verified prices** — All prices come from live MCP data or web search, timestamped and source-attributed. No rounding, no estimating, no fabricating.
- **Family-aware** — Knows child fare policies across operators (free on Deutsche Bahn, 30% off Eurostar, full fare on airlines) and includes children in booking links.
- **Parallel agent search** — Three specialist agents (flights, accommodation, activities) search simultaneously for faster trip planning.
- **Post-booking price monitoring** — Watches booked items for price drops and alerts with net savings after cancellation fees and loyalty implications.
- **Document scanning** — Upload boarding passes, receipts, and hotel bills for automatic extraction, expense tracking, and overcharge detection.
- **Gmail + Calendar integration** — Detects booking confirmations in Gmail and creates calendar events with airport buffer times and packing reminders.

---

## Setup

### Requirements

- **Node.js 18+** with `npx` in your PATH (for MCP servers)
- **Claude Code** or **Claude Cowork**

### Install

**Claude Code CLI:**
```bash
claude plugin install https://github.com/mlobo2012/claude-travel-agent
```

**From a local clone:**
```bash
git clone https://github.com/mlobo2012/claude-travel-agent.git
claude plugin install ./claude-travel-agent
```

**Claude Cowork (desktop app):**
1. Download or clone this repository
2. Zip the `claude-travel-agent` folder
3. Open Cowork → Browse Plugins → Personal → click **"+"**
4. Select the zip file

### MCP Servers

The plugin ships with five core MCP servers configured in `.mcp.json`. They run via `npx` automatically — no API keys needed.

| Server | What it does |
|--------|-------------|
| **Airbnb** | Accommodation search, property details, availability |
| **Google Flights** | Flight routes, prices, schedules |
| **Public Transport** | European/international train and bus routes |
| **TfL** | London transport journey planning |

Two optional connectors add extra capabilities:

| Connector | What it adds | Setup |
|-----------|-------------|-------|
| **Gmail** | Booking confirmation detection | Grant read-only permissions when prompted |
| **Google Calendar** | Trip events, reminders | Grant read-write permissions when prompted |

### Verify Setup

After installing, ask Claude: "What MCP tools are available?" You should see tools from airbnb, google-flights, public-transport, and tfl.

---

## Getting Started

### Onboarding

Onboarding is **optional**. The plugin builds your profile organically from conversation — every time you mention your home city, companions, loyalty numbers, or preferences, they're captured silently.

To do a full profile setup explicitly: `/travel-setup`

This asks about your home airport, travel style, children, accommodation preferences, budget, loyalty programmes, passport details, and more. Everything is saved and applied to future searches.

### Quick Start Without Onboarding

Just ask a travel question:

> Find me flights to Sicily in May for 2 adults

The plugin will ask only the 2-3 missing details it needs, run the search, and silently save what you told it for next time.

---

## How Persistence Works

The plugin uses a **dual-persistence** model:

| Store | Role | Reliability |
|-------|------|------------|
| **Project file** (`${CLAUDE_PLUGIN_DATA}/travel-profile.json`) | Primary, authoritative | Reliable within the same Claude Code project |
| **Claude memory** (markdown summary pushed to Claude's memory system) | Best-effort backup | May persist across projects/environments, but not guaranteed |

**How it works:**
- Every profile save writes to **both** stores
- On load, the plugin tries the file first, then Claude's memory, then asks you
- If Claude's memory has data but the file doesn't, it reconstructs and re-saves the file

**Limitations:**
- Cross-project persistence depends on Claude's memory system surfacing the data — this is best-effort
- In Cowork, the file store is session-scoped, so Claude memory is the only persistence path
- Claude memory stores a markdown summary, not the full JSON — some minor detail may be lost on reconstruction

---

## Commands

| Command | What it does |
|---------|-------------|
| `/travel-setup` | Full profile onboarding (optional — profile builds from conversation too) |
| `/travel-profile` | View your saved preferences |
| `/travel-profile update` | Update a specific preference |
| `/find-flights` | Search flights with live prices and deep booking links |
| `/find-trains` | Search trains with child fares and operator deep links |
| `/find-ferries` | Search ferries with cabin options and operator deep links |
| `/find-accommodation` | Search Airbnb and Booking.com with preference scoring |
| `/plan-trip` | Full end-to-end trip planning (parallel agents + transport intelligence) |
| `/loyalty-manager` | View, add, or manage loyalty programmes |
| `/price-monitor` | Set up pre-booking price watches |
| `/travel-expenses` | Generate expense report from scanned receipts |
| `/trip-pack` | Generate a pre-departure trip document |
| `/feedback` | Record post-trip feedback to improve future results |
| `/travel-help` | Quick reference for all commands |

---

## Example Prompts

**Family trip planning:**
> Plan a trip to Paris for me and my two kids (ages 5 and 8). We're in London. Mid-May, 4 nights.

The plugin recommends Eurostar (kids 30% off, free bags) over flights, finds family-friendly Airbnbs, plans kid-friendly activities, and provides TfL connections from St Pancras.

**Flight search with loyalty:**
> Find me flights to New York from Heathrow, late June, 2 people

Results include BA with your Executive Club number pre-filled in the booking link, plus alternatives with loyalty tradeoff explained.

**Challenge a recommendation:**
> Why didn't you suggest the Holiday Inn?

The plugin explains the scoring: "Holiday Inn scored 45 vs Marriott's 72 because (1) no loyalty match, (2) 'basic breakfast' is in your avoid tags from your Birmingham trip, (3) review score 3.8 vs your 4.0 minimum."

**Post-booking monitoring:**
> I booked that BA flight to NYC for £450

Confirms, logs the booking, and starts automatic price monitoring. If the price drops: "PRICE DROP: BA LHR→JFK dropped to £380. Net savings after cancellation: £70. Your Avios are preserved. [Rebook now]"

**Document scanning:**
> *Upload a hotel bill photo*

"Your Marriott bill shows £189/night but your booking was £165/night — that's £48 overcharged across 2 nights. Added to your NYC trip expenses (total: £1,247 / £1,500 budget)."

---

## Booking Links

Every search result includes a **deep booking link** that takes you directly to the specific option with route, dates, and passengers pre-filled. The plugin never links to a generic homepage.

**Coverage:**
- **Flights:** Ryanair, easyJet, BA, Wizz Air, Vueling, Lufthansa, KLM, Air France, Iberia, TAP, Aer Lingus, Norwegian, SAS, Turkish Airlines, Emirates, Qatar Airways — plus Google Flights fallback for any other airline
- **Trains:** Eurostar, Trainline, Deutsche Bahn, SNCF, Omio, Amtrak, Italo, Renfe
- **Ferries:** DFDS, Stena Line, P&O, Brittany Ferries, Blue Star, Viking Line, Jadrolinija, Irish Ferries, Color Line — plus Ferryhopper/Direct Ferries fallback
- **Hotels:** Airbnb (with dates + guests), Booking.com, Marriott (with Bonvoy number), Hilton, IHG, Accor

Link text always names the specific option: `Book BA548 LHR→FCO 14 May` — not `Search on Google Flights`.

Loyalty membership numbers are included in booking URLs where the platform supports it. Where it doesn't, the plugin advises logging in before booking.

**Current limitations:**
- Deep link URL templates are best-effort — airline and operator websites occasionally change their URL structure, which may cause a link to load a search page instead of the exact result. The fallback hierarchy (operator direct → aggregator → search with route pre-filled) handles this gracefully.
- Some platforms don't support child parameters or loyalty numbers in URLs — the plugin notes this and advises manual entry.

---

## How Transport Intelligence Works

Before any transport search, the plugin:
1. Reads your profile (transport preference, children, budget, loyalty programmes)
2. Checks the route against 30+ known train-competitive routes
3. Applies rules: under 4h by train + kids → lead with train; island routes → lead with ferry; cross-channel → always show Eurostar
4. Checks loyalty implications for each mode
5. Recommends the best option first with full reasoning

---

## Child & Family Quick Reference

| Operator | Free Under | Discount | Luggage |
|----------|-----------|----------|---------|
| Eurostar | 4 (on lap) | 4–11: ~30% off | Unlimited free |
| UK National Rail | 5 | 5–15: 50% off | Unlimited free |
| Deutsche Bahn | 6 | 6–14: FREE with adult | Unlimited free |
| SNCF (TGV) | 4 | 4–11: ~30% off | Unlimited free |
| Amtrak | 2 (on lap) | 2–12: 50% off | 5 free items |
| Most airlines | 2 (on lap, ~10%) | 2+: full fare | £25–45/bag |

---

## Troubleshooting

**MCP servers not starting:** Ensure Node.js 18+ is installed and `npx` is in your PATH. Run `npx @openbnb/mcp-server-airbnb --help` to test.

**"No profile found" on a new session:** The plugin will try the file first, then Claude's memory. If both are empty, it asks you. Profile persistence within the same project is reliable. Cross-project persistence depends on Claude's memory system.

**Booking link loads a generic search page:** Airline/operator websites occasionally change their URL structure. The plugin's fallback hierarchy handles this — if the direct link doesn't work, it falls back to an aggregator link with the route pre-filled.

**Gmail/Calendar not connecting:** These are optional connectors that require permission grants. The core search and planning features work without them.

---

## Development

```bash
# Validate the plugin
claude plugin validate

# Test locally
claude --plugin-dir ./claude-travel-agent
```

---

## Version History

| Version | Date | Highlights |
|---------|------|-----------|
| **2.4.0** | 2026-04-03 | Dedicated booking-links skill (canonical URL reference for 15+ airlines, 8+ rail, 10+ ferry, 6+ hotel platforms). Explicit primary/backup persistence architecture. Clean README rewrite. |
| **2.3.1** | 2026-03-28 | Cross-session persistence fix with dual-persistence strategy. Deep booking links for 7+ airlines, trains, ferries, hotels. Enhanced onboarding (seat, personal details, passport). |
| **2.2.0** | 2026-03-15 | Post-booking price re-shopping. Reasoning transparency ("Why this?"). Loyalty programme intelligence. Document scanner and expense tracking. |
| **2.1.0** | 2026-03-01 | Multi-modal transport (trains, ferries, local transit). Proactive transport intelligence. Family-aware child fare policies. 30+ route knowledge base. |
| **2.0.0** | 2026-02-15 | Persistent memory. Parallel agent search. Price monitoring. Pre-trip reminders. Trip packs. Gmail booking detection. Google Calendar integration. |

---

## License

MIT

---

Built by [AI Heroes](https://www.ai-heroes.co) — AI That Learns How You Win.

Created by Marco Lobo.
