# AI Heroes Travel Agent

An autonomous, learning travel agent plugin for **Claude Code** and **Claude Cowork**. Plans trips end-to-end with live-verified prices, anti-hallucination guardrails, and preference learning.

Built by [AI Heroes](https://www.ai-heroes.co) — AI That Learns How You Win.

---

## What It Does

- **Flight search** via Google Flights MCP — live prices, direct booking links
- **Accommodation search** via Airbnb MCP — live prices, review scores, amenities
- **Full trip planning** — flights + accommodation + day-by-day itinerary + practical info
- **Anti-hallucination guardrails** — every price is live-verified, timestamped, and source-attributed. No rounding, estimating, or fabricating prices. Ever.
- **Preference learning** — 10-question onboarding builds a profile that improves every search
- **Post-trip feedback** — learns from your experiences to refine future recommendations
- **Cross-checking** — prices verified across multiple sources when possible

---

## Installation

### Option 1: From GitHub (Claude Code CLI)

```bash
/plugin install ai-heroes-travel-agent@mlobo2012-claude-travel-agent
```

Or add the marketplace first, then install:

```bash
/plugin marketplace add https://github.com/mlobo2012/claude-travel-agent
/plugin install ai-heroes-travel-agent
```

After installing, run `/reload-plugins` to activate.

### Option 2: Claude Cowork Desktop App

1. Download this repository as a ZIP (green **Code** button > **Download ZIP**)
2. Open the Claude Cowork desktop app
3. Go to **Browse Plugins** > **Personal** > click the **"+"** button
4. Select the downloaded ZIP file
5. The plugin will appear under **Personal** plugins

### Option 3: Local Development

```bash
git clone https://github.com/mlobo2012/claude-travel-agent.git
claude --plugin-dir ./claude-travel-agent
```

---

## MCP Server Setup

This plugin uses two MCP servers for live data. They are configured in `.mcp.json` and run automatically via `npx` — no global installation needed. You just need **Node.js 18+** with `npx` in your PATH.

### Airbnb — `@openbnb/mcp-server-airbnb`

- **npm:** [`@openbnb/mcp-server-airbnb`](https://www.npmjs.com/package/@openbnb/mcp-server-airbnb)
- **What it does:** Searches Airbnb listings, retrieves property details, checks availability
- **API key:** None required
- **License:** MIT (not affiliated with Airbnb, Inc.)

### Google Flights — `google-flights-mcp-server`

- **npm:** [`google-flights-mcp-server`](https://www.npmjs.com/package/google-flights-mcp-server)
- **What it does:** Searches Google Flights for routes, prices, schedules, and date grids
- **API key:** None required

### Verifying MCP Servers

After installing the plugin, check that the MCP servers are connected:

```
# In Claude Code, ask:
"What MCP tools are available?"

# You should see tools from both airbnb and google-flights servers
```

If servers fail to start, ensure Node.js 18+ and `npx` are available in your PATH.

---

## Commands

| Command | Description |
|---------|-------------|
| `/onboarding` | First-time setup — 10 questions to build your travel profile |
| `/memory-manager` | View your saved travel preferences |
| `/memory-manager update` | Update a specific preference |
| `/memory-manager history` | See your past trips |
| `/flight-search` | Search for flights with live-verified prices |
| `/accommodation-search` | Search Airbnb and Booking.com |
| `/trip-planner` | Full end-to-end trip planning |
| `/feedback` | Record post-trip feedback to improve future results |
| `/travel-help` | Quick reference for all commands |

**Tip:** Run `/onboarding` first to get personalised results from day one.

---

## How It Works

### Research Pipeline

Every search follows a 3-stage quality pipeline:

1. **Live Retrieval** — Query MCP servers for real-time data
2. **Cross-Check** — Verify prices on a second source (web search fallback)
3. **Quality Filter** — Apply minimum review thresholds, preference scoring, and present the top 3

### Anti-Hallucination Rules

- Every price is retrieved live and timestamped: *"Price as of [time] — verify at checkout"*
- No rounding, estimating, or inferring prices
- Cross-check discrepancies >5% are flagged with both values shown
- Missing data is explicitly noted, never fabricated
- Direct booking links are verified, not constructed

### Preference Scoring

Results are ranked using your profile:
- **+15 points** for each matching preference
- **-50 points** for each matching "avoid" tag
- Score below 0 = excluded from results

### Data Storage

Your data is stored locally on your machine:

- Travel profile: `~/.claude/plugins/data/ai-heroes-travel-agent/travel-profile.json`
- Trip plans: `~/.claude/plugins/data/ai-heroes-travel-agent/trips/`

---

## Verified MCP Servers

| Server | Status | npm Package | Verified |
|--------|--------|-------------|----------|
| Airbnb | Real MCP server | [`@openbnb/mcp-server-airbnb`](https://www.npmjs.com/package/@openbnb/mcp-server-airbnb) | Yes — published on npm |
| Google Flights | Real MCP server | [`google-flights-mcp-server`](https://www.npmjs.com/package/google-flights-mcp-server) | Yes — published on npm |
| Booking.com | Fallback only | N/A — uses web search for cross-checking | No MCP server available |
| Skyscanner | Fallback only | N/A — uses web search for cross-checking | No MCP server available |

---

## Try It Out

After installation and `/onboarding`, test with:

> Plan a trip to Punta Secca in Sicily, Italy, for mid-May. 4 people, walkable to the beach. Small budget for accommodation. Direct flights only from London. I need the entire trip planned out, not just flights.

You should see live flight prices from Google Flights, real Airbnb listings with nightly rates and review scores, a day-by-day itinerary, and practical info — all timestamped and source-attributed.

---

## Plugin Structure

```
claude-travel-agent/
├── .claude-plugin/
│   └── plugin.json                    # Plugin manifest
├── .mcp.json                          # MCP server configuration
├── skills/
│   ├── travel-agent/SKILL.md          # Core identity and rules
│   ├── onboarding/SKILL.md            # First-time profile setup
│   ├── accommodation-search/SKILL.md  # Airbnb + Booking.com search
│   ├── flight-search/SKILL.md         # Google Flights search
│   ├── trip-planner/SKILL.md          # End-to-end trip planning
│   ├── memory-manager/SKILL.md        # Travel profile management
│   ├── feedback/SKILL.md              # Post-trip feedback
│   ├── research-pipeline/SKILL.md     # 3-stage quality pipeline (auto)
│   └── guardrails/SKILL.md            # Anti-hallucination rules (auto)
├── commands/
│   └── travel-help.md                 # /travel-help quick reference
├── LICENSE
└── README.md
```

---

## Development

### Validate the plugin

```bash
claude plugin validate
```

### Test locally

```bash
claude --plugin-dir ./claude-travel-agent
```

---

## License

[Apache-2.0](LICENSE)

---

## Credits

Built by [AI Heroes](https://www.ai-heroes.co) — AI That Learns How You Win.

Created by Marco Lobo.
