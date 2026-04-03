---
name: persistent-memory
description: "Persistent memory integration for travel preferences. Instructs Claude to use Cowork's built-in persistent memory to store and recall travel profiles, feedback, derived preferences, and trip history across sessions. Automatically active — not user-invocable."
user-invocable: false
---

# Persistent Memory — Travel Profile Storage

This skill governs how the AI Heroes Travel Agent uses Cowork's built-in persistent memory system to maintain travel preferences, trip history, and learned behaviours across sessions.

## Memory Keys

Use these persistent memory keys for all travel data:

| Key | Purpose | Format |
|-----|---------|--------|
| `travel_profile` | Full preference profile (home airport, defaults, contexts, loyalty, passport) | JSON object |
| `travel_feedback` | Array of post-trip feedback entries | JSON array |
| `travel_derived_preferences` | Learned prefer/avoid tags extracted from feedback and search behaviour | JSON object with `prefer` and `avoid` arrays |
| `travel_past_trips` | Trip history with outcomes, statuses, and key details | JSON array |
| `travel_watched_items` | Items being price-monitored (flights, accommodation) | JSON array |
| `travel_packing_preferences` | User's packing preferences and must-haves | JSON object |

## When to Read Memory

**Before ANY travel search or recommendation:**

1. Recall `travel_profile` from persistent memory
2. Recall `travel_derived_preferences` from persistent memory
3. If no profile exists in persistent memory, check `${CLAUDE_PLUGIN_DATA}/travel-profile.json` as fallback
4. If neither exists, ask 2-3 quick questions relevant to the current query and offer full onboarding after

**Before trip planning:**
- Also recall `travel_past_trips` to check for repeat destinations (offer "you visited [X] before — want similar or different this time?")
- Recall `travel_feedback` to apply lessons from past trips

## When to Write Memory

**After onboarding:**
- Save the complete profile to `travel_profile` in persistent memory
- Also write to `${CLAUDE_PLUGIN_DATA}/travel-profile.json` as backup

**After every search interaction:**
- If the user selects an option → add positive tags to `travel_derived_preferences.prefer`
- If the user rejects an option → analyse the reason and add tags to `travel_derived_preferences.avoid`
- Pattern detection: if the user rejects 3+ listings for the same reason (e.g., "too noisy", "too far from beach"), automatically add the corresponding tag

**After trip feedback:**
- Append to `travel_feedback`
- Update `travel_derived_preferences` based on feedback analysis
- Update the trip status in `travel_past_trips`

**After booking confirmation:**
- Update the relevant trip in `travel_past_trips` with booking details and status "booked"

## Preference Learning Rules

### Adding "prefer" tags
- User rates an aspect 4+ and mentions a specific feature → add feature as prefer tag
- User selects a listing and says why they like it → extract tags
- User returns to the same type of accommodation/flight 3+ times → add pattern as prefer tag
- Examples: `beachfront`, `quiet_area`, `kitchen`, `balcony`, `direct_flights`, `morning_departures`

### Adding "avoid" tags
- User rates an aspect 2 or below → analyse and add avoid tag
- User rejects a listing and gives a reason → extract tags
- User explicitly says "I don't like X" → add immediately
- Pattern: 3+ rejections for the same reason → auto-add
- Examples: `noisy_area`, `tourist_traps`, `shared_spaces`, `long_layovers`, `red_eye_flights`

### Tag Extraction Examples
| User says | Tag to add | Where |
|-----------|-----------|-------|
| "the neighbourhood was too touristy" | `avoid_tourist_areas` | `avoid` |
| "loved having a kitchen" | `kitchen` | `prefer` |
| "the pool was amazing" | `pool`, `resort_style` | `prefer` |
| "too far from the centre" | `central_location` | `prefer` |
| "never again with shared bathrooms" | `shared_bathrooms` | `avoid` |
| "the early morning flight was perfect" | `morning_departures` | `prefer` |

## Scoring Integration

When ranking search results, apply persistent memory preferences:
- +15 for each matching `prefer` tag
- -50 for each matching `avoid` tag
- +10 for features that appeared in 4+ rated past trips
- -30 for features that appeared in 2- rated past trips
- Score < 0 = exclude entirely

## Memory Sync

- Persistent memory is the primary store (survives across sessions)
- `${CLAUDE_PLUGIN_DATA}/travel-profile.json` is the secondary backup
- On profile changes, update both locations
- If a conflict is detected (e.g., different data in memory vs file), prefer persistent memory as the source of truth and update the file to match
