# AI Heroes Travel Agent v2.2

A full multi-agent travel system plugin for **Claude Code** and **Claude Cowork** with **deep intelligence features**. Plans trips end-to-end with proactive transport mode selection, reasoning transparency on every recommendation, loyalty programme intelligence, post-booking price re-shopping, document scanning, expense tracking, parallel agent search, persistent memory learning, price monitoring, pre-trip reminders, trip packs, Gmail booking detection, and Google Calendar integration. All prices are live-verified with anti-hallucination guardrails.

Built by [AI Heroes](https://www.ai-heroes.co).

## What's New in v2.2

### Post-Booking Price Re-Shopping
After you book a flight or hotel, the plugin automatically monitors the price. If it drops significantly, you get a proactive alert with:
- Original price vs new price, with exact savings amount
- Cancellation fee (if known) and **net savings** after the fee
- Direct rebooking link for the new price
- Recommended action (rebook, hold, or wait for further drops)
- Loyalty implications if rebooking on a different carrier ("Rebooking on Ryanair saves £80 but you lose 4,500 Avios from the BA booking")
- **Price trend analysis** — tracks price history over time and surfaces patterns: "This route typically drops 15% 3 weeks before departure based on your booking history"

Configurable alert thresholds (default: >10% drop or >£50 savings, whichever triggers first).

### Reasoning Transparency — "Why This?"
Every single recommendation now includes a visible **Why this?** block that traces the reasoning:
- Which profile preferences influenced the choice
- Which past trip feedback was applied ("You rated the Marriott 5/5 last time in NYC")
- Which trip context signals were detected ("Travelling with 2 kids under 5 → family-friendly filter applied")
- Which transport intelligence rules fired ("Route under 3h by train + eco preference → Eurostar recommended over flight")
- Price vs comfort vs speed tradeoff that was made
- Loyalty earning opportunities factored in

**Challenge any recommendation** — say "Why didn't you suggest X?" and the agent explains what filtered X out and offers to update your preferences. Over time, the agent learns which factors you actually care about vs which you say you care about, by tracking which recommendations you accept vs override.

### Loyalty Programme Intelligence
Full loyalty programme management across airlines, hotels, and rail:
- **Auto-apply** — membership numbers are applied to every relevant search
- **Tier tracking** — "You need 2 more BA flights before March to keep Gold status. This trip could count if you fly BA."
- **Cross-programme optimisation** — "Marriott gives you a suite upgrade at your tier, Hilton is £30/night cheaper. Given your comfort preference, recommending Marriott."
- **Earning opportunities** — flags when a slightly more expensive option earns significant points
- **Alliance awareness** — knows oneworld, Star Alliance, SkyTeam partnerships
- **Rail loyalty** — Eurostar Club Eurostar, Amtrak Guest Rewards, BahnCard, UK Railcards
- **Proactive detection** — "I see you've flown BA 3 times. Do you have an Executive Club number?"

Manage with `/loyalty-manager` (view, add, update, status).

### Document Scanner & Expense Tracking
Upload a photo of any travel document and the agent extracts everything:
- **Boarding passes** — flight number, route, seat, gate, boarding time, frequent flyer number. Auto-updates trip record and loyalty tracker.
- **Receipts** — vendor, amount, currency, category. Logged to expense tracker with running trip total and budget comparison.
- **Booking confirmations** — all details extracted, trip record created, price reshop monitoring activated.
- **Hotel bills** — itemised charges compared against booked rate. Flags overcharges: "Your hotel charged £189/night but your booking was £165/night."

**Expense reports** with `/travel-expenses` — grouped by category, with totals, VAT breakdown, per-day option, and comparison to your historical spending patterns.

**Predictive spending intelligence** — "Your last 3 European trips averaged £180/night. This booking is £220/night — want me to look for alternatives?"

### Cross-Feature Intelligence
All v2.2 features integrate deeply with each other and existing features:
- Loyalty manager informs reasoning transparency ("BA flight is £80 more but earns 4,500 Avios and keeps your Gold status")
- Price reshop factors in loyalty implications when suggesting rebooking on different carriers
- Document scanner feeds the loyalty tracker (boarding pass → Avios balance update)
- Expense patterns inform trip planning budgets ("Based on your last 5 trips, a 4-night European city break typically costs £1,800")
- Reasoning transparency explains all of the above in every recommendation

### Previously in v2.1

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
- **Reasoning transparency** — Every recommendation explains WHY it was chosen, grounded in your profile, past trips, and context
- **Loyalty programme intelligence** — Auto-applies loyalty programmes, tracks tier progress, flags earning opportunities, cross-programme optimisation
- **Post-booking price reshop** — Monitors booked items for price drops, alerts with net savings after cancellation fees and loyalty implications
- **Document scanning** — Upload boarding passes, receipts, booking confirmations, hotel bills for automatic extraction and tracking
- **Expense tracking** — Running totals per trip, budget comparison, category breakdown, predictive spending patterns
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

## How the Deep Intelligence Works

### Reasoning Transparency

Unlike other travel tools that just show results, this agent explains its thinking:

| You see | What's happening |
|---------|-----------------|
| **Why this?** You prefer direct flights, have BA Executive Club Gold, and rated your last BA flight 5/5. This is BA direct with window seat available. | Profile preferences + past feedback + loyalty status → specific recommendation |
| **Why this over Ryanair?** Ryanair is £80 cheaper, but you'd lose 4,500 Avios and your Gold status needs 2 more flights. Given your comfort preference, leading with BA. | Cross-feature: loyalty + transport preference + tier tracking → tradeoff explanation |
| **Tradeoff:** Option A saves £40 but is 1h longer. Your profile says "budget priority" — showing A first. Want me to lead with speed instead? | Explicit tradeoff with offer to recalibrate |

Say "Why didn't you suggest X?" to understand what was filtered and update your preferences if needed.

### Post-Booking Price Intelligence

After you book, the agent watches prices automatically:

1. **Booking detected** (via manual confirmation, document scan, or Gmail)
2. **Price monitoring activated** — checks twice daily
3. **Price drop detected** → alert with original price, new price, cancellation fee, net savings
4. **Trend analysis** → "Prices on this route have dropped 3 days in a row — they may stabilise soon"
5. **Loyalty impact** → "Rebooking saves £120 but you lose your BA seat selection and 2,400 Avios"
6. **Action recommended** → rebook now, wait for further drop, or hold current booking

### Loyalty Programme Intelligence

```
Before search: Check loyalty programmes → auto-apply membership numbers
During search: Score results with loyalty bonus → flag earning opportunities
After search:  Track tier qualification → alert on status milestones
After booking: Monitor loyalty implications of any price reshop alternatives
After trip:    Update estimated points balance from scanned boarding passes
```

### Document Scanner Flow

```
User uploads photo → Detect document type → Extract all fields
  → Boarding pass: Update trip + loyalty tracker
  → Receipt: Log expense + update running total
  → Booking confirmation: Create trip + activate price reshop
  → Hotel bill: Compare against booked rate + flag overcharges
```

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

### Gmail (Booking Detection + Document Scanning)

- **Purpose:** Read-only access to detect booking confirmation emails from airlines and accommodation platforms. The document scanner can also offer to auto-scan confirmations from your inbox.
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
| `/onboarding` | First-time setup — conversational questions to build your travel profile (transport preferences, children's ages, loyalty programmes) |
| `/memory-manager` | View your saved travel preferences |
| `/memory-manager update` | Update a specific preference |
| `/memory-manager history` | See your past trips |
| `/flight-search` | Search for flights with live-verified prices |
| `/train-search` | Search for trains with child fare policies |
| `/ferry-search` | Search for ferries with cabin options |
| `/accommodation-search` | Search Airbnb and Booking.com |
| `/trip-planner` | Full end-to-end trip planning (parallel agent search + transport intelligence + local transit) |
| `/price-monitor` | Set up or manage pre-booking price watches |
| `/loyalty-manager` | View, add, update, or check status of loyalty programmes |
| `/travel-expenses` | Generate an expense report for a trip (supports `--daily` and `--vat` flags) |
| `/trip-pack` | Generate your pre-departure trip pack |
| `/feedback` | Record post-trip feedback to improve future results |
| `/travel-help` | Quick reference for all commands |

**Tip:** Run `/onboarding` first to get personalised results from day one — including loyalty programmes!

---

## Use Cases

### Family Travel with Loyalty Intelligence
"Plan a trip to Paris for me and my two kids (ages 5 and 8), from London"
- Proactively recommends Eurostar (2h17, kids ~30% off, unlimited free bags vs £25-45/bag on flights)
- **Why this?** "You have Eurostar Club Eurostar membership, kids under 12 get 30% off, and you rated your last Eurostar trip 5/5. Your eco preference also favours train."
- Shows family-friendly Airbnb listings with reasoning for each
- Provides St Pancras → accommodation connection via TfL

### Post-Booking Price Drop
"I booked that BA flight to NYC for £450"
- Confirms booking, extracts details, starts price reshop monitoring
- 3 days later: "PRICE DROP: Your BA LHR→JFK flight dropped from £450 to £380. BA allows free cancellation within the fare rules — net savings: £70. You'd keep your Avios. [Rebook now]"

### Document Scanning
*User uploads hotel bill photo*
- "Your Marriott bill shows £189/night but your booking was £165/night — that's £48 overcharged across 2 nights. I'd recommend querying this at checkout. Added to your NYC trip expenses (total: £1,247 / £1,500 budget — 83%)."

### Loyalty Tier Warning
"Find me flights to Berlin next month"
- Shows options with loyalty callout: "You need 2 more BA flights before March 31 to keep Gold status. This BA flight (£40 more than easyJet) would count as a qualifying flight. The easyJet option saves money but earns nothing."

### Business Trip
"I need to be in DC on Tuesday for meetings, flying from NYC"
- Shows Acela (2h45, Penn Station to Union Station) alongside flights
- **Why Acela first?** "Total door-to-door time is similar, but your profile says comfort priority for work trips. Train avoids airport security and you can work onboard. Your Amtrak Guest Rewards number has been applied."

### Challenge a Recommendation
"Why didn't you suggest the Holiday Inn?"
- "Holiday Inn scored 45 vs Marriott's 72. It was filtered because: (1) no loyalty programme match (-10), (2) you marked 'basic breakfast' as an avoid tag after your Birmingham trip in January (-50), (3) review score 3.8 vs your 4.0 minimum. Want me to remove the breakfast filter?"

---

## How It Works

### Transport Intelligence

Before any transport search, the agent:
1. Reads your profile (transport preference, children's ages, budget, loyalty programmes)
2. Checks the route against 30+ known train-competitive routes
3. Applies proactive rules (under 4h train + kids → lead with train)
4. Checks loyalty programme implications for each mode
5. Recommends the best option first with full reasoning transparency

### Parallel Agent Search

When you use `/trip-planner`, three specialist agents search simultaneously:

1. **flight-searcher** — Queries Google Flights MCP for all matching routes
2. **accommodation-searcher** — Queries Airbnb MCP for matching listings
3. **activities-researcher** — Researches activities, restaurants, and day trips via web search

Results are combined into a unified trip plan with transport, accommodation, activities, practical info, and budget summary — each with a **Why this?** reasoning block.

### Persistent Memory

Your travel profile is stored in Cowork's persistent memory:
- `travel_profile` — full preference profile (including loyalty programmes)
- `travel_derived_preferences` — learned prefer/avoid tags
- `travel_feedback` — post-trip feedback history
- `travel_past_trips` — trip history with outcomes
- `travel_reasoning_effectiveness` — which recommendation factors you actually respond to

Preferences are automatically refined after every search interaction and trip feedback.

### Research Pipeline

Every search follows a 3-stage quality pipeline:

1. **Live Retrieval** — Query MCP servers for real-time data
2. **Cross-Check** — Verify prices on a second source (web search fallback)
3. **Quality Filter** — Apply minimum review thresholds, preference scoring, loyalty scoring, and present top 3 with reasoning

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
- **+10 points** for loyalty programme match (alliance partner or direct)
- Score below 0 = excluded from results

### Data Storage

- **Primary:** Cowork persistent memory (survives across sessions)
- **Secondary backup:** `~/.claude/plugins/data/ai-heroes-travel-agent/travel-profile.json`
- **Trip plans:** `~/.claude/plugins/data/ai-heroes-travel-agent/trips/`
- **Booked trips:** `~/.claude/plugins/data/ai-heroes-travel-agent/booked-trips.json`
- **Expense log:** `~/.claude/plugins/data/ai-heroes-travel-agent/expense-log.json`
- **Reasoning effectiveness:** `~/.claude/plugins/data/ai-heroes-travel-agent/reasoning-effectiveness.json`

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

## Plugin Structure

```
claude-travel-agent/
├── .claude-plugin/
│   └── plugin.json                         # Plugin manifest (v2.2.0)
├── .mcp.json                               # MCP server + connector configuration (7 servers)
├── agents/                                 # Agent Teams (v2.0)
│   ├── flight-agent.md                     # Flight search specialist
│   ├── accommodation-agent.md              # Accommodation search specialist
│   └── activities-agent.md                 # Activities researcher
├── skills/
│   ├── travel-agent/SKILL.md               # Core identity and rules
│   ├── transport-intelligence/SKILL.md     # Proactive transport mode selection (v2.1)
│   ├── reasoning-transparency/SKILL.md     # Why This? reasoning on every recommendation (v2.2)
│   ├── loyalty-manager/SKILL.md            # Loyalty programme intelligence (v2.2)
│   ├── price-reshop/SKILL.md              # Post-booking price re-shopping (v2.2)
│   ├── document-scanner/SKILL.md           # Receipt, boarding pass, hotel bill scanning (v2.2)
│   ├── train-search/SKILL.md               # Train route search (v2.1)
│   ├── ferry-search/SKILL.md               # Ferry route search (v2.1)
│   ├── local-transit/SKILL.md              # Last-mile connections (v2.1)
│   ├── onboarding/SKILL.md                 # First-time profile setup (updated v2.2)
│   ├── accommodation-search/SKILL.md       # Airbnb + Booking.com search
│   ├── flight-search/SKILL.md              # Google Flights search
│   ├── trip-planner/SKILL.md               # End-to-end trip planning (agent teams + transport intelligence)
│   ├── memory-manager/SKILL.md             # Travel profile management
│   ├── feedback/SKILL.md                   # Post-trip feedback (learning engine)
│   ├── research-pipeline/SKILL.md          # 3-stage quality pipeline (auto)
│   ├── guardrails/SKILL.md                 # Anti-hallucination rules (auto)
│   ├── persistent-memory/SKILL.md          # Memory integration (auto)
│   ├── price-monitor/SKILL.md              # Pre-booking price monitoring (v2.0)
│   ├── trip-reminders/SKILL.md             # Pre-trip reminders (auto, v2.0)
│   ├── trip-pack/SKILL.md                  # Trip pack generation (v2.0)
│   ├── booking-detection/SKILL.md          # Gmail detection (auto, v2.0)
│   └── calendar-integration/SKILL.md       # Calendar events (auto, v2.0)
├── commands/
│   ├── travel-help.md                      # /travel-help quick reference
│   └── travel-expenses.md                  # /travel-expenses expense report (v2.2)
├── LICENSE
└── README.md
```

---

## Try It Out

After installation and `/onboarding`, test with:

> Plan a trip to Paris for me and my two kids (ages 5 and 8). We're in London. Mid-May, 4 nights. I want the entire trip planned out.

You should see: Eurostar recommended with a **Why this?** reasoning block, real Airbnb listings with reasoning, TfL connection from St Pancras, a day-by-day itinerary with kid-friendly activities, loyalty programme applied, and practical info — all timestamped and source-attributed.

**More tests:**

- **Reasoning transparency:** "Why did you suggest that hotel?" → see the full score breakdown
- **Loyalty intelligence:** "/loyalty-manager add" → add your BA Executive Club, then search flights
- **Document scanning:** Upload a photo of a boarding pass or hotel receipt
- **Price reshop:** "I booked that easyJet flight" → automatic post-booking monitoring starts
- **Expense report:** "/travel-expenses" after uploading some receipts
- **Challenge a recommendation:** "Why didn't you suggest the Hilton?"
- **Agent Teams:** "Plan a trip to Punta Secca in Sicily, mid-May, 4 people, direct flights from London"
- **Price monitoring:** "Watch the easyJet flight price for me"
- **Persistent memory:** "What do you remember about my travel preferences?"
- **Trip pack:** "Send me my trip pack"

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
