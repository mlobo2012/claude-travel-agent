---
name: trip-feedback
description: "Record feedback after a trip to improve future recommendations. Updates the travel profile with learned preferences. Use when: user says how a trip went, gives feedback on a recommendation, says 'trip feedback', or returns from a trip."
argument-hint: "[destination or trip reference]"
---

# Trip Feedback

Capture post-trip feedback to improve future recommendations.

## When to Trigger

- User mentions they returned from a trip
- User says something like "the trip was great/terrible/ok"
- User explicitly runs `/trip-feedback`
- User comments on a past recommendation

## Feedback Collection

Ask these questions conversationally (adapt based on what the user volunteers):

1. **Overall rating:** "How was the trip overall, 1-5?"
2. **Accommodation:** "How was the accommodation? Anything you particularly liked or would avoid next time?"
3. **Flights:** "How were the flights? Any issues?"
4. **Activities:** "What was the highlight? Anything you'd skip?"
5. **Neighbourhood/Location:** "How was the location? Would you stay in that area again?"
6. **Budget:** "Did the trip feel good value? Over or under budget?"

## Processing Feedback

After collecting feedback:

### 1. Save to Feedback History

Read `${CLAUDE_PLUGIN_DATA}/travel-profile.json` and append to `feedback_history`:

```json
{
  "trip_id": "[destination]-[date]",
  "date": "YYYY-MM-DD",
  "destination": "string",
  "rating": 1-5,
  "liked": ["specific things they liked"],
  "disliked": ["specific things they didn't like"],
  "accommodation_rating": 1-5,
  "flight_rating": 1-5,
  "would_return": true/false,
  "notes": "free-text summary"
}
```

### 2. Update Derived Preferences

Analyse the feedback and update `derived_preferences`:

**Add to "prefer" if:**
- User rated something 4+ and mentioned specific features
- User said they'd return / would stay there again
- User highlighted a specific amenity, location type, or experience

**Add to "avoid" if:**
- User rated something 2 or below
- User explicitly said they'd avoid something
- User described a negative experience tied to a specific feature

### 3. Update Past Trips

Find the matching trip in `past_trips` and update its status to "completed".

### 4. Update Persistent Memory

After processing feedback, update persistent memory:

1. **Append to `travel_feedback`** in persistent memory — the full structured feedback entry
2. **Update `travel_derived_preferences`** in persistent memory:
   - Extract positive tags from liked items → add to `prefer` array
   - Extract negative tags from disliked items → add to `avoid` array
   - Examples:
     - "the neighbourhood was too touristy" → add `avoid_tourist_areas` to `avoid`
     - "loved the kitchen" → add `kitchen` to `prefer`
     - "the pool was incredible" → add `pool` to `prefer`
     - "too noisy at night" → add `quiet_area` to `prefer`, `noisy_area` to `avoid`
3. **Update `travel_past_trips`** in persistent memory — set trip status to "completed"
4. **Update scoring weights** based on feedback history:
   - Features from trips rated 4-5 get +5 weight bonus in future scoring
   - Features from trips rated 1-2 get -10 weight penalty in future scoring

### 5. Adjust Defaults (if pattern emerges)

If 3+ feedback entries show a consistent pattern (e.g., always rates Airbnbs higher than hotels, always complains about no kitchen), suggest updating the default preferences:

"I've noticed you consistently [pattern]. Want me to update your default preferences to reflect this?"

Only update defaults with explicit user approval.

### 6. Proactive Feedback Request

After a trip's return date has passed (detected via trip dates in persistent memory), proactively ask for feedback in the next conversation:

"Welcome back from [destination]! How was the trip? I'd love to hear your feedback so I can make future recommendations even better. Want to do a quick review?"

Only ask once per trip. Track whether feedback has been requested in the trip's memory entry.

## Feedback Confirmation

After saving:

```
Thanks for the feedback! Here's what I've learned:

**Liked:** [list]
**Will avoid:** [list]
**Updated preferences:** [what changed, if anything]

This will improve your future recommendations. You now have [N] trips in your history.
```
