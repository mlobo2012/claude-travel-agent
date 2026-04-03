# AI Heroes Travel Agent v2.0

A full multi-agent travel system plugin for Claude Code / Claude Cowork. Plans trips end-to-end with parallel agent search, persistent memory learning, price monitoring, pre-trip reminders, and Gmail/Calendar integration. All prices are live-verified with anti-hallucination guardrails.

Built by [AI Heroes](https://www.ai-heroes.co).

## What's New in V2.0

- **Persistent Memory** — Preferences, feedback, and trip history persist across all sessions via Cowork's built-in memory
- **Parallel Agent Search** — 3 specialist agents (flights, accommodation, activities) search simultaneously for faster trip planning
- **Price Monitoring** — Watch flights and accommodation prices twice daily; get alerts on drops >10% or rises >20%
- **Pre-Trip Reminders** — Dynamic reminders at T-14, T-7, T-3, and T-1 days with weather forecasts and packing suggestions
- **Trip Pack** — Branded pre-departure document with all trip details, itinerary, practical info, and packing checklist
- **Gmail Booking Detection** — Auto-detect booking confirmation emails and log them to your trip (requires Gmail connector)
- **Google Calendar Integration** — Auto-create calendar events for flights, accommodation, activities, and packing reminders (requires Google Calendar connector)
- **Learning Engine** — Every search interaction and trip feedback refines future recommendations

## Features

### Core (v1.0)
- **Flight search** via Google Flights MCP — live prices, direct booking links, multi-airport London search
- **Accommodation search** via Airbnb MCP — live prices, review scores, amenities, preference scoring
- **Full trip planning** — flights + accommodation + day-by-day itinerary + practical info + budget summary
- **Anti-hallucination guardrails** — every price is live-verified, timestamped, and source-attributed
- **Preference learning** — 10-question onboarding builds a profile that improves every search
- **Post-trip feedback** — learns from your experiences to refine future recommendations
- **Cross-checking** — prices verified across multiple sources when possible

### V2.0 Additions
- **Persistent memory** — profile, preferences, and trip history survive across sessions
- **Agent Teams** — parallel search via flight-searcher, accommodation-searcher, and activities-researcher agents
- **Price monitoring** — scheduled tasks check prices twice daily with configurable alert thresholds
- **Pre-trip reminders** — weather-aware, destination-appropriate reminders before departure
- **Trip pack generation** — comprehensive pre-departure document auto-generated or on-demand
- **Booking detection** — Gmail connector reads airline and hotel confirmation emails
- **Calendar integration** — Google Calendar connector creates events with airport buffer times

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

## MCP Server Setup

This plugin uses two MCP servers for live data. They run automatically via `npx` — no global installation needed.

### Airbnb (`@openbnb/mcp-server-airbnb`)

- **Package:** `@openbnb/mcp-server-airbnb` (v0.1.3)
- **What it does:** Searches Airbnb listings, retrieves details, checks availability
- **Setup:** Runs automatically — no API key required

### Google Flights (`google-flights-mcp-server`)

- **Package:** `google-flights-mcp-server` (v0.2.1)
- **What it does:** Searches Google Flights for routes, prices, and schedules
- **Setup:** Runs automatically — no API key required

### Verifying MCP Servers

After installing the plugin, check that MCP servers are connected:

```
# In Claude Code, the servers should auto-start
# You can verify by asking: "What MCP tools are available?"
```

If servers fail to start, ensure you have Node.js 18+ and npx available in your PATH.

## Optional Connectors

These connectors enable additional V2.0 features. They are optional — the core search and planning features work without them.

### Gmail (Booking Detection)

- **Purpose:** Read-only access to detect booking confirmation emails from airlines and accommodation platforms
- **Setup:** When prompted by Cowork, grant Gmail read-only permissions
- **What it reads:** Only booking confirmations from known airline and hotel senders (Ryanair, easyJet, BA, Airbnb, Booking.com, etc.)
- **Privacy:** Never sends, deletes, or modifies emails. Only extracts booking references and dates.

### Google Calendar (Trip Events)

- **Purpose:** Create calendar events for flights, accommodation, activities, and reminders
- **Setup:** When prompted by Cowork, grant Google Calendar read-write permissions
- **What it creates:** Flight events (with airport buffer times), check-in/check-out events, activity events, packing reminders, trip date blocks

## Commands

| Command | Description |
|---------|-------------|
| `/onboarding` | First-time onboarding — 10 questions to build your travel profile |
| `/memory-manager` | View your saved preferences |
| `/memory-manager update` | Update a specific preference |
| `/memory-manager history` | See your past trips |
| `/flight-search` | Search for flights with live-verified prices |
| `/accommodation-search` | Search Airbnb and Booking.com |
| `/trip-planner` | Full end-to-end trip planning (parallel agent search) |
| `/price-monitor` | Set up or manage price watches |
| `/trip-pack` | Generate your pre-departure trip pack |
| `/feedback` | Record post-trip feedback |
| `/travel-help` | Quick reference for all commands |

## How It Works

### Parallel Agent Search (V2.0)

When you use `/trip-planner`, three specialist agents search simultaneously:

1. **flight-searcher** — Queries Google Flights MCP for all matching routes
2. **accommodation-searcher** — Queries Airbnb MCP for matching listings
3. **activities-researcher** — Researches activities, restaurants, and day trips via web search

Results are combined into a unified trip plan with flights, accommodation, activities, practical info, and budget summary.

### Persistent Memory (V2.0)

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
- +15 points for each matching preference
- -50 points for each matching "avoid" tag
- Score < 0 = excluded from results

### Data Storage

- **Primary:** Cowork persistent memory (survives across sessions)
- **Secondary backup:** `~/.claude/plugins/data/ai-heroes-travel-agent/travel-profile.json`
- **Trip plans:** `~/.claude/plugins/data/ai-heroes-travel-agent/trips/`

## MCP Servers — What's Real vs. Stubbed

| Server | Status | Package |
|--------|--------|---------|
| Airbnb | Real (MCP) | `@openbnb/mcp-server-airbnb` v0.1.3 |
| Google Flights | Real (MCP) | `google-flights-mcp-server` v0.2.1 |
| Gmail | Connector (HTTP) | Google Gmail MCP — read-only, optional |
| Google Calendar | Connector (HTTP) | Google Calendar MCP — read-write, optional |
| Booking.com | Fallback (web search) | No working MCP — uses web search for cross-checking |
| Skyscanner | Fallback (web search) | No working MCP — uses web search for cross-checking |
| Maps | Not included | `@mapbox/mcp-server` available but not bundled (optional) |

## Development

### Plugin Structure

```
claude-travel-agent/
├── .claude-plugin/
│   └── plugin.json                    # Plugin manifest (v2.0.0)
├── .mcp.json                          # MCP server + connector configuration
├── agents/                            # Agent Teams (V2.0)
│   ├── flight-agent.md                # Flight search specialist
│   ├── accommodation-agent.md         # Accommodation search specialist
│   └── activities-agent.md            # Activities researcher
├── skills/
│   ├── travel-agent/SKILL.md          # Core identity and rules
│   ├── onboarding/SKILL.md            # /travel-setup command
│   ├── accommodation-search/SKILL.md  # /find-accommodation
│   ├── flight-search/SKILL.md         # /find-flights
│   ├── trip-planner/SKILL.md          # /plan-trip (agent teams)
│   ├── memory-manager/SKILL.md        # /travel-profile
│   ├── feedback/SKILL.md              # /trip-feedback (learning engine)
│   ├── research-pipeline/SKILL.md     # Quality pipeline (auto)
│   ├── guardrails/SKILL.md            # Anti-hallucination (auto)
│   ├── persistent-memory/SKILL.md     # Memory integration (auto)
│   ├── price-monitor/SKILL.md         # /price-monitor
│   ├── trip-reminders/SKILL.md        # Pre-trip reminders (auto)
│   ├── trip-pack/SKILL.md             # /trip-pack
│   ├── booking-detection/SKILL.md     # Gmail detection (auto)
│   └── calendar-integration/SKILL.md  # Calendar events (auto)
├── commands/
│   └── travel-help.md                 # /travel-help quick reference
└── README.md
```

### Validating

```bash
claude plugin validate
```

### Testing Locally

```bash
claude --plugin-dir ./claude-travel-agent
```

Then test with:

**Agent Teams test:**
> "Plan a trip to Punta Secca in Sicily, mid-May, 4 people, direct flights from London, mid-range accommodation, cheap flights"

**Price monitoring test:**
> "Watch the easyJet flight price for me"

**Persistent memory test:**
> "What do you remember about my travel preferences?"

**Trip pack test:**
> "Send me my trip pack"

## License

MIT

## Credits

Built by [AI Heroes](https://www.ai-heroes.co) — AI That Learns How You Win.
