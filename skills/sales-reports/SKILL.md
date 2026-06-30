---
name: sales-reports
description: Build sales and revenue reports from the EasyEsuite ERP — order counts and revenue by period, marketplace or status, invoiced vs. fulfilled, top products, and period-over-period comparisons. Use for questions like "how many orders last week", "revenue by marketplace this month", or "what sold best in April".
---

# Sales reports

Read-only reporting over sales orders, invoices, fulfillments, and line items. Follow the
date-filter and pagination rules in **erp-quickstart** — getting those wrong is the main way
sales numbers come out wrong.

## Which tool answers which question

| Question | Tool | Notes |
|----------|------|-------|
| How many orders? Orders by status/marketplace/date | `sales_orders` (`list`) | One row = one order. Report `_total_records`. |
| What's on a specific order? Line-level detail | `sales_order_items` (`list`) | One row = one product line. Never use to count orders. |
| Billed revenue, invoice totals | `sales_orders_invoices` (`list`) | Invoices use ONLY `date_after`/`date_before`. |
| Shipped/delivered volume | `sales_orders_fulfillments` (`list`) | Filter by `date_after`/`date_before` or `sales_order`. |
| Credit memos / refunds (revenue offsets) | `sales_orders_credit_memos`, `sales_orders_customer_refunds` | Subtract from gross. |

## Core recipes

**Order count for a period**
```
sales_orders  action=list  query={"date_after":"2026-04-01T00:00:00Z","date_before":"2026-04-30T23:59:59Z"}
```
Answer with `_total_records`.

**Orders by marketplace** — get marketplace IDs first (the order rows include marketplace
info, or use `core_global_search` / the marketplaces list). Then either loop the filter
`query={"marketplace": <id>, "date_after":..., "date_before":...}` per marketplace, or page
through all orders for the period and group by marketplace yourself.

**Open / unshipped orders**
```
sales_orders  action=list  query={"status":"Open"}
```
Valid statuses: `Open`, `Pending Fulfillment`, `Partial Fulfilled`,
`Partial Fulfilled/Pending Invoice`, `Fulfilled/Pending Invoice`, `Invoiced`, `Voided`.

**Invoiced revenue this month**
```
sales_orders_invoices  action=list  query={"date_after":"2026-06-01T00:00:00Z","date_before":"2026-06-30T23:59:59Z","status":"Paid"}
```
Page through all results (`offset` += 50) and sum the amount field. Use `ordering=-date` or
`ordering=-total_amount_with_original_currency` to surface the largest first.

**Top-selling products** — page `sales_order_items` (or invoice line items) for the period
and aggregate quantity / line total by item. There is no server-side "top N" — collect the
pages, then rank client-side and present the top rows.

## Period-over-period comparison

Run the same `list` twice with the two date windows, capture `_total_records` (and summed
amounts), then present a small table: This period | Prior period | Δ | Δ%. Keep the two
windows the same length and aligned (e.g. full calendar months).

## Presentation

- Always report `_total_records` for counts; sum across **all** pages for money.
- Use document numbers (`SO-…`, `IN-…`), never raw ids.
- Net revenue = invoiced − credit memos − refunds when the user asks for "net".
- Render as a compact inline markdown table; never an artifact.

## Gotchas

- Don't mix `created_date_*` into sales/invoice date questions (see quickstart) — it
  over-counts and, for invoices, is silently ignored.
- "Orders" ≠ "line items" ≠ "invoices" ≠ "fulfillments". Confirm which the user means; a
  multi-line order is one order but many line items.
- A single order can have multiple invoices and multiple fulfillments — don't assume 1:1.
