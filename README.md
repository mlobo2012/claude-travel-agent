# AI Heroes Travel Agent

An intelligent travel plugin for Claude Code and Claude Cowork that turns travel planning into an ongoing decision system, not a one-shot search.

Built by [AI Heroes](https://www.ai-heroes.co).

## Key Features

- Multi-modal trip planning across flights, trains, ferries, accommodation, local transit, and activities
- Preference-aware recommendations with explicit "Why this?" reasoning on every shortlist
- Flexible date search that can scan windows like "mid-May" and show price spread, not just one date
- Loyalty-aware planning, including programme tradeoffs, status benefits, and post-booking re-shop logic
- Gmail, document, and calendar workflows for turning bookings into actions
- Dispatch-ready reminders and price alerts when Cowork scheduled tasks are enabled

## What Makes This Different

Most travel tools are good at retrieving options. They are weak at judgment. They can show ten flights, twenty Airbnbs, or a generic itinerary, but they usually make you do the hard part yourself: weighing awkward departure times against price, balancing toddler constraints against transport mode, deciding whether loyalty is worth the premium, or figuring out whether a "drop" is actually worth rebooking after fees.

This plugin is built around that intelligence layer. It carries a travel profile, remembers your patterns, applies family and loyalty context, and explains its recommendations. It is designed to reason across constraints at the same time: dates, fare rules, travel time, amenities, child policies, status benefits, and what you have said matters more than price. That is the difference between search and advice.

It also aims to stay useful after the booking moment. The model is not just "find me a flight". It is "run my travel system". That means tracking shortlisted options, turning inbox confirmations into calendar actions, preparing pre-trip reminders, and helping you decide when to act. Where Claude and Cowork support background automation, the plugin can plug into that. Where they do not, the README below says so plainly.

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

### Dispatch Setup

Dispatch is Claude Code and Cowork's notification mechanism for proactive messages that arrive without you manually re-prompting in the moment. In practice, this plugin uses Dispatch for price alerts, pre-trip reminders, packing nudges, and weekly price summaries when the surrounding Claude product supports scheduled execution.

To enable it in Cowork or the Claude app, turn on Dispatch in Claude settings for the app you are using.

Example Dispatch alert:

```text
Price Alert: BA LHR→FCO
Current: £298 (was £335)
Net saving after cancellation fee: £22
Action: Rebook before 18 April to lock in the lower fare
```

## Use Cases

### Planning & Search

> "I'm flexible around mid-May, 5-7 nights. What are the best-value dates to fly London to Sicily without red-eye flights or departures before 7am? Show me the price spread across the month so I can pick."

The plugin treats this as a flexible-date planning problem, not a single-date search. It searches across the full mid-May window, removes unsociable departures, compares the valid combinations, and presents a price spread so you can see the cheap pockets of the month at a glance before committing to exact dates.

> "We're going to Lake Como in June. My wife is vegetarian, our toddler needs a cot, and we want somewhere with a washing machine for a week. Find Airbnbs near Varenna under €150/night with those filters applied."

This is where the plugin acts like an actual selector. It applies all the constraints together, family setup, cot requirement, washing machine, budget cap, location, and trip length, then explains why each shortlisted property survived the filter and what compromises remain. That is materially different from handing back generic listings.

> "Compare flying vs taking the train from London to Amsterdam next Friday. Factor in that I have Eurostar loyalty, my toddler rides free on trains, and I care more about total door-to-door time than the ticket price."

Instead of comparing headline fares, the plugin reasons across the trip. It accounts for airport transfer and check-in friction, city-centre arrival on Eurostar, toddler fare treatment, and your existing loyalty position, then gives a verdict based on your stated objective, door-to-door time over raw ticket price.

### Price Intelligence

> "Watch BA548 LHR-FCO for me. I booked at £335 but prices were dropping. Alert me via Dispatch if it falls below £280, and factor in BA's cancellation fee so I know the actual net saving."

The important part is not the number drop, it is whether the drop is actionable. The plugin stores the booked baseline, compares later prices against your threshold, and frames the alert as net value after cancellation or change-cost friction. That turns a noisy fare movement into a decision.

> "I've been watching three shortlisted flights for Barcelona in late June. Give me a summary of how the prices have moved this week and tell me which one is heading in the right direction."

This is summary judgment, not scraping. The plugin gathers the tracked items, reports their movement direction, and explains which candidate is improving, deteriorating, or holding steady so you can decide whether to buy now or keep waiting.

### Booking & Calendar

> "I just forwarded you my Eurostar booking confirmation from my inbox. Add it to my calendar including travel time from my home to St Pancras, and remind me 3 days before to check whether I need to print my tickets."

The plugin can parse the booking details, create the actual travel event, and also create a separate "travel to station" block using home-location context and default or TfL-derived journey time. It then prepares the reminder workflow so the booking becomes operational, not just stored.

> "I booked the Marriott Venice. My Bonvoy Platinum status should get me something, what upgrade or benefit should I ask for at check-in, and is there a better Marriott property in Venice where Platinum carries more weight?"

This is the loyalty-intelligence layer. The plugin looks at status entitlements for the booked brand and compares nearby alternatives where the same status might convert into more meaningful value. That gives you a practical recommendation on whether to keep the booking and what to ask for.

### The Live Travel System

> "Set up my travel system for the summer. I've shortlisted 4 flights and 3 Airbnbs. Watch them all, send me a Dispatch alert if anything drops more than 10%, give me a weekly summary on Sunday morning, and remind me 48 hours before any booking window closes."

This prompt turns the plugin from a finder into a travel operations layer. It records the watched items, thresholds, and reminder rules, then maps them onto whatever scheduled execution Claude currently supports. In Cowork with scheduled tasks configured, that can become a real recurring workflow. In plain Claude Code, it is not a native always-on daemon, and the limitation is documented below.

> "I've got a flight to Rome next week. Sort out everything I need before I go: packing list for 5 nights in May, check the weather, remind me to check in 24h before, and give me the TfL journey to Heathrow."

The plugin breaks this into coordinated prep work: weather-aware packing guidance, check-in reminder logic, and ground transport planning from the user's saved home context. The point is that these tasks are connected, so the answer should feel like one travel system doing the thinking.

## Flexible Date Search

When you ask for a vague date window like "mid-May" or "the first two weeks of June", the plugin should interpret that as a multi-date search across the requested span instead of forcing a single departure date. It can then apply additional filters such as no red-eyes, no departures before 7am, preferred trip length, or direct flights only, and present the resulting price spread so you can choose the best-value window.

## How Persistence Works

The plugin uses a dual-storage pattern in Claude Code:

- Primary: `${CLAUDE_PLUGIN_DATA}` files such as `travel-profile.json` and watched-item files
- Backup: Claude memory summaries when useful as a fallback

That is reliable enough for repeated use in Claude Code when the same plugin environment is available. In Cowork, file persistence is less dependable and should not be treated as a long-term database. The practical rule is simple: same environment and same project context tend to work best.

## Scheduled Tasks And Live Monitoring

The live travel system concept is real: the plugin is designed to support price watches, recurring summaries, and pre-trip reminders so travel help continues after the initial chat.

The current platform reality is narrower than the ambition:

- Cowork has native scheduled tasks and Dispatch, and Anthropic's help docs say they are created via `/schedule` or the Scheduled sidebar.
- Those Cowork tasks only run while the Claude desktop app is open and the computer is awake.
- Claude Code plugin skills do not create an operating-system cron job by themselves.
- This repo does not currently ship a separate background worker or cron installer.
- In this session, the requested CLI validation run could not complete because the Claude CLI returned `You've hit your limit · resets 6pm (Europe/London)` before producing a runtime answer.

So the honest current state is: scheduled monitoring is a Cowork workflow when scheduled tasks are enabled there. It is not a guaranteed always-on background service from the plugin alone, and in plain Claude Code you should assume you may need to re-prompt or explicitly set up Cowork scheduling rather than expecting a native twice-daily daemon.

## Known Limitations

- Live prices depend on the MCP servers and provider availability in that session.
- Some booking links are best-effort because operators change URL structures.
- Flexible-date calendar-style price spread depends on Claude actually executing the wider search correctly, it is an intelligence pattern, not a dedicated calendar UI component.
- Dispatch and recurring monitoring depend on product support around the plugin, especially Cowork scheduled tasks.
- Cowork scheduled tasks are not equivalent to an external always-on cron service.
- Cross-session persistence is stronger in Claude Code than in Cowork.

## Getting the Most Out of This Plugin?

If you need help with setup, want it customised to your travel workflow, or just want to talk through what's possible, reach out to [AI Heroes](https://www.ai-heroes.co/contact). We're building these tools to be as valuable as possible and your input drives that.

## License

[Apache 2.0 + Commons Clause](LICENSE) - free for private and internal business use. Commercial resale is not permitted.

## Version History

- `2.5.0` (2026-04-03): README rewrite, Dispatch/setup clarification, scheduled-task honesty, calendar travel-time logic, Commons Clause Apache license
- `2.4.1` (2026-04-03): booking links, persistence architecture, cleaner README
- `2.3.1` (2026-03-28): persistence fixes and richer onboarding
- `2.2.0` (2026-03-15): price re-shop, loyalty intelligence, document scanning
- `2.1.0` (2026-03-01): multi-modal transport, family-aware routing, route intelligence
- `2.0.0` (2026-02-15): persistent memory, price monitoring, reminders, Gmail and calendar support
