---
name: erp-quickstart
description: Foundational reference for the EasyEsuite ERP MCP — how to pick the right tool, the grouped-tool action/id/query/body calling convention, the date-filter rules, pagination via _total_records, and the identifier display rules. Read this before using any other EasyEsuite ERP skill.
---

# EasyEsuite ERP — Quickstart

The `easyesuite` MCP server exposes the EasyEsuite ERP (a multi-tenant e-commerce ERP
integrating Amazon, Walmart, eBay, Temu, TikTok Shop and more). Every tool call runs as the
authenticated user and is authorized by Cerbos, so you only ever see tools you may use.

Read this skill first. The report and workflow skills assume the conventions below.

## 1. Two kinds of tools

**Grouped resource tools** (most of them) — one tool per resource, e.g. `sales_orders`,
`items`, `purchase_orders`, `sales_orders_invoices`, `shipping_shipments`,
`core_global_search`. They take an `action` plus a payload:

| Parameter | Type | Use |
|-----------|------|-----|
| `action` | enum | `list`, `retrieve`, `create`, `update`, `partial_update`, `destroy`, or a resource-specific action (e.g. `line_items`). Only the actions you're allowed to call are accepted. |
| `id` | int/str | Numeric DB id of a single record (for `retrieve`/`update`/`destroy`). |
| `query` | object | Filters + pagination for `list` (e.g. `{"status": "Open", "limit": 50, "offset": 50}`). |
| `body` | object | Field values for `create` / `update`. |
| `path_params` | object | Extra URL params when a route needs them (rare). |

**Named action tools** — purpose-built helpers and write workflows that take plain fields:
`find_item`, `find_vendor`, `find_warehouse`, `find_warehouse_item`,
`find_financial_accounts`, `get_id`, `get_item_image`, `create_sales_order`,
`fulfill_sales_order`, `create_po`, `receive_po`, `add_item`, `create_transfer`, etc.
Call these with their documented fields directly (no `action`).

This plugin does **not** currently expose a standalone `product_lookup` MCP tool. If you need
UPC/GTIN/ASIN enrichment, use another surface or extend the MCP task registry first.

## 2. Pick the right tool first

Routing cheat-sheet (mirrors the server's own instructions):

- "how many orders / orders yesterday / open orders / by marketplace" → `sales_orders` (one row = one order).
- "what items are on SO-12345 / change a line qty or price" → `sales_order_items` (one row = one product line). **Never count orders with this.**
- "invoices / invoice totals this month" → `sales_orders_invoices`.
- "purchase orders / open POs" → `purchase_orders`.
- "bills / receipts / bill payments" → `purchase_orders_bills` / `purchase_orders_receipts` / `purchase_orders_bill_payments`.
- "credit memos / customer refunds" → `sales_orders_credit_memos` / `sales_orders_customer_refunds`.
- "shipped orders / fulfillments" → `sales_orders_fulfillments`.
- "where is X / find anything by name" → `core_global_search`.

When unsure which resource holds something, run `core_global_search` with
`action="list", query={"query": "<term>"}` — it returns ranked, type-labeled hits across
modules.

## 3. Date filters (read every line — this is the #1 source of wrong answers)

- For ANY user question about dates ("orders on April 2", "invoices last month") use
  **`date_after` / `date_before`** — the marketplace placement / document date.
- **Never** default to `created_date_after` / `created_date_before`. Those reflect when the
  ERP *synced* the record and over-count due to backlog batches. Only use them if the user
  explicitly asks about ERP ingestion time.
- **Invoices have ONLY `date_after` / `date_before`.** Sending `created_date_*` to the
  invoices endpoint is silently ignored and returns ALL invoices — making your total the
  all-time count, not the period.
- Always use full **ISO 8601 UTC** datetimes: `YYYY-MM-DDTHH:MM:SSZ`. Bare dates like
  `2026-04-02` lose the timezone.
- **Single-day rule** — same-day start and end; never use the next day as the end bound:
  - ✗ `date_after='2026-04-02'  date_before='2026-04-03'`
  - ✓ `date_after='2026-04-02T00:00:00Z'  date_before='2026-04-02T23:59:59Z'`

Compute concrete dates from "today" before calling — don't pass relative words.

## 4. Pagination

`list` responses contain `_total_records` (total across ALL pages) and `results` (this page,
max ~50). **For any "how many" question, report `_total_records`, not `len(results)`.**
To page, pass `query: {"limit": 50, "offset": 50}`, then `offset: 100`, etc. Use `limit`/
`offset` — **not** `page_size`.

## 5. Showing results to the user

- Present results **inline** as text or a simple markdown table. Do **not** stage them in an
  artifact/canvas/code block.
- Show human-readable document numbers: `SO-018419`, `PO-000762`, `IN-075558`. **Never**
  show raw numeric DB ids or `_ref` in user-facing replies — those are internal.
- If a record has only `_ref` and no document number, describe it by customer, date, amount,
  and status instead. If a response includes `_display_as`, follow it exactly.

## 6. Resolving names → IDs

Most write actions need numeric IDs. Resolve first:

- `get_id(entity_type, identifier)` — exact resolver for `purchase_order`, `sales_order`,
  `invoice`, `credit_memo`, `refund`, `vendor`, `customer`, `item`, `transfer`, `adjustment`,
  `bill`, `bill_payment`, `warehouse`, `financial_account`, `payment_method`, `bank`.
- `find_item`, `find_vendor`, `find_warehouse`, `find_warehouse_item`,
  `find_financial_accounts` — fuzzy lookups that return ranked candidates.
- The `create_*` / `fulfill_*` workflow tools resolve names internally — you can pass
  `customer_name`, `warehouse_name`, etc. and let them match.

See the **building-workflows** skill for chaining patterns and how to recover when a
name resolves ambiguously.
