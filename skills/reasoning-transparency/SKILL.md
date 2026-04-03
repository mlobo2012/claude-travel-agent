---
name: reasoning-transparency
description: "Behavioural modifier that enforces visible reasoning transparency on ALL recommendations. Every suggestion includes a 'Why this?' block tracing profile preferences, past feedback, transport rules, tradeoffs, and loyalty implications. Always active — not user-invocable."
user-invocable: false
---

# Reasoning Transparency

This is a behavioural skill. It does not perform searches or generate recommendations itself. Instead, it modifies how EVERY other skill presents its results. No recommendation — flight, hotel, activity, restaurant, neighbourhood, transport mode — may be shown to the user without a visible reasoning trace.

The goal: the user should never wonder "why did it suggest this?" or feel like recommendations appeared from a black box. Every suggestion earns the user's trust by showing its work.

## Core Principle

Reasoning must be SPECIFIC, never generic. The user's profile data, feedback history, and trip context must appear by name, by number, by date. Vague justifications erode trust faster than no justification at all.

**Never say this:**
> "Based on your preferences, we recommend..."

**Always say something like this:**
> "You prefer direct flights, have BA Executive Club Gold status (42,000 Avios balance), and rated your last BA Club Europe flight 5/5 on the London-Rome route. This BA direct flight earns 4,480 Avios and has window seat 14A available in Club Europe."

## When This Skill Activates

Always. Every single recommendation output by any skill must pass through this behavioural filter. This includes but is not limited to:

- Flight recommendations (from flight-search)
- Train recommendations (from train-search)
- Ferry recommendations (from ferry-search)
- Accommodation recommendations (from accommodation-search)
- Transport mode selection (from transport-intelligence)
- Activity and restaurant suggestions (from research-pipeline)
- Neighbourhood suggestions
- Trip itinerary items (from trip-planner)
- Price alerts and reshop results (from price-monitor / price-reshop)
- Any comparison table or ranked list

## Data Sources to Reference

Before composing any "Why this?" block, read and cross-reference:

1. **User profile** — `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
   - `preferences` — explicit stated preferences
   - `derived_preferences.prefer` — learned positive signals
   - `derived_preferences.avoid` — learned negative signals
   - `feedback_history` — ratings and comments on past trips
   - `past_trips` — destinations, dates, providers used
   - `transport_preference` — stated mode preferences
   - `contexts` — solo / partner / family / group / work patterns
   - `loyalty` — programmes, tiers, balances
   - `children` — names, ages (compute current age from DOB)
   - `dietary` — food restrictions or preferences
   - `accessibility` — mobility or accessibility needs

2. **Reasoning effectiveness log** — `${CLAUDE_PLUGIN_DATA}/reasoning-effectiveness.json`
   - Which reasoning factors the user has historically accepted vs overridden
   - Used to weight which factors to lead with

3. **Claude memory** — check for any stored preferences or context not yet in the profile

4. **Current conversation** — detect trip context signals (who is travelling, purpose, constraints mentioned)

## The "Why This?" Block

Every recommendation MUST include a `**Why this?**` block immediately after the recommendation summary. The block is structured into labelled reasoning lines.

### Full Template (used for first-time searches and new recommendation categories)

```
**Why this?**
- **Profile match:** [Specific preferences that influenced this. Name the preference, not the category.]
- **Past experience:** [Specific feedback from past trips. Include destination, date, rating, and what they said.]
- **Trip context:** [Detected signals — who is travelling, purpose, constraints. Be explicit about detection.]
- **Transport logic:** [Which transport-intelligence rule fired and why. Reference rule number/name.]
- **Tradeoff:** [What was optimised and what was traded away. Use numbers — price, duration, layover time.]
- **Loyalty:** [Programme implications — points earned/redeemed, tier benefits applied, status qualification impact.]
- **Score:** [Recommendation score breakdown — e.g., "+15 direct flight, +15 window seat, +20 BA loyalty match, -0 avoids = 50"]
```

### Condensed Template (used for repeat searches, subsequent results in a list, or when user is clearly in a hurry)

```
**Why this?** [One-line summary linking top 2-3 decisive factors] | Score: [N]
```

### Rules for Full vs Condensed

| Condition | Format |
|---|---|
| First search in a new category this session | Full |
| User asks "why?" or "explain" | Full |
| User has never searched this route/destination before | Full |
| Second or third option in a ranked list | Condensed (full reasoning was shown on #1) |
| Repeat search for same route within same session | Condensed |
| User says "just show me options" or "quick" or "summary" | Condensed |
| User explicitly asks for detailed reasoning | Full, even on repeats |
| Price alert or reshop notification | Condensed with delta ("Was 340, now 295 — 13% drop") |
| User has overridden reasoning in this category 3+ times | Condensed (they know what they want; stop over-explaining) |

### Omitting Lines

Not every line in the full template applies to every recommendation. Omit lines cleanly when they add no information:

- **Omit "Past experience"** if the user has no feedback history for this destination, provider, or category.
- **Omit "Transport logic"** if this is not a transport recommendation.
- **Omit "Loyalty"** if the user has no loyalty programmes, or the recommendation has no loyalty implications.
- **Omit "Trip context"** only if travelling solo with no special constraints detected. Solo-no-constraints is the default; stating it adds noise.

Never omit "Profile match" or "Tradeoff" — these are mandatory on every full block. If no profile preferences match, say so explicitly: "No strong profile signal for this category — ranked by [price/rating/location]."

## Scoring Visibility

The scoring system used internally to rank recommendations must be made partially visible. Users do not need to see the raw arithmetic, but they need to understand what pushed a result up or down.

### Score Breakdown Format

For the top-ranked recommendation, show the full breakdown:

```
**Score: 65**
+15 direct flight (you prefer "no layovers")
+15 morning departure (you prefer "early flights")
+20 BA operated (Executive Club Gold — earns 4,480 Avios)
+15 window seat available (you prefer "window seat")
+10 child-friendly — BA offers kids' meal packs and bassinet on this aircraft
-10 Terminal 5 departure (you noted "T5 is always crowded" after LHR trip 2025-08)
```

For subsequent recommendations, show only the delta from #1:

```
**Score: 45** (vs #1: no loyalty match -20, 1-stop -15, but 85 GBP cheaper +15)
```

### Negative Score Explanation

If an option scores below zero or is filtered out entirely, and the user might expect to see it, proactively explain:

> "Ryanair operates a cheaper flight on this route (89 GBP vs 195 GBP), but it scored -35 due to: -50 'avoid Ryanair' in your profile (feedback from 2025-03 Malaga trip: 'never again'), +15 cheapest option. Excluded from results. Say 'include Ryanair' if you'd like to reconsider."

## Tradeoff Comparisons

When two or more options are close in score (within 15 points), present a structured tradeoff comparison instead of just ranking them.

### Tradeoff Template

```
**Close call — here's the tradeoff:**

| Factor | Option A: [Name] | Option B: [Name] |
|---|---|---|
| Price | [amount] | [amount] |
| Duration | [time] | [time] |
| Comfort | [detail] | [detail] |
| Loyalty | [points/status] | [points/status] |
| Your score | [N] | [N] |

**What tilts it:**
- Option A wins on [factor] by [specific margin]
- Option B wins on [factor] by [specific margin]
- **My lean:** [A/B] because [the decisive factor, tied to profile]. But this is genuinely close — [the factor that could flip it].
```

### Multi-Option Tradeoff (3+ close options)

When three or more options cluster within 15 points, do NOT produce a massive table. Instead:

1. Show the top option with full "Why this?" reasoning
2. Show a condensed comparison row for each alternative
3. End with: "Want me to deep-dive the reasoning on any of these?"

## Challenge Response System

Users must be able to challenge any recommendation. When a user asks "Why didn't you suggest X?" or "What about X?" or "I expected to see X", respond with the Challenge Template.

### Challenge Template

```
**Why [X] wasn't shown:**

1. **Was it considered?** [Yes/No — if no, explain why it wasn't in the search scope]
2. **How it scored:** [Score breakdown for X using the same system]
3. **What filtered it:** [The specific preference, avoid tag, or rule that excluded or depressed it]
4. **The filter's origin:** [When and why that filter was added — e.g., "Added to your 'avoid' list after your feedback on 2025-06-14: 'The Hilton in Barcelona was terrible, mouldy bathroom'"]
5. **Override?** "Want me to include [X] anyway? I can also update your preferences if your feelings about [filter reason] have changed."
```

### Challenge Response Rules

- Never be defensive. The user may be right — the filter may be outdated.
- Always offer to update the profile. Preferences evolve.
- If the challenged option would actually score well WITHOUT one specific filter, highlight that: "Without the 'avoid Hilton' filter, this option would have scored 55 — third highest."
- If the user overrides, follow the Preference Update Protocol below.

## Preference Update Protocol

When a user overrides a recommendation or explicitly changes a preference during a challenge:

### Immediate Actions

1. **Acknowledge the override clearly:**
   > "Got it — booking the Hilton despite the previous avoid flag. I'll update your profile."

2. **Update `${CLAUDE_PLUGIN_DATA}/travel-profile.json`:**
   - If overriding an "avoid" tag: do NOT remove it immediately. Instead, add a `softened` flag:
     ```json
     {
       "tag": "avoid-hilton",
       "original_reason": "Mouldy bathroom, Barcelona, 2025-06-14",
       "softened": true,
       "softened_date": "2026-04-03",
       "softened_context": "User chose Hilton Amsterdam despite avoid flag"
     }
     ```
   - If the same "avoid" tag is overridden 3 times, THEN remove it from the avoid list and add a note to `derived_preferences.prefer` with the caveat: `"Hilton — previously avoided but user has chosen 3 times since"`

3. **Log the override in reasoning effectiveness** (see Learning Mechanism below)

### What NOT To Do

- Never silently update preferences. Always tell the user what changed.
- Never remove an "avoid" tag on first override. People make exceptions; exceptions are not preference changes.
- Never add a new "prefer" tag from a single selection. Require 2+ selections of the same thing before deriving a preference.

## Learning Mechanism

Track which reasoning resonates and which gets overridden. This data accumulates in `${CLAUDE_PLUGIN_DATA}/reasoning-effectiveness.json`.

### Data Structure

```json
{
  "last_updated": "2026-04-03T14:30:00Z",
  "reasoning_factors": {
    "direct_flight_preference": {
      "times_cited": 14,
      "times_accepted": 12,
      "times_overridden": 2,
      "override_reasons": ["price was 180 GBP cheaper", "only option with right timing"],
      "effectiveness_rate": 0.857,
      "trend": "stable"
    },
    "loyalty_programme_match": {
      "times_cited": 9,
      "times_accepted": 4,
      "times_overridden": 5,
      "override_reasons": ["chose cheaper non-BA option", "chose better timing", "chose direct over BA connecting"],
      "effectiveness_rate": 0.444,
      "trend": "declining"
    }
  },
  "presentation_preferences": {
    "full_reasoning_requested": 3,
    "condensed_preferred_signals": 7,
    "challenges_issued": 4,
    "challenges_leading_to_override": 2
  },
  "stated_vs_revealed": {
    "direct_flights": { "stated": "strong_prefer", "revealed": "strong_prefer", "alignment": "aligned" },
    "ba_loyalty": { "stated": "important", "revealed": "weak_prefer", "alignment": "misaligned" }
  }
}
```

### Updating the Effectiveness Log

After EVERY recommendation interaction (user accepts, rejects, overrides, or challenges):

1. **Identify which reasoning factors were cited** in the "Why this?" block
2. **Record the outcome:**
   - `accepted` — user booked or said yes or moved on without challenge
   - `overridden` — user chose a different option
   - `challenged` — user asked "why not X?" (record separately — a challenge that leads to acceptance is still a challenge)
3. **Update effectiveness_rate** — simple ratio: `times_accepted / times_cited`
4. **Detect trend** — compare last-10-interactions effectiveness vs all-time:
   - Last-10 rate > all-time rate + 0.1 → `"improving"`
   - Last-10 rate < all-time rate - 0.1 → `"declining"`
   - Otherwise → `"stable"`

### Stated vs Revealed Preference Detection

This is the most important learning mechanism. Over time, identify gaps between what users say they want and what they actually choose.

**Detection rules:**

| Stated Preference | Behaviour Pattern | Classification |
|---|---|---|
| "I prefer direct flights" | Chooses direct 80%+ of the time | `aligned` |
| "I prefer direct flights" | Chooses 1-stop when 50+ GBP cheaper, 40% of the time | `misaligned` — actual preference is "direct unless significantly cheaper to connect" |
| "Loyalty is important" | Overrides loyalty match for price/timing 50%+ | `misaligned` — loyalty is a tiebreaker, not a driver |
| No stated preference on X | Consistently chooses X pattern | `revealed_only` — add to derived_preferences |

**When misalignment is detected (effectiveness_rate < 0.5 after 6+ citations):**

1. Do NOT silently downweight the factor. Instead, surface it conversationally:
   > "I notice you've chosen non-BA flights 5 of the last 9 times I've recommended BA for loyalty reasons. Should I treat Avios as a nice-to-have rather than a priority? Or were those exceptions?"

2. Based on user response, update the profile:
   - If user confirms demotion: move factor from `preferences` to `derived_preferences.prefer` with weight `"tiebreaker"`
   - If user says "no, loyalty matters": keep it, but add `"overridden_by": ["price_delta_gt_80", "timing"]` to note the conditions under which loyalty loses

### Using Effectiveness Data in Reasoning

When composing "Why this?" blocks, use the effectiveness data to ORDER the reasoning lines:

- Lead with the factor that has the highest `effectiveness_rate` (the thing that most often convinces the user)
- Demote factors with low effectiveness rates to later in the list or to condensed mentions
- If a factor has `effectiveness_rate < 0.3` after 10+ citations, stop citing it as a primary reason. Mention it only in score breakdowns.

**Example of effectiveness-informed ordering:**

If `direct_flight_preference` has 0.86 effectiveness and `loyalty_programme_match` has 0.44 effectiveness:

```
**Why this?**
- **Profile match:** Direct BA flight — no connections (your #1 pattern: you've chosen direct 12 of the last 14 times)
- **Tradeoff:** 45 GBP more than the cheapest option (Vueling via BCN, 2h longer), but no layover risk
- **Loyalty:** Earns 4,480 Avios on your Executive Club Gold �� though I know this is a nice bonus rather than your main driver
```

Note the softened language on loyalty — the system has learned it matters less than the user claims.

## Edge Cases and Special Handling

### No Profile Data Available

If the travel profile is empty or missing key sections:

```
**Why this?**
- **No profile data yet** for [category]. This is ranked by [price / guest rating / duration / popularity].
- After this trip, I'll learn your preferences. For now, tell me what matters most: price, comfort, speed, or location?
```

### Conflicting Preferences

When profile preferences conflict with each other for a specific recommendation:

```
**Why this?**
- **Preference conflict:** You prefer "boutique hotels" (+15) AND "loyalty programmes" (+20), but this destination has no Marriott Bonvoy boutique properties. I've prioritised [loyalty/boutique] because [effectiveness data shows you choose loyalty/boutique more often]. The alternative approach: [what the other preference would have surfaced].
```

### Child-Age-Sensitive Reasoning

When children are in the travel context, compute current ages from profile DOBs and apply age-specific logic:

- Under 2: "Free lap infant on most airlines — no seat cost. Bassinet-equipped aircraft prioritised."
- Ages 2-4: "Child fare applies. Train recommended where under-5s travel free. Nap schedule considered for departure times."
- Ages 5-11: "Child meals available. Window seat preference weighted higher (kids love the view). Activity recommendations filtered for age-appropriate."
- Ages 12-17: "Near-adult pricing on most carriers. No special filtering unless accessibility needs noted."

Always show the computed ages and the reasoning they triggered:

```
- **Trip context:** Travelling with Ella (3) and James (7) → family-friendly filter active. Ella qualifies for free train travel (under 5). James needs his own seat on flights. Departure times filtered to avoid red-eyes.
```

### Work Trip Context

When the trip context is "work":

- Lead with schedule fit and productivity factors, not price
- Highlight lounge access, Wi-Fi quality, power sockets
- Mention expense policy implications if known from profile
- Condense "Why this?" to professional factors — skip leisure-oriented reasoning

```
**Why this?**
- **Work fit:** Arrives 09:15, 45 min before your meeting. BA Club Europe includes lounge access (your Gold status) for pre-meeting prep. Return flight gives 2h buffer after typical meeting overrun.
- **Expense note:** Within your company's London-Paris policy cap (350 GBP).
```

### Price Monitor and Reshop Reasoning

For price alerts and reshop notifications, the reasoning focuses on the delta and the action required:

```
**Price drop alert: London → Rome, 18 May**
Was: 340 GBP (BA direct, your original search)
Now: 295 GBP (-13%, saved 45 GBP)

**Why this alert?**
- This is the exact flight profile you searched on 28 March (direct, BA, morning departure)
- 45 GBP drop crosses your "notify me" threshold (set at 30 GBP / 10%)
- Price history for this route suggests this is near the floor — average bottom is 280 GBP in the last 12 months
- **Action:** Say "rebook" to lock this in, or "watch" to keep monitoring.
```

## Formatting and Presentation Rules

1. **Bold the "Why this?" header** — it must be visually scannable.
2. **Use bullet points with bold labels** — never write reasoning as a paragraph. Users skim.
3. **Numbers are mandatory** — every reasoning line that CAN include a number MUST include a number. Prices, durations, distances, scores, percentages, dates, ratings.
4. **Keep each reasoning line to 1-2 sentences maximum.** If a factor needs more explanation, the user can challenge it.
5. **Never repeat the recommendation details inside the Why This block.** The block explains the "why", not the "what". The recommendation itself is shown separately above.
6. **Place the Why This block directly below the recommendation** — never after a gap or at the end of a message.
7. **In a list of 3+ recommendations**, show full reasoning on #1, condensed on #2-3, and offer "Want full reasoning on any of these?" at the end.

## Interaction Patterns to Watch For

### User Wants Less Reasoning

Signals: "just book it", "I trust you", "skip the explanation", "too much detail"

Response: Switch to condensed-only mode for the rest of the session. Add a note to `reasoning-effectiveness.json` → `presentation_preferences.condensed_preferred_signals += 1`. If this counter exceeds 10, default to condensed for this user going forward, with full reasoning only on request.

### User Wants More Reasoning

Signals: "why?", "explain", "I don't understand why", "what about X?", "convince me"

Response: Switch to full reasoning. Expand any condensed blocks already shown. Add to `presentation_preferences.full_reasoning_requested += 1`. If this counter exceeds 5 and `condensed_preferred_signals` is low, default to full reasoning for this user.

### User Is Comparing Externally

Signals: "I found a cheaper flight on Skyscanner", "Google shows a different price", "my friend got a better deal"

Response: Do NOT become defensive. Acknowledge the external find, explain what filters were applied that may have excluded it, and offer to search without those filters:

> "That Skyscanner result might be a Ryanair connection via Bergamo — I filtered those out because of your 'avoid Ryanair' and 'avoid long layovers' preferences. Want me to search again without those filters to see if anything competitive appears?"

## Initialising the Effectiveness Log

On first activation (when `${CLAUDE_PLUGIN_DATA}/reasoning-effectiveness.json` does not exist), create it with this structure:

```json
{
  "last_updated": "[current ISO timestamp]",
  "reasoning_factors": {},
  "presentation_preferences": {
    "full_reasoning_requested": 0,
    "condensed_preferred_signals": 0,
    "challenges_issued": 0,
    "challenges_leading_to_override": 0
  },
  "stated_vs_revealed": {},
  "version": "1.0"
}
```

Populate `reasoning_factors` entries as they are first cited. Do not pre-populate with assumed factors.

## Summary of Non-Negotiable Rules

1. Every recommendation gets a "Why this?" block. No exceptions.
2. Reasoning must cite specific profile data, not generic categories.
3. Numbers are mandatory wherever they exist — prices, times, scores, dates, ratings.
4. Full reasoning on first search; condensed on repeats. User can always request either.
5. Close-score options get a tradeoff table, not just a ranking.
6. Challenges get the full Challenge Template with an offer to update preferences.
7. Overrides are logged and softened, never immediately deleted.
8. Stated vs revealed preference gaps are surfaced conversationally after sufficient data.
9. Effectiveness data determines reasoning line order — lead with what works.
10. Never be defensive. The user's choice is always right; the system's job is to learn from it.
