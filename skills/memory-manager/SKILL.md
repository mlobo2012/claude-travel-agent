---
name: travel-profile
description: "View, edit, or manage your travel preference profile. Shows your current preferences, lets you update specific fields, and tracks trip history. Use when: user asks to see their profile, update preferences, check travel history, or says 'show my travel profile'."
argument-hint: "[view|update|history]"
---

# Travel Profile Manager

Manage the persistent travel preference profile using the dual-persistence approach (file + Claude memory).

## Loading the Profile

**Always use the persistent-memory fallback chain:**

1. **First:** Try reading `${CLAUDE_PLUGIN_DATA}/travel-profile.json` with the Read tool
2. **Fallback:** If the file doesn't exist or is empty, check Claude's own memory for travel profile facts
3. **If neither exists:** Tell the user "No travel profile found. Run /travel-setup to create one."

If memory has data but the file doesn't, reconstruct the JSON from memory and re-save it to the file.

## Commands

### View Profile (default)

If the user runs `/travel-profile` or `/travel-profile view`:

1. Load the profile using the fallback chain above
2. If no profile exists anywhere, suggest: "No travel profile found. Run /travel-setup to create one."
3. If profile exists, display it in a friendly format:

```
## Your Travel Profile

**Name:** [full name]
**Email:** [email]
**Home:** [airport] ([city])
**Currency:** [currency]
**Onboarded:** [date]

### Default Preferences
- Accommodation: [type preference]
- Budget: [amount]/night
- Cabin class: [class] | Seat: [preference]
- Must-haves: [amenities list]
- Min hotel rating: [stars]

### Travel Contexts
[For each context that has data:]
- **[Context]:** Budget [amount]/night, prefers [type], vibe: [neighbourhood]

### Loyalty Programmes
[List or "None set up"]

### Passport
[Nationality], expires [date] — or "Not provided"

### Learned Preferences
- Likes: [prefer tags]
- Avoids: [avoid tags]

### Trip History
[N] past trips recorded
[List recent 3 with destination, date, status]
```

### Update Profile

If the user runs `/travel-profile update` or says they want to change a preference:

1. Load the current profile using the fallback chain
2. Ask what they want to change
3. Update only the specified fields
4. **Save using dual-persistence:** write to `${CLAUDE_PLUGIN_DATA}/travel-profile.json` AND update Claude's memory
5. Confirm the change

### Trip History

If the user runs `/travel-profile history`:

1. Read the profile's `past_trips` array
2. Display trips in reverse chronological order
3. Show: destination, dates, travellers, status, key costs

## Profile Schema Reference

```json
{
  "travel": {
    "onboarding_complete": true,
    "onboarding_date": "YYYY-MM-DD",
    "personal": {
      "full_name": "string",
      "email": "string",
      "phone": "string (optional)"
    },
    "home_airport": "string (IATA code)",
    "home_city": "string",
    "currency": "string (ISO 4217)",
    "transport_preference": "speed|budget|comfort|eco|no_preference",
    "children": [3],
    "default": {
      "cabin_class": "economy|premium_economy|business|first",
      "seat_preference": "aisle|window|extra_legroom|no_preference|minimize_cost",
      "hotel_min_stars": 3-5,
      "max_nightly_budget": number,
      "amenities": ["string"],
      "accommodation_type_preference": "airbnb|hotel|either"
    },
    "contexts": {
      "solo|partner|group|work|family": {
        "accommodation_type_preference": "string",
        "max_nightly_budget": number,
        "neighbourhood_vibe": "string",
        "notes": "string"
      }
    },
    "loyalty": {
      "programmes": [
        {
          "name": "string",
          "type": "airline|hotel|rail",
          "alliance": "string",
          "membership_number": "string",
          "tier": "string",
          "points_balance": number,
          "points_currency": "string",
          "tier_expiry": "string",
          "auto_apply": true
        }
      ]
    },
    "passport": {
      "nationality": "string",
      "expiry": "YYYY-MM",
      "number": "string (optional)"
    },
    "feedback_history": [],
    "derived_preferences": {
      "prefer": ["string"],
      "avoid": ["string"]
    },
    "past_trips": []
  }
}
```

## Auto-Read Before Search

**Important:** The travel-agent core skill requires reading the profile before every search. Always use the persistent-memory fallback chain (file -> memory -> quick setup). Never read only one source.
