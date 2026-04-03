---
name: travel-setup
description: "First-time travel profile onboarding. Run this when the user has never set up their travel preferences, or wants to redo their profile. Asks conversational questions to build a complete travel preference profile including transport mode preferences. Use when: user says 'set up travel', 'travel preferences', 'new traveller setup', or this is their first travel query and no profile exists."
argument-hint: ""
---

# Travel Profile Setup

You are running first-time onboarding for the AI Heroes Travel Agent. Your goal is to have a natural, conversational exchange that builds a complete travel preference profile.

## Before Starting

Check if a profile already exists at `${CLAUDE_PLUGIN_DATA}/travel-profile.json`. If it does, ask the user if they want to update it or start fresh.

## Quick Mode — When User Asks a Specific Travel Question Without a Profile

If the user is asking a specific travel question (e.g., "find me flights to Sicily") and no profile exists, **DO NOT block them with full onboarding**. Instead:
1. Ask only the 2-3 questions needed for THIS specific search (e.g., dates, budget, party size, departure airport)
2. Run the search immediately with those answers
3. After presenting results, offer full onboarding: "Want me to save your preferences for next time? Run `/travel-setup` to set up your full travel profile."

Only run the full 12-question onboarding below when the user explicitly asks for it (e.g., `/travel-setup`, "set up my travel preferences").

## Full Onboarding Flow

Ask these questions **one or two at a time** in a natural conversational flow. Don't dump all questions at once. Adapt based on answers — if someone says they only travel solo, skip group-specific questions. If they don't have children, skip child-specific questions.

### The Questions

**Q1: Home Base**
"Where do you usually fly from? (airport code or city)"
→ Stores: `home_airport`, `home_city`

**Q2: Travel Style**
"How do you usually travel — solo, with a partner, in a group, or for work? (You can pick more than one)"
→ Stores: `contexts` keys

**Q3: Transport Preferences**
"When there's a good train or ferry option for shorter routes, do you generally prefer those over flying? What matters most to you: speed, budget, comfort, or environmental impact?"
→ Stores: `transport_preference` (one of: speed, budget, comfort, eco, no_preference)

**Q4: Family Travel**
"Do you travel with children? If so, what are their ages? (This helps me find free/discounted fares on trains and ferries)"
→ Stores: `children` (array of ages), updates context-specific profiles
→ If no children: skip and move on

**Q5: Accommodation Philosophy**
"When it comes to where you stay — are you more of a hotel person, Airbnb person, or does it depend on the trip?"
→ Stores: `accommodation_type_preference`

**Q6: Budget**
"What's your comfortable per-night budget for accommodation? And what currency do you prefer?"
→ Stores: `max_nightly_budget`, `currency`

**Q7: Must-Have Amenities**
"Any non-negotiables for where you stay? (wifi, kitchen, pool, parking, washing machine, etc.)"
→ Stores: `amenities`

**Q8: Flight Preferences**
"Any flight preferences? Favourite airlines, cabin class, aisle or window?"
→ Stores: `cabin_class`, `seat_preference`, preferred airlines

**Q9: Loyalty Programmes**
"Are you in any airline, hotel, or rail loyalty programmes? If so, tell me the programme name, your membership number, and your current tier/status. I'll auto-apply these to every search and track your points and tier progress."
→ Stores: `loyalty.programmes[]` — for each programme, capture:
  - `name` (e.g., "British Airways Executive Club")
  - `type` ("airline" | "hotel" | "rail")
  - `alliance` (e.g., "oneworld", "Star Alliance", "SkyTeam" — for airlines)
  - `membership_number`
  - `tier` (e.g., "Gold", "Platinum")
  - `points_balance` (user-reported, approximate is fine)
  - `points_currency` (e.g., "Avios", "Hilton Honors Points")
  - `tier_expiry` (if known)
  - `auto_apply`: true (default — apply to searches automatically)

If the user mentions flying a specific airline frequently but has no loyalty programme, note it and suggest: "You've mentioned flying BA often — they have an Executive Club programme where you'd earn Avios on every flight. Want me to remind you to sign up?"

After onboarding, remind the user: "You can manage your loyalty programmes anytime with `/loyalty-manager`."

**Q10: Neighbourhood & Vibe**
"When you're in a new city, what vibe are you after? (central & walkable, quiet residential, beachfront, near nightlife, etc.)"
→ Stores: context-specific `neighbourhood_vibe`

**Q11: Past Trip Reference**
"Tell me about a trip you loved — and one that didn't quite work out. What made the difference?"
→ Stores: `derived_preferences`, initial `past_trips`

**Q12: Passport**
"What passport do you hold? And roughly when does it expire? (This helps me flag visa requirements)"
→ Stores: `passport`

## After All Questions

Build the complete profile JSON and save it to `${CLAUDE_PLUGIN_DATA}/travel-profile.json`:

```json
{
  "travel": {
    "onboarding_complete": true,
    "onboarding_date": "YYYY-MM-DD",
    "home_airport": "LHR",
    "home_city": "London",
    "currency": "GBP",
    "transport_preference": "comfort",
    "children": [8, 5],
    "default": {
      "cabin_class": "economy",
      "seat_preference": "aisle",
      "hotel_min_stars": 4,
      "max_nightly_budget": 150,
      "amenities": ["wifi", "kitchen"],
      "accommodation_type_preference": "airbnb"
    },
    "contexts": {
      "solo": {
        "accommodation_type_preference": "airbnb",
        "max_nightly_budget": 100,
        "neighbourhood_vibe": "central, walkable"
      },
      "partner": {
        "accommodation_type_preference": "either",
        "max_nightly_budget": 180,
        "neighbourhood_vibe": "quiet, scenic"
      },
      "family": {
        "accommodation_type_preference": "airbnb",
        "max_nightly_budget": 200,
        "neighbourhood_vibe": "family-friendly, spacious",
        "transport_preference": "comfort",
        "notes": "Prefer trains for shorter routes — kids ride free on many operators"
      },
      "group": {
        "accommodation_type_preference": "airbnb",
        "max_nightly_budget": 200,
        "neighbourhood_vibe": "spacious, near amenities"
      },
      "work": {
        "accommodation_type_preference": "hotel",
        "max_nightly_budget": 250,
        "neighbourhood_vibe": "central, business district"
      }
    },
    "loyalty": {},
    "passport": {
      "nationality": "",
      "expiry": ""
    },
    "feedback_history": [],
    "derived_preferences": {
      "prefer": [],
      "avoid": []
    },
    "past_trips": []
  }
}
```

Only populate the fields the user actually answered. Leave others as sensible defaults.

## Confirmation

After saving, summarise what you captured back to the user in a friendly way:

"Here's your travel profile — I'll use this for every search going forward:
- Home: [airport] ([city])
- Budget: [amount]/night in [currency]
- Style: [accommodation preference]
- Transport: [preference — e.g., "Prefer comfort — I'll lead with trains for shorter routes"]
- Children: [ages or "None"]
- Must-haves: [amenities]
- Vibe: [neighbourhood preference]

You can update any of these anytime by saying '/travel-setup' again."
