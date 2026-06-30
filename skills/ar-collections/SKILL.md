---
name: ar-collections
description: Build accounts-receivable and collections reports from the EasyEsuite ERP MCP surface. Use when the user wants open invoices, partial-paid invoices, recent payments received, customer collections exposure, or AR aging from invoices and payments.
---

# AR collections

Use this skill for **receivables** work: open invoices, balances, aging, and recent customer
payments.

## Tools

- `sales_orders_invoices` (`action=list`) — invoice totals, dates, and statuses.
- `sales_orders_payments` (`action=list`) — recorded customer payments.
- `core_global_search` (`action=list`) — optional helper to resolve invoice or customer names.

## Core invoice queries

**Open invoices**
```
sales_orders_invoices(
  action="list",
  query={
    "status": "Open",
    "ordering": "-date",
    "limit": 50,
    "offset": 0
  }
)
```

**Partially paid invoices**
```
sales_orders_invoices(
  action="list",
  query={
    "status": "Partial Paid",
    "ordering": "-date",
    "limit": 50,
    "offset": 0
  }
)
```

**Payments received in a period**
```
sales_orders_payments(
  action="list",
  query={
    "date_after": "2026-06-01T00:00:00Z",
    "date_before": "2026-06-30T23:59:59Z",
    "limit": 50,
    "offset": 0
  }
)
```

## Collections workflows

### Outstanding AR

1. Pull `Open` invoices.
2. Pull `Partial Paid` invoices.
3. Sum **balances**, not totals.
4. Report total exposure and optionally the largest / oldest invoices.

### AR aging

1. Page all `Open` and `Partial Paid` invoices.
2. Compute age from invoice date or due date, depending on the user's wording.
3. Bucket into `0-30`, `31-60`, `61-90`, `90+`.
4. Sum balances in each bucket.

### Recent collections

1. Page `sales_orders_payments` for the requested period.
2. Sum payment amounts across all pages.
3. Optionally compare against invoice volume from the same period.

## Presentation

- Show `IN-...` numbers, customer, date, status, total, and balance.
- Use small inline tables for aging buckets and top exposures.
- If the user asks "what is outstanding," answer with **balance**, not invoice total.

## Gotchas

- Invoices only support `date_after` / `date_before`. Do not send `created_date_*`.
- One customer may have multiple invoices and multiple payments; do not assume 1:1 matching.
- Always page through all results before giving money totals.
