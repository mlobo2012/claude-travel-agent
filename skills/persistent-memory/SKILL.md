---
name: persistent-memory
description: "Persistent memory integration for travel preferences. Governs how the travel agent stores and recalls travel profiles, feedback, derived preferences, and trip history across sessions using DUAL persistence: file-based storage AND Claude's own memory system. Automatically active — not user-invocable."
user-invocable: false
---

# Persistent Memory — Dual Persistence for Travel Profiles

This skill governs how the AI Heroes Travel Agent persists travel data across sessions. It uses a **dual-persistence** strategy so the profile survives in both Claude Code and Cowork environments.

## Why Dual Persistence?

- **Claude Code**: `${CLAUDE_PLUGIN_DATA}` resolves to `~/.claude/plugins/data/{plugin-id}/` and persists across sessions. File-based storage is reliable here.
- **Cowork**: `${CLAUDE_PLUGIN_DATA}` resolves to a session-scoped ephemeral path that does NOT persist. Files are lost between sessions.
- **Claude's memory system**: Works in both environments. Claude can save and recall facts using its built-in memory/auto-memory feature.

By saving to BOTH, the profile is available regardless of environment.

---

## Storage Locations

### Primary: File-Based (`${CLAUDE_PLUGIN_DATA}`)

| File | Purpose |
|------|---------|
| `${CLAUDE_PLUGIN_DATA}/travel-profile.json` | Full preference profile |
| `${CLAUDE_PLUGIN_DATA}/booked-trips.json` | Confirmed bookings |
| `${CLAUDE_PLUGIN_DATA}/expense-log.json` | Scanned receipt and expense data |
| `${CLAUDE_PLUGIN_DATA}/reasoning-effectiveness.json` | Recommendation effectiveness tracking |

### Secondary: Claude's Own Memory System

After every profile save, also save key profile facts to Claude's memory so they're available even if the file can't be read. This acts as the critical fallback for Cowork sessions.

---

## HOW TO LOAD THE PROFILE (Fallback Chain)

**Before ANY travel search, recommendation, or profile operation, you MUST follow this exact sequence:**

### Step 1: Try the file
Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel-profile.json`.

- If the file exists and contains valid JSON → **use it**. This is the most complete source.
- If the file doesn't exist, is empty, or returns an error → proceed to Step 2.

### Step 2: Try Claude's memory
Check your own memory system for travel profile information. Look for memories about:
- Home airport and city
- Travel companions (partner, children, ages)
- Transport preferences
- Budget range and currency
- Loyalty programme names, numbers, and tiers
- Seat preference, cabin class
- Accommodation preferences
- Prefer/avoid tags
- Personal details (name, email)
- Passport nationality and expiry

If memory contains profile data → **reconstruct the profile from memory** and use it. Also re-save it to the file for next time:
1. Build the JSON from remembered facts
2. Use the **Write tool** to save to `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
3. Continue with the reconstructed profile

### Step 3: No profile found — trigger quick setup
If neither file nor memory has profile data:
- For a specific travel query: ask only the 2-3 questions needed for THIS search, run the search, then offer full onboarding afterwards
- For a general "plan a trip" request: run the full onboarding flow (`/travel-setup`)
- **NEVER** say "no profile found" and stop. Always either use what you have or ask what you need.

---

## HOW TO SAVE THE PROFILE (Dual Write)

**Every time the profile is created or updated, you MUST save to BOTH locations:**

### Save Step 1: Write the file
1. Build the complete profile JSON object
2. Use the **Write tool** to save to `${CLAUDE_PLUGIN_DATA}/travel-profile.json`
3. Confirm the file was written

### Save Step 2: Save to Claude's memory
Immediately after writing the file, save the key profile facts to your own memory system. Write a memory entry that includes ALL of the following (only include fields that have values):

**What to save to memory:**
- Home airport (IATA code) and home city
- Full name (as on passport/ID)
- Email address
- Phone number
- Travel companions: partner, children (with ages)
- Transport preference (speed/budget/comfort/eco)
- Default cabin class and seat preference
- Accommodation type preference
- Budget per night and currency
- Must-have amenities
- Neighbourhood vibe preferences
- Loyalty programmes: for EACH programme, save name, type, membership number, tier
- Passport nationality and expiry
- Key prefer tags (top 5)
- Key avoid tags (top 5)

**Format for memory save:**
Save as a single memory entry titled "Travel Profile" with type "user". Include all facts in a structured format so they can be easily recalled and reconstructed into JSON.

Example memory content:
```
Travel profile for AI Heroes Travel Agent:
- Name: Marco Sherwood
- Email: marco@example.com
- Home: LHR (London)
- Companions: partner + 1 child (age 3)
- Transport: prefers comfort, trains for routes under 4h
- Cabin: economy, seat: window
- Accommodation: Airbnb, budget £150/night GBP
- Amenities: wifi, kitchen
- Loyalty: BA Executive Club Gold #12345678, Marriott Bonvoy Platinum #987654321, Club Eurostar #EU789456
- Passport: British, expires 2030-06
- Prefer: direct_flights, central_location, kitchen
- Avoid: noisy_area, shared_spaces
```

### Save Step 3: Confirm to user
Tell the user: "Your travel profile has been saved and will persist across sessions."

---

## HOW TO UPDATE THE PROFILE

When any profile field changes:

1. **Read** the current profile (using the fallback chain above)
2. **Merge** the changes into the existing profile
3. **Write** the updated profile to the file
4. **Update** Claude's memory with the changed facts
5. **Confirm** the update to the user

---

## Derived Preferences

Track learned preferences as part of the travel profile under `derived_preferences`:

### Adding "prefer" tags
- User rates an aspect 4+ and mentions a specific feature → add feature as prefer tag
- User selects a listing and says why they like it → extract tags
- User returns to the same type of accommodation/flight 3+ times → add pattern
- Examples: `beachfront`, `quiet_area`, `kitchen`, `balcony`, `direct_flights`, `morning_departures`

### Adding "avoid" tags
- User rates an aspect 2 or below → analyse and add avoid tag
- User rejects a listing and gives a reason → extract tags
- User explicitly says "I don't like X" → add immediately
- Pattern: 3+ rejections for the same reason → auto-add
- Examples: `noisy_area`, `tourist_traps`, `shared_spaces`, `long_layovers`, `red_eye_flights`

After updating tags, save using the dual-write process above.

---

## Scoring Integration

When ranking search results, apply profile preferences:
- +15 for each matching `prefer` tag
- -50 for each matching `avoid` tag
- +10 for features that appeared in 4+ rated past trips
- -30 for features that appeared in 2- rated past trips
- Score < 0 = exclude entirely

---

## Booking Data

### Saving bookings
After booking confirmation:
1. Read the existing `${CLAUDE_PLUGIN_DATA}/booked-trips.json` (or start with `[]` if it doesn't exist)
2. Append the new booking
3. Use the **Write tool** to save the updated array
4. Also save key booking details to Claude's memory (destination, dates, reference number)

### Saving expenses
After scanning a receipt or document:
1. Read the existing `${CLAUDE_PLUGIN_DATA}/expense-log.json` (or start with `[]`)
2. Append the new expense entry
3. Use the **Write tool** to save
