---
description: Generate an expense report for a trip from scanned receipts and bills
---

# Travel Expense Report

Generate a formatted expense report for a specific trip using data from the document-scanner skill.

## Usage

- `/travel-expenses` — Show expenses for the most recent trip
- `/travel-expenses [trip name or destination]` — Show expenses for a specific trip
- `/travel-expenses --daily` — Include per-day breakdown
- `/travel-expenses --vat` — Include VAT/tax summary

## What It Does

1. Reads `${CLAUDE_PLUGIN_DATA}/expense-log.json` for all expenses associated with the specified trip
2. Groups expenses by category (transport, accommodation, food, activities, shopping, tips, fees, other)
3. Generates a formatted markdown expense report with:
   - Trip summary (destination, dates, travellers)
   - Total spend vs budget (if budget was set)
   - Category breakdown with totals and percentages
   - Individual expense line items
   - VAT/tax summary (if `--vat` flag used)
   - Per-day breakdown (if `--daily` flag used)
   - Comparison to your average spend on similar trips
4. Offers to save the report as a file

## If No Expenses Found

If no expenses are logged for the trip, suggest:
- "No expenses logged yet for this trip. Upload receipts, boarding passes, or hotel bills and I'll track them automatically. Just send a photo in the chat."

## Powered By

This command uses the **document-scanner** skill. Scan receipts and bills during your trip, and this command compiles them into a clean report afterwards.
