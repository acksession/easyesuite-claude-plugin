---
name: building-workflows
description: How to chain EasyEsuite ERP MCP tools into reliable multi-step workflows — resolving names to IDs, handling ambiguous/auto-selected matches, paginating through large result sets, validating before writing, and recovering from errors. Use when a task needs more than one tool call or any create/update/delete on the ERP.
---

# Building EasyEsuite ERP workflows

A workflow is any task that needs more than one tool call — "create an order and fulfill
it", "find low-stock items and raise a PO", "reconcile this month's invoices to payments".
This skill is the playbook for doing that safely. Pair it with **erp-quickstart** for the
calling conventions.

## The general loop

1. **Resolve** every human name/number to a numeric ID (`get_id`, `find_*`,
   `core_global_search`) — or use a workflow tool that resolves names for you.
2. **Read before you write.** Retrieve the current record so you know its real state and the
   exact field names/line-item IDs you'll patch.
3. **Confirm destructive or outward-facing steps** with the user before executing —
   creating invoices, applying payments, shipping, voiding, deleting. Summarize what you're
   about to do and the key numbers.
4. **Write** with the smallest payload that does the job (`update`/`partial_update` preserve
   unmentioned fields — only send what changes).
5. **Verify** by reading back or checking the response status, then report the result inline
   with the human-readable document number and a link if one is returned.

## Resolving names → IDs

```
get_id(entity_type="vendor", identifier="Acme Supplies")      # exact resolver
find_item(query="wireless mouse")                              # fuzzy, ranked candidates
core_global_search(action="list", query={"query": "SO-018419"})  # don't know the module
```

- Prefer `get_id` when you have an exact name or document number.
- Use `find_*` when the user was vague; show the top candidates and ask if it's ambiguous.
- The `create_sales_order` / `create_po` / `fulfill_sales_order` tools accept names
  (`customer_name`, `warehouse_name`, `vendor_name`, …) and resolve them internally.

## Handling auto-selected / ambiguous matches

The workflow tools auto-select the first match but return the full candidate list (e.g.
`warehouse_options`, `customer_options`) plus an `auto_selected` flag. Always sanity-check:

- Is the auto-selected name a good match for what the user said? If yes, continue.
- If a better candidate exists (typo, partial name), **re-call the tool with the explicit
  numeric ID** instead of the name (e.g. `warehouse_id=3` rather than `warehouse_name`).
- If several are equally plausible, ask the user which one before proceeding.

## Paginating to completeness

For totals/aggregations, never trust a single page:

1. Call `list` with `query: {"limit": 50}`.
2. Read `_total_records`. If it exceeds the page size, loop with `offset` += 50 until you've
   collected all rows (or enough to answer).
3. Report `_total_records` for "how many"; aggregate over all pages for sums/averages.

For a quick count you usually only need page 1 — `_total_records` is the answer.

## Validate before writing

- Quantities and prices must be positive; amounts that are discounts are negative
  (`financial_items: [{"expense_name": "Discount", "amount": -10.00}]`).
- For line-item updates, the line's own id is required — get it from the detail/`retrieve`
  response (e.g. `po_item_id` from `get_po_details`, `sales_order_id` from an invoice row).
- Dates in bodies are usually `YYYY-MM-DD`; date *filters* are ISO 8601 UTC (see quickstart).
- Check inventory availability (`items_available_quantity_in_warehouses` /
  `find_warehouse_item`) before committing a sale or transfer you expect to ship.

## Error recovery

- A failed call returns `{"success": false, "status_code": ..., "error": "..."}`. Read the
  error — common causes are unresolved names, missing required body fields, or insufficient
  permission (Cerbos deny). Fix the input and retry; don't blindly repeat the same call.
- "Not found" on a `retrieve` usually means you passed a document-number string where a
  numeric id was expected — `list` with `search` first to get the numeric id.
- A Cerbos denial means the user's role can't perform that action — tell them, don't try to
  work around it.

## Composing the common chains

- **Quote → order → ship**: `find_item` (availability + price) → `create_sales_order` →
  `fulfill_sales_order` → `shipping_shipments_rerate` / `shipping_shipments_validate_for_purchase`
  / `shipping_shipments_buy`. See **create-sales-order** and
  **fulfill-and-ship-orders**.
- **Restock**: `inventory-reports` (find low stock) → `create_po` → `receive_po` →
  `purchase_orders_bills` → `create_bill_payment`. See **purchase-order-lifecycle**.
- **Bill → pay**: `sales_orders_invoices` create → `sales_orders_payments` /
  `pay_invoice`. See **invoicing-and-payments**.

Keep each step's output around — later steps almost always need an ID returned earlier.
