---
name: ap-cash-outlook
description: Build accounts-payable and supplier cash-outlook reports from the EasyEsuite ERP MCP surface. Use when the user wants open bills, partially paid bills, recent vendor payments, AP exposure by vendor, or a near-term cash-out view from bills and bill payments.
---

# AP cash outlook

Use this skill for **payables** reporting: what the company owes, what has already been paid,
and which vendors are driving near-term cash requirements.

## Tools

- `purchase_orders_bills` (`action=list`) — supplier bills and balances.
- `purchase_orders_bill_payments` (`action=list`) — actual cash payments to vendors.
- `find_vendor` — optional fuzzy vendor resolver.
- `accounts_vendors` (`action=list`) — optional direct vendor lookup.

## Core bill queries

**Open bills**
```
purchase_orders_bills(
  action="list",
  query={
    "status": "Open",
    "limit": 50,
    "offset": 0
  }
)
```

**Partially paid bills**
```
purchase_orders_bills(
  action="list",
  query={
    "status": "Partially Paid",
    "limit": 50,
    "offset": 0
  }
)
```

**Recent cash paid**
```
purchase_orders_bill_payments(
  action="list",
  query={
    "date_after": "2026-06-01T00:00:00Z",
    "date_before": "2026-06-30T23:59:59Z",
    "limit": 50,
    "offset": 0
  }
)
```

## Common reports

### AP exposure

1. Page `Open` bills.
2. Page `Partially Paid` bills.
3. Sum remaining balances across both sets.
4. Group by vendor if the user wants the largest obligations.

### Vendor-specific exposure

1. Resolve the vendor with `find_vendor(query="...")` when needed.
2. Filter `purchase_orders_bills(action="list", query={"vendor_account": <id>, ...})`.
3. Sum balances and show open documents.

### Near-term cash-out

1. Pull open and partially paid bills.
2. Bucket by due date, for example `Due now`, `Next 7 days`, `Next 30 days`.
3. Sum balances per bucket.
4. Optionally compare against recent `purchase_orders_bill_payments`.

## Presentation

- Show bill numbers, vendor, date/due date, status, total, and balance.
- Use inline tables for due buckets and top-vendor exposure.
- Distinguish **billed**, **outstanding**, and **cash paid** clearly.

## Gotchas

- Do not mix PO totals with bill balances; they answer different questions.
- Always aggregate across all pages before reporting totals.
- If the user says "cash out this month," that usually means bill payments, not open bills.
