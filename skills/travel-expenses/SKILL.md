---
name: travel-expenses
description: Generate a structured expense report for a trip from logged receipts, bills, and scanned travel documents.
user-invocable: true
argument-hint: "[trip name or date range]"
---

# Travel Expenses

Generate an expense report from `${CLAUDE_PLUGIN_DATA}/expense-log.json`.

## What to include

- group expenses by category
- show totals per category
- show grand total
- show currencies captured
- highlight unusual or duplicate charges
- if a trip budget exists, compare actual spend vs budget

If no expenses are logged yet, tell the user clearly and suggest scanning receipts or hotel bills first.
