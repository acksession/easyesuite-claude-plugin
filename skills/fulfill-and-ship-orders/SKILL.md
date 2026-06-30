---
name: fulfill-and-ship-orders
description: Fulfill sales orders in the EasyEsuite ERP and ship them — create fulfillment records, create shipment records, rerate and validate shipments, buy labels, and work with shipment batches. Use when the user wants to ship an order, mark items fulfilled, generate a label, or manage shipment batches.
---

# Fulfill and ship orders

Two layers: **fulfillment** (recording that items left the warehouse for a sales order) and
**shipping** (buying carrier labels and end-of-day paperwork). Often you do both.

## Step 1 — Fulfill the sales order

Use **`fulfill_sales_order`**. Pass the order number and the items; shipping method,
carrier, cost, warehouse, marketplace and customer are auto-mapped from the order.

```
fulfill_sales_order(
  sales_order_number="SO-018227",
  items=[{"item_upc_or_name": "212", "fulfill_quantity": 1}],
  tracking_number="1Z999AA10123456784",   # optional
  memo="Rush"                              # optional
)
```

- `fulfill_quantity` is optional — defaults to all unfulfilled units; must be
  `0 < qty ≤ unfulfilled_quantity`.
- The order must be in a fulfillable state (`Pending Fulfillment`, `Partial Fulfilled`, or
  `Partial Fulfilled/Pending Invoice`). If it isn't, tell the user the current status.
- The tool resolves the SO number and validates quantities. Confirm the match if the number
  was fuzzy.

For lower-level control you can use `sales_orders_fulfillments` (`create`) directly with a
body like `{"sales_order": 12, "tracking_number": "...", "carrier": 1, "shipped_date": "..."}`,
but prefer `fulfill_sales_order` — it does the lookups for you.

## Step 2 — Create / price / buy the shipment

The current MCP surface exposes shipping as several grouped tools, not as custom actions on
`shipping_shipments`:

- `shipping_shipments` — standard shipment CRUD (`action=list|retrieve|create|update|partial_update`).
- `shipping_shipments_rerate` — rerate a shipment (`action=create`).
- `shipping_shipments_validate_for_purchase` — pre-flight validation before purchase
  (`action=create`).
- `shipping_shipments_buy` — buy the label (`action=create`).
- `shipping_shipments_carriers`, `shipping_shipments_carrier_types`,
  `shipping_shipments_predefined_packages` — reference data (`action=retrieve`).
- `shipping_presets`, `shipping_printing_setup` — saved shipping defaults / printing config.

**Always confirm with the user before `shipping_shipments_buy`** — that spends real postage
money. Show the chosen carrier, service, and price first.

## Step 3 — Batches

- `shipping_batches` — batch shipment records (`action=list|retrieve|create`).
- `shipping_presets` and `shipping_printing_setup` configure defaults and printing behavior.

The current MCP surface does **not** expose scan forms, shipping balance, automation rules,
carrier-mapping helpers, or label-download/tracking-detail helpers as plugin tools. Do not
instruct Claude to call them from this plugin.

## Presentation & cautions

- Report the `SO-…` number, tracking number, carrier/service, and cost — never raw ids.
- Label purchase, batch buy, and scan-form close are irreversible spends/commitments —
  summarize and get an explicit go-ahead.
- If postage `shipping_balance` is insufficient, surface it and stop before buying.

## Next steps

- Invoice the fulfilled order → **invoicing-and-payments**.
- Handle a return → **returns-and-refunds**.
