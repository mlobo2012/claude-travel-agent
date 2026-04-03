# AI Heroes Travel Agent v2.1

A full multi-agent travel system plugin for **Claude Code** and **Claude Cowork** with **multi-modal transport intelligence**. Plans trips end-to-end with proactive transport mode selection, parallel agent search, persistent memory learning, price monitoring, pre-trip reminders, trip packs, Gmail booking detection, and Google Calendar integration. All prices are live-verified with anti-hallucination guardrails.

Built by [AI Heroes](https://www.ai-heroes.co).

## What's New in v2.1

- **Multi-modal transport** — Trains, ferries, and local transit alongside flights
- **Proactive intelligence** — Automatically recommends the best transport mode based on your profile, not just flights by default
- **Family-aware** — Knows that kids ride free on Deutsche Bahn, get 30% off Eurostar, and pay full fare on airlines
- **Local transit** — Last-mile connections from stations/airports to your accommodation (TfL MCP for London, web search elsewhere)
- **Embedded knowledge base** — 30+ train-competitive routes, child policies, scenic routes, and ferry connections

### Previously in v2.0

- **Persistent Memory** — Preferences, feedback, and trip history persist across all sessions via Cowork's built-in memory
- **Parallel Agent Search** — 3 specialist agents (flights, accommodation, activities) search simultaneously for faster trip planning
- **Price Monitoring** — Watch flights and accommodation prices twice daily; get alerts on drops >10% or rises >20%
- **Pre-Trip Reminders** — Dynamic reminders at T-14, T-7, T-3, and T-1 days with weather forecasts and packing suggestions
- **Trip Pack** — Branded pre-departure document with all trip details, itinerary, practical info, and packing checklist
- **Gmail Booking Detection** — Auto-detect booking confirmation emails and log them to your trip (requires Gmail connector)
- **Google Calendar Integration** — Auto-create calendar events for flights, accommodation, activities, and packing reminders (requires Google Calendar connector)
- **Learning Engine** — Every search interaction and trip feedback refines future recommendations

---

## What It Does

- **Proactive transport intelligence** — Analyses the route, your profile, and trip context to recommend trains, flights, or ferries before you ask
- **Flight search** via Google Flights MCP — live prices, direct booking links, multi-airport London search
- **Train search** via Public Transport MCP — European and US rail routes, child fare policies
- **Ferry search** via Ferryhopper MCP — European ferry routes, island hopping, overnight sailings
- **Local transit** via TfL MCP (London) + web search — airport/station transfers, getting around
- **Accommodation search** via Airbnb MCP — live prices, review scores, amenities, preference scoring
- **Full trip planning** — transport + accommodation + local transit + day-by-day itinerary + practical info
- **Anti-hallucination guardrails** — every price is live-verified, timestamped, and source-attributed. No rounding, estimating, or fabricating prices. Ever.
- **Preference learning** — onboarding builds a profile that improves every search
- **Post-trip feedback** — learns from your experiences to refine future recommendations
- **Cross-checking** — prices verified across multiple sources when possible
- **Family travel** — child fares, luggage policies, and family-friendly recommendations built in

---

## How the Proactive Intelligence Works

Unlike a basic search tool, this agent **thinks about the best way to travel before searching**:

1. **Reads your profile** — transport preferences, children's ages, budget, comfort vs speed vs eco priorities
2. **Analyses the route** — checks against 30+ known train-competitive routes in Europe and the US
3. **Applies rules** — "Under 4 hours by train + travelling with kids? Lead with train and explain why."
4. **Presents the best option first** — with a clear comparison if multiple modes are competitive

### Examples

| You say | What happens |
|---------|-------------|
| "Plan a trip to Paris" (profile: London, 2 kids age 5 & 8) | Leads with Eurostar (2h17, kids 30% off, unlimited free bags) — notes flights as alternative |
| "Find me transport to Munich" (profile: Berlin, eco preference) | Leads with ICE train (4h, kids under 14 free with adult) |
| "I need to get to DC" (profile: NYC) | Shows Acela (2h45, city centre) alongside flights |
| "Trip to Santorini" | Shows ferry options from Athens, plus flights |
| "Find me flights to Amsterdam" | Searches flights as requested, but notes: "Eurostar does this in 3h52 if you'd like to compare" |

---

## Installation

### Option 1: Claude Code CLI

```bash
claude plugin install /path/to/claude-travel-agent
```

### Option 2: Claude Cowork Desktop App (Local Upload)

1. Download or clone this repository
2. Zip the `claude-travel-agent` folder (the folder containing `.claude-plugin/`)
3. Open Claude Cowork desktop app
4. Go to **Browse Plugins** > **Personal** > click the **"+"** button
5. Select the zip file
6. The plugin will install and show up under Personal plugins

### Option 3: From GitHub

```bash
claude plugin install https://github.com/mlobo2012/claude-travel-agent
```

---

## MCP Server Setup

This plugin uses five core MCP servers plus two optional connectors. They are configured in `.mcp.json` and run automatically via `npx` — no global installation needed. You just need **Node.js 18+** with `npx` in your PATH.

### Core MCPs (zero API keys required)

| Server | Package | What it does |
|--------|---------|-------------|
| **Airbnb** | [`@openbnb/mcp-server-airbnb`](https://www.npmjs.com/package/@openbnb/mcp-server-airbnb) | Search Airbnb listings, property details, availability |
| **Google Flights** | [`google-flights-mcp-server`](https://www.npmjs.com/package/google-flights-mcp-server) | Flight routes, prices, schedules, date grids |
| **Public Transport** | [`mcp-server-public-transport`](https://www.npmjs.com/package/mcp-server-public-transport) | European/international train and bus routes |
| **TfL** | [`@daanrongen/tfl-mcp`](https://www.npmjs.com/package/@daanrongen/tfl-mcp) | London transport journey planning |
| **Ferryhopper** | Remote: `https://mcp.ferryhopper.com/mcp` | European ferry routes and schedules |

All core MCPs work out of the box with no API keys.

### Optional Power-User MCPs

These are **not** included in the default `.mcp.json` — add them manually if you want enhanced coverage:

| Server | How to add | What it adds |
|--------|-----------|-------------|
| **Google Maps** | Requires `GOOGLE_MAPS_API_KEY` | Walking/driving directions, place details |
| **SNCF** | Check npm for SNCF MCP servers | Enhanced French rail coverage |
| **Deutsche Bahn** | Check npm for DB MCP servers | Enhanced German rail coverage |
| **National Rail** | Check npm for UK rail MCP servers | Enhanced UK rail coverage |

To add an optional MCP, edit `.mcp.json` and add the server entry.

### Verifying MCP Servers

After installing the plugin, check that MCP servers are connected:

```
# In Claude Code, ask:
"What MCP tools are available?"

# You should see tools from airbnb, google-flights, public-transport, tfl, and ferryhopper
```

If servers fail to start, ensure you have Node.js 18+ and npx available in your PATH.

---

## Optional Connectors

These connectors enable additional features. They are optional — the core search and planning features work without them.

### Gmail (Booking Detection)

- **Purpose:** Read-only access to detect booking confirmation emails from airlines and accommodation platforms
- **Setup:** When prompted by Cowork, grant Gmail read-only permissions
- **What it reads:** Only booking confirmations from known airline and hotel senders (Ryanair, easyJet, BA, Airbnb, Booking.com, etc.)
- **Privacy:** Never sends, deletes, or modifies emails. Only extracts booking references and dates.

### Google Calendar (Trip Events)

- **Purpose:** Create calendar events for flights, accommodation, activities, and reminders
- **Setup:** When prompted by Cowork, grant Google Calendar read-write permissions
- **What it creates:** Flight events (with airport buffer times), check-in/check-out events, activity events, packing reminders, trip date blocks

---

## Commands

| Command | Description |
|---------|-------------|
| `/onboarding` | First-time setup — conversational questions to build your travel profile (including transport preferences and children's ages) |
| `/memory-manager` | View your saved travel preferences |
| `/memory-manager update` | Update a specific preference |
| `/memory-manager history` | See your past trips |
| `/flight-search` | Search for flights with live-verified prices |
| `/train-search` | Search for trains with child fare policies |
| `/ferry-search` | Search for ferries with cabin options |
| `/accommodation-search` | Search Airbnb and Booking.com |
| `/trip-planner` | Full end-to-end trip planning (parallel agent search + transport intelligence + local transit) |
| `/price-monitor` | Set up or manage price watches |
| `/trip-pack` | Generate your pre-departure trip pack |
| `/feedback` | Record post-trip feedback to improve future results |
| `/travel-help` | Quick reference for all commands |

**Tip:** Run `/onboarding` first to get personalised results from day one.

---

## Use Cases

### Family Travel
"Plan a trip to Paris for me and my two kids (ages 5 and 8), from London"
- Proactively recommends Eurostar (2h17, kids ~30% off, unlimited free bags vs £25-45/bag on flights)
- Shows family-friendly Airbnb listings
- Provides St Pancras → accommodation connection via TfL
- Day-by-day itinerary with kid-friendly activities

### Business Trip
"I need to be in DC on Tuesday for meetings, flying from NYC"
- Shows Acela (2h45, Penn Station to Union Station) alongside flights
- Notes that total door-to-door time is similar, but train avoids airport security
- Hotel recommendations near the meeting area

### Island Hopping
"Plan a Greek island trip — Mykonos and Santorini"
- Ferry routes between islands (the practical option)
- Flight to/from Athens
- Accommodation on each island
- Local transit and tips

### Budget Backpacking
"Cheapest way to get from London to Berlin"
- Compares: Eurostar + Thalys/ICE, budget airlines, coach
- Shows advance purchase savings
- Notes the USA Rail Pass equivalent for European multi-city (Interrail)

### Scenic Route
"We're not in a rush — what's the most scenic way to get from Zurich to Italy?"
- Recommends the Bernina Express (UNESCO World Heritage route) or Gotthard Panorama
- Notes journey times, booking requirements, and what you'll see

---

## How It Works

### Transport Intelligence

Before any transport search, the agent:
1. Reads your profile (transport preference, children's ages, budget)
2. Checks the route against 30+ known train-competitive routes
3. Applies proactive rules (under 4h train + kids → lead with train)
4. Recommends the best mode with clear reasoning

### Parallel Agent Search

When you use `/trip-planner`, three specialist agents search simultaneously:

1. **flight-searcher** — Queries Google Flights MCP for all matching routes
2. **accommodation-searcher** — Queries Airbnb MCP for matching listings
3. **activities-researcher** — Researches activities, restaurants, and day trips via web search

Results are combined into a unified trip plan with transport, accommodation, activities, practical info, and budget summary.

### Persistent Memory

Your travel profile is stored in Cowork's persistent memory:
- `travel_profile` — full preference profile
- `travel_derived_preferences` — learned prefer/avoid tags
- `travel_feedback` — post-trip feedback history
- `travel_past_trips` — trip history with outcomes

Preferences are automatically refined after every search interaction and trip feedback.

### Research Pipeline

Every search follows a 3-stage quality pipeline:

1. **Live Retrieval** — Query MCP servers for real-time data
2. **Cross-Check** — Verify prices on a second source (web search fallback)
3. **Quality Filter** — Apply minimum review thresholds, preference scoring, and present top 3

### Anti-Hallucination Rules

- Every price is retrieved live and timestamped
- No rounding, estimating, or inferring prices
- Cross-check discrepancies >5% are flagged with both values
- Missing data is explicitly noted, never fabricated
- Direct booking links are verified, not constructed

### Preference Scoring

Results are ranked using your profile:
- **+15 points** for each matching preference
- **-50 points** for each matching "avoid" tag
- **+20 points** for transport mode matching your preference
- **+10 points** for child-friendly features (when travelling with children)
- Score below 0 = excluded from results

### Data Storage

- **Primary:** Cowork persistent memory (survives across sessions)
- **Secondary backup:** `~/.claude/plugins/data/ai-heroes-travel-agent/travel-profile.json`
- **Trip plans:** `~/.claude/plugins/data/ai-heroes-travel-agent/trips/`

---

## Child & Family Quick Reference

| Operator | Free Under | Discount | Luggage |
|---|---|---|---|
| Eurostar | 4 (on lap) | 4–11: ~30% off | Unlimited free |
| UK National Rail | 5 | 5–15: 50% off | Unlimited free |
| Deutsche Bahn | 6 | 6–14: FREE with adult | Unlimited free |
| SNCF (TGV) | 4 | 4–11: ~30% off | Unlimited free |
| Amtrak | 2 (on lap) | 2–12: 50% off | 5 free items |
| Most airlines | 2 (on lap, ~10% fare) | 2+: full fare | £25–45/bag |

---

## MCP Servers — Status

| Server | Status | Package |
|--------|--------|---------|
| Airbnb | Real (MCP) | `@openbnb/mcp-server-airbnb` |
| Google Flights | Real (MCP) | `google-flights-mcp-server` |
| Public Transport | Real (MCP) | `mcp-server-public-transport` |
| TfL | Real (MCP) | `@daanrongen/tfl-mcp` |
| Ferryhopper | Remote (MCP) | `https://mcp.ferryhopper.com/mcp` |
| Gmail | Connector (HTTP) | Google Gmail MCP — read-only, optional |
| Google Calendar | Connector (HTTP) | Google Calendar MCP — read-write, optional |
| Booking.com | Fallback (web search) | No working MCP — uses web search for cross-checking |
| Skyscanner | Fallback (web search) | No working MCP — uses web search for cross-checking |
| Trainline | Fallback (web search) | No working MCP — uses web search for cross-checking |

---

## Try It Out

After installation and `/onboarding`, test with:

> Plan a trip to Paris for me and my two kids (ages 5 and 8). We're in London. Mid-May, 4 nights. I want the entire trip planned out.

You should see: Eurostar recommended (with child fare savings highlighted), real Airbnb listings, TfL connection from St Pancras, a day-by-day itinerary with kid-friendly activities, and practical info — all timestamped and source-attributed.

**More tests:**

- **Agent Teams:** "Plan a trip to Punta Secca in Sicily, mid-May, 4 people, direct flights from London"
- **Price monitoring:** "Watch the easyJet flight price for me"
- **Persistent memory:** "What do you remember about my travel preferences?"
- **Trip pack:** "Send me my trip pack"

---

## Plugin Structure

```
claude-travel-agent/
├── .claude-plugin/
│   └── plugin.json                         # Plugin manifest (v2.1.0)
├── .mcp.json                               # MCP server + connector configuration (7 servers)
├── agents/                                 # Agent Teams (v2.0)
│   ├── flight-agent.md                     # Flight search specialist
│   ├── accommodation-agent.md              # Accommodation search specialist
│   └── activities-agent.md                 # Activities researcher
├── skills/
│   ├── travel-agent/SKILL.md               # Core identity and rules
│   ├── transport-intelligence/SKILL.md     # Proactive transport mode selection (v2.1)
│   ├── train-search/SKILL.md               # Train route search (v2.1)
│   ├── ferry-search/SKILL.md               # Ferry route search (v2.1)
│   ├── local-transit/SKILL.md              # Last-mile connections (v2.1)
│   ├── onboarding/SKILL.md                 # First-time profile setup
│   ├── accommodation-search/SKILL.md       # Airbnb + Booking.com search
│   ├── flight-search/SKILL.md              # Google Flights search
│   ├── trip-planner/SKILL.md               # End-to-end trip planning (agent teams + transport intelligence)
│   ├── memory-manager/SKILL.md             # Travel profile management
│   ├── feedback/SKILL.md                   # Post-trip feedback (learning engine)
│   ├── research-pipeline/SKILL.md          # 3-stage quality pipeline (auto)
│   ├── guardrails/SKILL.md                 # Anti-hallucination rules (auto)
│   ├── persistent-memory/SKILL.md          # Memory integration (auto)
│   ├── price-monitor/SKILL.md              # Price monitoring (v2.0)
│   ├── trip-reminders/SKILL.md             # Pre-trip reminders (auto, v2.0)
│   ├── trip-pack/SKILL.md                  # Trip pack generation (v2.0)
│   ├── booking-detection/SKILL.md          # Gmail detection (auto, v2.0)
│   └── calendar-integration/SKILL.md       # Calendar events (auto, v2.0)
├── commands/
│   └── travel-help.md                      # /travel-help quick reference
├── LICENSE
└── README.md
```

---

## Development

### Validating

```bash
claude plugin validate
```

### Testing Locally

```bash
claude --plugin-dir ./claude-travel-agent
```

---

## License

MIT

---

## Credits

Built by [AI Heroes](https://www.ai-heroes.co) — AI That Learns How You Win.

Created by Marco Lobo.
