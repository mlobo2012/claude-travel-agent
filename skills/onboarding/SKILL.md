---
name: travel-setup
description: "First-time travel profile onboarding. Run this when the user has never set up their travel preferences, or wants to redo their profile. Asks conversational questions to build a complete travel preference profile including transport mode preferences. Use when: user says 'set up travel', 'travel preferences', 'new traveller setup', or this is their first travel query and no profile exists."
argument-hint: ""
---

# Travel Profile Setup

You are running first-time onboarding for the AI Heroes Travel Agent. Your goal is to have a natural, conversational exchange that builds a complete travel preference profile.

## IMPORTANT: Explicit Onboarding is Optional

The travel profile builds **organically from conversation** via progressive capture (see persistent-memory skill). The user does NOT need to run `/travel-setup` to have a profile. Every time they mention travel-relevant details (home city, companions, loyalty numbers, preferences), those are captured silently into the profile.

This explicit onboarding flow should ONLY run when:
- The user explicitly asks (`/travel-setup`, "set up my travel preferences")
- The user wants to do a comprehensive profile review/rebuild

## Before Starting

Check if a profile already exists using the persistent-memory fallback chain:
1. Try reading `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
2. If not found, check Claude's auto-memory for travel profile facts (look for `travel_profile.md` or `travel-profile` in MEMORY.md)
3. If not found, scan Claude's existing memories for any travel-relevant facts from other conversations

If a profile exists, show the user what you already have and ask if they want to update it, fill in gaps, or start fresh.

## Quick Mode — When User Asks a Specific Travel Question Without a Profile

If the user is asking a specific travel question (e.g., "find me flights to Sicily") and no profile exists, **DO NOT block them with full onboarding**. Instead:
1. Ask only the 2-3 questions needed for THIS specific search (e.g., dates, budget, party size, departure airport)
2. Run the search immediately with those answers
3. **Silently save** whatever they told you to the profile (progressive capture)
4. After presenting results, offer full onboarding: "Want me to save more detailed preferences for future searches? Run `/travel-setup` to set up your full travel profile."

Only run the full onboarding below when the user explicitly asks for it.

## Full Onboarding Flow

Ask these questions **one or two at a time** in a natural conversational flow. Don't dump all questions at once. Adapt based on answers — if someone says they only travel solo, skip group-specific questions. If they don't have children, skip child-specific questions.

**If the user provides many details at once** (e.g., "Home airport is LHR, I travel with my partner and toddler, budget £150/night, prefer trains under 4h, BA Gold #12345678"), extract ALL details from their message and only ask follow-up questions for missing fields.

### The Questions

**Q1: Personal Details**
"Let's start with a couple of basics — what's your full name (as it appears on your passport or ID)? And an email address I can associate with your profile?"
-> Stores: `personal.full_name`, `personal.email`

**Q1b: Phone (optional)**
"And a phone number? (Optional — useful for booking references)"
-> Stores: `personal.phone`

**Q2: Home Base**
"Where do you usually fly from? (airport code or city)"
-> Stores: `home_airport`, `home_city`

**Q3: Travel Style**
"How do you usually travel — solo, with a partner, in a group, or for work? (You can pick more than one)"
-> Stores: `contexts` keys

**Q4: Transport Preferences**
"When there's a good train or ferry option for shorter routes, do you generally prefer those over flying? What matters most to you: speed, budget, comfort, or environmental impact?"
-> Stores: `transport_preference` (one of: speed, budget, comfort, eco, no_preference)

**Q5: Family Travel**
"Do you travel with children? If so, what are their ages? (This helps me find free/discounted fares on trains and ferries)"
-> Stores: `children` (array of ages), updates context-specific profiles
-> If no children: skip and move on

**Q6: Accommodation Philosophy**
"When it comes to where you stay — are you more of a hotel person, Airbnb person, or does it depend on the trip?"
-> Stores: `accommodation_type_preference`

**Q7: Budget**
"What's your comfortable per-night budget for accommodation? And what currency do you prefer?"
-> Stores: `max_nightly_budget`, `currency`

**Q8: Must-Have Amenities**
"Any non-negotiables for where you stay? (wifi, kitchen, pool, parking, washing machine, etc.)"
-> Stores: `amenities`

**Q9: Flight Preferences**
"Any flight preferences? Favourite airlines, cabin class?"
-> Stores: `cabin_class`, preferred airlines

**Q9b: Seat Preference**
"Seat preference — window, aisle, extra legroom, or just minimize cost?"
-> Stores: `seat_preference` (one of: window, aisle, extra_legroom, no_preference, minimize_cost)

**Q10: Loyalty Programmes**
"Are you in any airline, hotel, or rail loyalty programmes? If so, tell me the programme name, your membership number, and your current tier/status. I'll auto-apply these to every search and track your points and tier progress."
-> Stores: `loyalty.programmes[]` — for each programme, capture:
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

**Q11: Neighbourhood & Vibe**
"When you're in a new city, what vibe are you after? (central & walkable, quiet residential, beachfront, near nightlife, etc.)"
-> Stores: context-specific `neighbourhood_vibe`

**Q12: Past Trip Reference**
"Tell me about a trip you loved — and one that didn't quite work out. What made the difference?"
-> Stores: `derived_preferences`, initial `past_trips`

**Q13: Passport**
"What passport do you hold? And roughly when does it expire? (This helps me flag visa requirements)"
-> Stores: `passport.nationality`, `passport.expiry`

**Q13b: Passport Number (optional)**
"Want to store your passport number too? (Optional — handy for international bookings, stored securely in your local profile)"
-> Stores: `passport.number`

## After All Questions

Build the complete profile JSON and save it using the **dual-persistence** approach:

### Step 1: Write the file

Save to `${CLAUDE_PLUGIN_DATA}/travel-profile.json`:

```json
{
  "travel": {
    "onboarding_complete": true,
    "onboarding_date": "YYYY-MM-DD",
    "personal": {
      "full_name": "Marco Sherwood",
      "email": "marco@example.com",
      "phone": "+44..."
    },
    "home_airport": "LHR",
    "home_city": "London",
    "currency": "GBP",
    "transport_preference": "comfort",
    "children": [3],
    "default": {
      "cabin_class": "economy",
      "seat_preference": "window",
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
    "loyalty": {
      "programmes": []
    },
    "passport": {
      "nationality": "",
      "expiry": "",
      "number": ""
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

### Step 2: Save to Claude's auto-memory system

Immediately after writing the file, you MUST also save the profile to Claude's persistent auto-memory. This is CRITICAL for cross-session persistence — especially in Cowork where the plugin data file may not survive.

**In Claude Code:** Use the **Write tool** to create/overwrite a file named `travel_profile.md` in your project memory directory (the same directory where `MEMORY.md` lives). Use this format:

```markdown
---
name: travel-profile
description: AI Heroes Travel Agent — complete travel preference profile including home airport, companions, loyalty programmes, and accommodation preferences
type: user
---

Travel profile for AI Heroes Travel Agent:
- Name: [full name]
- Email: [email]
- Home: [IATA] ([city])
- Companions: [details]
- Children: [ages]
- Transport: [preference]
- Cabin: [class], seat: [preference]
- Accommodation: [type], budget £[amount]/night [currency]
- Amenities: [list]
- Loyalty: [programme] [tier] #[number], ...
- Passport: [nationality], expires [date]
- Prefer: [tags]
- Avoid: [tags]
```

Then update `MEMORY.md` to include: `- [Travel Profile](travel_profile.md) — AI Heroes Travel Agent user preferences, loyalty, home airport`

**In Cowork:** Save the same structured data as a memory entry titled "Travel Profile" with type "user".

Include ALL collected profile data:
- Full name, email, phone
- Home airport and city
- Travel companions and children's ages
- Transport preference
- Cabin class and seat preference
- Budget and currency
- Accommodation preferences and amenities
- Loyalty programme names, numbers, and tiers
- Passport nationality, expiry, number
- Neighbourhood vibe preferences
- Any initial prefer/avoid tags from the past trip discussion

### Step 3: Confirm to user

After saving, summarise what you captured back to the user in a friendly way:

"Here's your travel profile — I'll use this for every search going forward:
- **Name:** [full name]
- **Home:** [airport] ([city])
- **Budget:** [amount]/night in [currency]
- **Style:** [accommodation preference]
- **Transport:** [preference — e.g., "Prefer comfort — I'll lead with trains for shorter routes"]
- **Seat:** [preference]
- **Children:** [ages or "None"]
- **Must-haves:** [amenities]
- **Vibe:** [neighbourhood preference]
- **Loyalty:** [programme names and tiers]
- **Passport:** [nationality], expires [date]

Your profile has been saved and will persist across sessions. You can update any of these anytime by saying '/travel-setup' again, or manage specific fields with '/travel-profile update'."
