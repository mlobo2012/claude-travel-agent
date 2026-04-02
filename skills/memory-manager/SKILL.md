---
name: travel-profile
description: "View, edit, or manage your travel preference profile. Shows your current preferences, lets you update specific fields, and tracks trip history. Use when: user asks to see their profile, update preferences, check travel history, or says 'show my travel profile'."
argument-hint: "[view|update|history]"
---

# Travel Profile Manager

Manage the persistent travel preference profile stored at `${CLAUDE_PLUGIN_DATA}/travel-profile.json`.

## Commands

### View Profile (default)

If the user runs `/travel-profile` or `/travel-profile view`:

1. Read `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
2. If no file exists, tell the user: "No travel profile found. Run /travel-setup to create one."
3. If file exists, display it in a friendly format:

```
## Your Travel Profile

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

1. Read the current profile
2. Ask what they want to change
3. Update only the specified fields
4. Save back to `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
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
    "home_airport": "string (IATA code)",
    "home_city": "string",
    "currency": "string (ISO 4217)",
    "default": {
      "cabin_class": "economy|premium_economy|business|first",
      "seat_preference": "aisle|window|middle|no_preference",
      "hotel_min_stars": 3-5,
      "max_nightly_budget": number,
      "amenities": ["string"],
      "accommodation_type_preference": "airbnb|hotel|either"
    },
    "contexts": {
      "solo|partner|group|work": {
        "accommodation_type_preference": "string",
        "max_nightly_budget": number,
        "neighbourhood_vibe": "string",
        "notes": "string"
      }
    },
    "loyalty": {
      "airlines": [{ "programme": "string", "number": "string" }],
      "hotels": [{ "programme": "string", "number": "string" }]
    },
    "passport": {
      "nationality": "string",
      "expiry": "YYYY-MM"
    },
    "feedback_history": [
      {
        "trip_id": "string",
        "date": "YYYY-MM-DD",
        "rating": 1-5,
        "liked": ["string"],
        "disliked": ["string"],
        "notes": "string"
      }
    ],
    "derived_preferences": {
      "prefer": ["string"],
      "avoid": ["string"]
    },
    "past_trips": [
      {
        "destination": "string",
        "dates": { "start": "YYYY-MM-DD", "end": "YYYY-MM-DD" },
        "travellers": number,
        "context": "string",
        "status": "planned|completed|cancelled",
        "created_at": "YYYY-MM-DD"
      }
    ]
  }
}
```

## Auto-Read Before Search

**Important:** The travel-agent core skill requires reading the profile before every search. This skill provides the profile schema and location. The profile is always at `${CLAUDE_PLUGIN_DATA}/travel-profile.json`.

If the profile file doesn't exist when a search is requested, prompt the user to run `/travel-setup` first, or offer to use sensible defaults for this search only.
