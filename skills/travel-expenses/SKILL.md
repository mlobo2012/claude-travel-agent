---
name: travel-expenses
description: Generate a structured expense report for a trip from logged receipts, bills, and scanned travel documents.
user-invocable: true
argument-hint: "[trip name or date range]"
---

# Travel Expenses

Use the **Read tool** to load `${CLAUDE_PLUGIN_DATA}/expense-log.json`, then generate an expense report from its contents.

## What to include

- group expenses by category
- show totals per category
- show grand total
- show currencies captured
- highlight unusual or duplicate charges
- if a trip budget exists, compare actual spend vs budget

If no expenses are logged yet, tell the user clearly and suggest scanning receipts or hotel bills first.
