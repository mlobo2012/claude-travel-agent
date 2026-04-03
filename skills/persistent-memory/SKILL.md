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

### Secondary: Claude's Auto-Memory System

After every profile save, also save key profile facts to Claude's auto-memory system. This is the critical fallback that enables cross-session persistence — especially in Cowork where the file path is ephemeral.

**How Claude's auto-memory works:**
- In **Claude Code**: Claude has a project-level memory directory (typically `~/.claude/projects/<project-hash>/memory/`). To save a memory, use the **Write tool** to create a markdown file in that directory with frontmatter (name, description, type), then update `MEMORY.md` in that directory.
- In **Cowork**: Claude has a built-in memory system. To save a memory, explicitly state: "I need to save this to my memory for future sessions" and write the structured profile data as a memory entry.
- The key is to be **explicit**: do not just "remember" the data — actively write it to the memory system using the appropriate mechanism for the current environment.

**Memory file format (for Claude Code):**
```markdown
---
name: travel-profile
description: AI Heroes Travel Agent — complete travel preference profile for the user
type: user
---

Travel profile for AI Heroes Travel Agent:
- Name: [full name]
- Email: [email]
- Home: [IATA code] ([city])
...all profile fields...
```

---

## HOW TO LOAD THE PROFILE (Fallback Chain)

**Before ANY travel search, recommendation, or profile operation, you MUST follow this exact sequence:**

### Step 1: Try the file
Use the **Read tool** on `${CLAUDE_PLUGIN_DATA}/travel-profile.json`.

- If the file exists and contains valid JSON → **use it**. This is the most complete source.
- If the file doesn't exist, is empty, or returns an error → proceed to Step 2.

### Step 2: Try Claude's auto-memory
Search your auto-memory system for a memory file named "travel-profile" or containing "AI Heroes Travel Agent" or "travel profile". In Claude Code, look for a file like `travel_profile.md` or `travel-profile.md` in your project memory directory. Read the `MEMORY.md` index file first to find it.

Look for any of these data points:
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

### Step 3: Scan existing memories for travel-relevant information
Before giving up, scan Claude's memory for ANY travel-relevant facts the user may have mentioned in previous non-travel conversations. Look for:
- Cities mentioned as "home" or "I live in..."
- Family composition (partner, children, ages)
- Airline or hotel loyalty mentions
- Travel preferences mentioned in passing

If you find scattered travel facts → combine them into a partial profile and use what you have.

### Step 4: No profile found — use conversation context + offer setup
If no profile data exists anywhere:
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

### Save Step 2: Save to Claude's auto-memory system
Immediately after writing the file, you MUST save the profile to Claude's persistent memory. This is the step that enables cross-session persistence.

**In Claude Code:** Use the **Write tool** to create (or overwrite) a memory file in your project memory directory. The file should be named `travel_profile.md` and formatted as:

```markdown
---
name: travel-profile
description: AI Heroes Travel Agent — complete travel preference profile including home airport, companions, loyalty programmes, and accommodation preferences
type: user
---

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

After writing the file, update `MEMORY.md` in the same memory directory to include a pointer:
`- [Travel Profile](travel_profile.md) — AI Heroes Travel Agent user preferences, loyalty programmes, home airport`

**In Cowork / other environments:** Explicitly save the same structured data to Claude's built-in memory system as a "user" type memory entry.

**What to save to memory (include ALL fields that have values):**
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

## Progressive Profile Capture

**The travel profile should build organically from conversation — explicit onboarding is optional.**

### How it works
Any time the user discloses travel-relevant information during ANY conversation (not just during `/travel-setup`), automatically capture it into the profile. This should happen **silently** — do NOT announce "I've updated your profile" every time. Only confirm if the user explicitly asks about their profile or if a major detail changes.

### What to capture automatically
Listen for and extract these from natural conversation:

| Signal in conversation | Profile field to update |
|------------------------|------------------------|
| "I'm flying from London" / "I live in west London" | `home_airport`, `home_city` |
| "my partner and I" / "travelling with my wife" | `contexts` → partner |
| "my 2.5-year-old" / "our toddler" / "kids are 8 and 5" | `children` array |
| "I have BA Gold" / "my Executive Club number is..." | `loyalty.programmes[]` |
| "I prefer trains" / "hate flying short distances" | `transport_preference` |
| "we always stay in Airbnbs" | `accommodation_type_preference` |
| "budget is around £150/night" | `max_nightly_budget`, `currency` |
| "I need a kitchen" / "must have wifi" | `amenities` |
| "I love beachfront places" / "hate touristy areas" | `derived_preferences.prefer/avoid` |
| "window seat always" / "aisle please" | `seat_preference` |
| "we're celebrating our anniversary" | travel context → partner |
| "British passport" / "EU passport" | `passport.nationality` |

### Capture process
1. Extract the travel-relevant fact from the user's message
2. Load the existing profile (if any) using the fallback chain
3. Merge the new fact into the profile
4. Save using dual-write (file + memory) — **silently**
5. Continue with the conversation naturally

### First interaction scan
On the very first travel-related interaction, before asking ANY questions:
1. Check Claude's auto-memory for existing travel-related memories from other conversations
2. Check the plugin data file
3. If you find scattered facts (e.g., the user once mentioned living in London in a non-travel context), combine them into a partial profile
4. Use whatever you have — a partial profile is better than no profile

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
