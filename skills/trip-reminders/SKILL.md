---
name: trip-reminders
description: "Pre-trip reminders and Dispatch-ready travel nudges. Supports Cowork scheduled-task workflows for T-14, T-7, T-3, and T-1 reminders when scheduling is available. Use when: a trip is booked or the user says 'remind me before my trip'."
user-invocable: false
---

# Pre-Trip Reminders

Prepare reminder content for booked trips and use Dispatch when the surrounding Claude product supports scheduled execution.

## Current Reality

- Cowork supports scheduled tasks and Dispatch.
- Anthropic's help docs say those tasks are set up through `/schedule` or the Scheduled sidebar.
- This plugin does not install a separate always-on cron service.
- In Claude Code, do not promise that pre-trip reminders will fire automatically unless the product explicitly confirms that scheduling exists in the current workflow.

## When To Prepare Reminders

- When a trip is confirmed as booked
- When the user says "remind me before my trip" or "set up trip reminders"
- When a booking is detected via Gmail

## Reminder Schedule

Use this as the target reminder plan when scheduling is available:

- T-14 for longer trips
- T-7 for standard trips
- T-3 for short trips or specific ticket-printing prompts
- T-1 for departure-eve reminders

## Reminder Content Requirements

All reminders should include, where relevant:

1. Current weather for the destination
2. Flight or train details if booked
3. Accommodation details if booked
4. Packing guidance based on trip type and weather
5. Destination-specific notes such as adapter, currency, or visa reminders

## If Scheduling Is Available

Use Cowork scheduled tasks or the product's scheduling surface, then confirm the reminder plan and cadence to the user.

## If Scheduling Is Not Available

Be honest. Save the trip context, explain that the reminder content is prepared, and tell the user that recurring delivery is not guaranteed from the plugin alone.
