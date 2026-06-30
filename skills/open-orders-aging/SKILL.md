---
name: open-orders-aging
description: Build an open-orders aging report from the EasyEsuite ERP MCP surface. Use when the user wants open or pending sales orders bucketed by age, wants to see how old unshipped or uninvoiced orders are, or wants an operational backlog view by status, customer, or marketplace.
---

# Open orders aging

Use this skill when the user is asking for an **aging/backlog** view of sales orders rather
than just a raw count.

## Tools

- `sales_orders` (`action=list`) — the source for the order rows.
- `core_global_search` (`action=list`) — optional helper when you need to resolve a customer,
  marketplace, or document number first.

## Statuses to include

Typical "still open" buckets come from:

- `Open`
- `Pending Fulfillment`
- `Partial Fulfilled`
- `Partial Fulfilled/Pending Invoice`
- `Fulfilled/Pending Invoice`

Exclude `Invoiced` and `Voided` unless the user explicitly asks for them.

## Core recipe

```
sales_orders(
  action="list",
  query={
    "status": "Pending Fulfillment",
    "date_after": "2026-06-01T00:00:00Z",
    "date_before": "2026-06-30T23:59:59Z",
    "limit": 50,
    "offset": 0
  }
)
```

Repeat with `offset += 50` until all rows are collected. Use `_total_records` for counts.

## Aging method

There is no server-side aging bucket. Build it client-side:

1. Pull all matching rows from `sales_orders`.
2. Use the order/document date from the row.
3. Compute age in days from today.
4. Bucket into `0-3`, `4-7`, `8-14`, `15-30`, `31+` days, or whatever buckets the user asks for.
5. Group again by status / marketplace / customer when useful.

## Useful variants

- **Pending fulfillment backlog** — filter `status="Pending Fulfillment"`.
- **Pending invoice backlog** — filter `status="Fulfilled/Pending Invoice"` or
  `status="Partial Fulfilled/Pending Invoice"`.
- **All active backlog** — page all active statuses and combine them client-side.
- **Customer-specific backlog** — use `core_global_search(action="list", query={"query": "Customer Name"})`
  if needed, then filter with the appropriate customer identifier present in order rows.

## Presentation

- Present a compact inline table like `Bucket | Orders | Example statuses`.
- If the user asks for detail, show the oldest few `SO-...` numbers with customer, date, and status.
- Never report only the first page as the total.

## Gotchas

- This is an **order** report, not a line-item report. Do not use `sales_order_items`.
- Use `date_after` / `date_before`, not `created_date_*`.
- One order can be partially fulfilled and still open from an operations standpoint; keep status
  labels visible in the output.
