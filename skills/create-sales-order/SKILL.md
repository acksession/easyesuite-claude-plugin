---
name: create-sales-order
description: Create a sales order in the EasyEsuite ERP from a customer, marketplace, warehouse, shipping method, and a list of items — names are resolved to IDs automatically, with serial tracking, discounts/charges, and address overrides supported. Use when the user wants to place, draft, or enter a new sales order.
---

# Create a sales order

Use the **`create_sales_order`** tool. It runs the whole resolve-and-create workflow in one
call: it matches the customer / marketplace / warehouse / shipping-method names to IDs, looks
up each item in warehouse inventory, pulls the customer's address, and creates the order.

## Minimum required

- `customer_name` (or `customer_id`)
- `marketplace_name` (or `marketplace_id`) — e.g. "Amazon US", "Walmart", "eBay", "Macys US"
- `warehouse_name` (or `warehouse_id`) — the source warehouse
- `shipping_method_name` (or `shipping_method_id`) — e.g. "Standard", "Express", "Pop"
- `items`: list of `{"item_upc_or_name": ..., "quantity": ..., "price": ...}` (≥ 1)
- Shipping address (auto-filled from the customer; required if the customer has none):
  `address1`, `city`, `zip_code`, `company_name`, `contact_name`, `email`, `country` (default `US`)

## Example

```
create_sales_order(
  customer_name="N-cus",
  marketplace_name="Macys US",
  warehouse_name="SunValley Warehouse",
  shipping_method_name="Pop",
  items=[{"item_upc_or_name": "212", "quantity": 1, "price": 10.00, "description": "title"}],
)
```

## Optional fields

- `serial_required=True` — set whenever the user says "serial" / "serialized". Provide
  per-item `serial_numbers: ["SN001", ...]` inside the item objects.
- `financial_items=[{"expense_name": "Discount", "amount": -10.00}]` — positive = charge,
  negative = discount (resolved to a financial account).
- `shipping_cost`, `tax_amount` — non-negative; default to 0 if omitted.
- `po_number`, `memo`, `date_value`, `ship_date` (ISO `YYYY-MM-DD`, default today).
- Address extras: `address2`, `state`, `cell_phone`, `work_phone`.

## Handling name matches

The tool auto-selects the first match for each name and returns the full options list plus an
`auto_selected` flag. Before trusting it:

- Check the selected name actually matches what the user said. If a better candidate is in
  the options (typo, partial), **re-call with the explicit id** (`warehouse_id=3`, etc.).
- If several are equally plausible, ask the user which they meant.
- Item not found → the item doesn't exist; offer to create it (see **manage-catalog-items**).

## After creation

The response includes `order_number` (e.g. `SO-018419`), an order link, totals, customer,
marketplace, and status (`Open`). Report the `SO-…` number and link inline — never the raw id.
The order then flows into fulfillment → invoicing.

## Next steps

- Ship it → **fulfill-and-ship-orders**.
- Bill it → **invoicing-and-payments**.
- Check stock before promising → **inventory-reports**.

## Notes

- Sub-total = Σ(quantity × price); total = sub-total + shipping + tax + financial items.
- Confirm the order summary with the user before creating if the request was ambiguous —
  this is an outward-facing commitment.
