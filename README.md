# AI Heroes Travel Agent

An intelligent travel plugin for Claude Code and Claude Cowork that turns travel planning into an ongoing decision system, while staying honest about what recurring monitoring is actually supported today.

Built by [AI Heroes](https://www.ai-heroes.co).

## Key Features

- Multi-modal trip planning across flights, trains, ferries, accommodation, local transit, and activities
- Preference-aware recommendations with explicit "Why this?" reasoning on every shortlist
- Flexible date search that can scan windows like "mid-May" and show price spread, not just one date
- Loyalty-aware planning, including programme tradeoffs, status benefits, and post-booking re-shop logic
- Gmail, document, and calendar workflows for turning bookings into actions
- Local scheduled-task monitoring for price watches and reminders when Claude Desktop scheduling is enabled

## What Makes This Different

Most travel tools are good at retrieving options. They are weak at judgment. They can show ten flights, twenty Airbnbs, or a generic itinerary, but they usually make you do the hard part yourself: weighing awkward departure times against price, balancing toddler constraints against transport mode, deciding whether loyalty is worth the premium, or figuring out whether a "drop" is actually worth rebooking after fees.

This plugin is built around that intelligence layer. It carries a travel profile, remembers your patterns, applies family and loyalty context, and explains its recommendations. It is designed to reason across constraints at the same time: dates, fare rules, travel time, amenities, child policies, status benefits, and what you have said matters more than price. That is the difference between search and advice.

It also stays useful after the booking moment. The model is not just "find me a flight". It is "run my travel system". That means tracking shortlisted options, turning inbox confirmations into calendar actions, preparing pre-trip reminders, and helping you decide when to act. The recurring monitoring architecture in this repo is grounded in Claude Desktop local scheduled tasks, not speculative hidden APIs.

## Setup

### Install

**Claude Code**
```bash
claude plugin install https://github.com/mlobo2012/claude-travel-agent
```

**Local repo**
```bash
git clone https://github.com/mlobo2012/claude-travel-agent.git
claude plugin install ./claude-travel-agent
```

**Claude Cowork**
1. Download or clone the repo.
2. Zip the plugin contents with `.claude-plugin/`, `skills/`, and `.mcp.json` at the zip root.
3. Open Cowork, go to Plugins, then upload the zip from the Personal tab.

### MCP Servers

The plugin is configured for these connectors in `.mcp.json`:

- `airbnb` for Airbnb search and listing details
- `google-flights` for flight search
- `public-transport` for train and public transport routing
- `tfl` for London journey planning
- `gmail` for booking detection
- `google-calendar` for calendar event creation

### Verify

Run:

```bash
claude plugin validate .
```

Then ask Claude a simple question like: `What travel tools and skills are available?`

## Getting Started

### Onboarding

Onboarding is optional. The plugin can build a profile progressively from normal conversation, home airport, companions, loyalty numbers, transport preferences, accommodation preferences, and recurring constraints.

If you want the explicit setup flow, run `/travel-setup`.

### Supported Live Monitoring Architecture

This plugin supports robust recurring monitoring through **Claude Desktop local scheduled tasks**.

- Watch registration can happen inside the plugin conversation.
- The plugin stores watches in a local project state store under `.claude/travel-monitor/`.
- Recurring checks run through **one** local scheduled task on the user's machine, typically named `ai-heroes-travel-monitor`.
- That task should be created or managed through Claude's supported scheduling flows such as `/schedule` or the Scheduled sidebar.
- Remote tasks are not used for plugin-based monitoring because the official remote runtime docs describe a GitHub-backed remote environment, while the Cowork scheduled-task docs explicitly describe local scheduled tasks with access to installed plugins.

Example alert:

```text
Price Alert: BA LHR->FCO
Current: GBP 270 (was GBP 335)
Net saving after fees: GBP 20
Action: Rebook before the cancellation window closes
```

## Use Cases

### Planning & Search

> "I'm flexible around mid-May, 5-7 nights. What are the best-value dates to fly London to Sicily without red-eye flights or departures before 7am? Show me the price spread across the month so I can pick."

The plugin treats this as a flexible-date planning problem, not a single-date search. It searches across the full mid-May window, removes unsociable departures, compares the valid combinations, and presents a price spread so you can see the cheap pockets of the month before committing to exact dates.

> "We're going to Lake Como in June. My wife is vegetarian, our toddler needs a cot, and we want somewhere with a washing machine for a week. Find Airbnbs near Varenna under EUR 150/night with those filters applied."

This is where the plugin acts like an actual selector. It applies all the constraints together, family setup, cot requirement, washing machine, budget cap, location, and trip length, then explains why each shortlisted property survived the filter and what compromises remain.

> "Compare flying vs taking the train from London to Amsterdam next Friday. Factor in that I have Eurostar loyalty, my toddler rides free on trains, and I care more about total door-to-door time than the ticket price."

Instead of comparing headline fares, the plugin reasons across the trip. It accounts for airport transfer and check-in friction, city-centre arrival on Eurostar, toddler fare treatment, and your existing loyalty position, then gives a verdict based on your stated objective.

### Price Intelligence

> "Watch BA548 LHR-FCO for me. I booked at GBP 335 but prices were dropping. Alert me if it falls below GBP 280, and factor in BA's cancellation fee so I know the actual net saving."

The important part is not the number drop, it is whether the drop is actionable. The plugin saves the booked baseline to the local watch store, compares later prices against your threshold, and frames the alert as net value after cancellation or change-cost friction. When the local scheduled task is enabled, the recurring check runs through that task.

> "I've been watching three shortlisted flights for Barcelona in late June. Give me a summary of how the prices have moved this week and tell me which one is heading in the right direction."

This is summary judgment, not scraping. The plugin gathers the tracked items from the local watch store, reports their movement direction, and explains which candidate is improving, deteriorating, or holding steady so you can decide whether to buy now or keep waiting.

### Booking & Calendar

> "I just forwarded you my Eurostar booking confirmation from my inbox. Add it to my calendar including travel time from my home to St Pancras, and remind me 3 days before to check whether I need to print my tickets."

The plugin can parse the booking details, create the actual travel event, and also create a separate "travel to station" block using home-location context and default or TfL-derived journey time. It then saves the reminder requirement into the local watch store so it can be handled by the dispatcher task when scheduling is enabled.

> "I booked the Marriott Venice. My Bonvoy Platinum status should get me something, what upgrade or benefit should I ask for at check-in, and is there a better Marriott property in Venice where Platinum carries more weight?"

This is the loyalty-intelligence layer. The plugin looks at status entitlements for the booked brand and compares nearby alternatives where the same status might convert into more meaningful value.

### The Live Travel System

> "Set up my travel system for the summer. I've shortlisted 4 flights and 3 Airbnbs. Watch them all, send me an alert if anything drops more than 10%, give me a weekly summary on Sunday morning, and remind me 48 hours before any booking window closes."

This prompt turns the plugin from a finder into a travel operations layer. It records the watched items, thresholds, and reminder rules in `.claude/travel-monitor/`, then routes recurring execution through one Claude Desktop local scheduled task. It is not a magic always-on daemon everywhere. It becomes robust when the user enables that local scheduled task.

## Flexible Date Search

When you ask for a vague date window like "mid-May" or "the first two weeks of June", the plugin should interpret that as a multi-date search across the requested span instead of forcing a single departure date. It can then apply additional filters such as no red-eyes, no departures before 7am, preferred trip length, or direct flights only, and present the resulting price spread.

## How Persistence Works

The plugin uses a dual-storage pattern in Claude Code:

- Primary: `${CLAUDE_PLUGIN_DATA}` files such as `travel-profile.json`
- Backup: Claude memory summaries when useful as a fallback

That remains the profile-memory model. Monitoring state is separate and should live in `.claude/travel-monitor/` so the local scheduled task and the conversation share the same watch store.

## How Recurring Monitoring Works

Use it the same way you would use a real travel agent: just say what you want monitored.

For example:

> "Watch BA548 LHR-FCO for me. I booked at £335. Alert me if it drops below £280 and tell me whether it's still worth rebooking after fees."

> "Track these three Barcelona flight options for the next week and tell me which one is improving."

> "Monitor my Rome trip this week and remind me 24 hours before check-in, plus two days before departure to review airport travel time."

The Travel Agent then:

1. saves the watch details in your project
2. sets up the recurring monitor in Claude Desktop
3. re-checks the watched items on schedule
4. only alerts you when something is actually worth acting on

The first time you ask for recurring monitoring, Claude may ask you to confirm the scheduled task. After that, the same monitor can keep handling future watches in the background.

### What you need to do

- Ask for monitoring in natural language
- Approve the scheduled task if Claude asks the first time
- Keep Claude Desktop open and your machine awake when you want local monitoring to run

### What the plugin handles for you

- keeping one shared watchlist instead of lots of separate tasks
- storing baseline prices and thresholds
- checking price changes repeatedly
- suppressing noisy duplicate alerts
- focusing alerts on actionable changes, not random fare movement

## Limitations

- Live prices still depend on MCP servers and provider availability in that session.
- Some booking links are best-effort because operators change URL structures.
- Flexible-date price spread is an intelligence workflow, not a dedicated calendar UI.
- Recurring monitoring works through Claude Desktop scheduled tasks, so the app needs to be open and the machine awake.
- Cross-session persistence is stronger in Claude Code than in Cowork.

## Getting the Most Out of This Plugin?

If you need help with setup, want it customised to your firm's workflows, or just want to talk through what's possible, reach out to [AI Heroes](https://www.ai-heroes.co/contact). We're building these tools to be as valuable as possible and your input drives that.

## License

[Apache 2.0 + Commons Clause](LICENSE) - free for private and internal business use. Commercial resale is not permitted.

## Version History

- `2.6.0` (2026-04-03): local scheduled-task monitoring architecture, project-local watch store, single-dispatcher pattern, README alignment
- `2.5.0` (2026-04-03): README rewrite, Dispatch/setup clarification, scheduled-task honesty, calendar travel-time logic, Commons Clause Apache license
- `2.4.1` (2026-04-03): booking links, persistence architecture, cleaner README
- `2.3.1` (2026-03-28): persistence fixes and richer onboarding
- `2.2.0` (2026-03-15): price re-shop, loyalty intelligence, document scanning
- `2.1.0` (2026-03-01): multi-modal transport, family-aware routing, route intelligence
- `2.0.0` (2026-02-15): persistent memory, price monitoring, reminders, Gmail and calendar support
