---
name: purchase-order-lifecycle
description: Run the full procurement cycle in the EasyEsuite ERP — create a purchase order from a vendor and items, receive goods against it, generate the vendor bill, and pay it. Also covers expense bills and vendor prepayments. Use when the user wants to raise a PO, restock, receive a shipment, or record/pay a supplier bill.
---

# Purchase order lifecycle

The procurement chain: **create PO → receive goods → bill → pay**. Each stage has a dedicated
tool. Resolve vendor/warehouse/item names to IDs (or let the workflow tools do it).

## 1. Create the PO

Use **`create_po`** — it resolves vendor and warehouse names automatically and takes a list
of PO items with quantities and costs. Use the task's real field names exactly. (For raising
POs from low-stock findings, start with
**inventory-reports**.)

```
create_po(
  vendor_company_name="Acme Supplies",
  warehouse_name="Main Warehouse",
  po_items=[{"item_upc_or_name": "212", "quantity_ordered": 100, "cost": 4.25}],
  memo="Monthly restock"
)
```

Confirm the vendor/warehouse match (auto-selected first hit + options) as in
**building-workflows**. The response includes the `PO-…` number.

To **edit an existing PO**, use `purchase_orders` (`update`): pass the numeric `id` at top
level and only changed fields in `body` (`memo`, `expected_date`). To change lines, include
`items` where each entry has `id` = the line's `po_item_id` (from `get_po_details`) plus
`quantity_ordered` and/or `cost`.

## 2. Receive goods

Use **`receive_po`** to record receipt of items against the PO (full or partial). This moves
the PO toward `Partial Received` / `Received/Pending Billing` and increases warehouse stock.
Inspect remaining quantities with `get_po_details` first if receiving partially.

`purchase_orders_receipts` lists/▸retrieves receipt records.

## 3. Bill it

Once received, create the vendor bill:

- **`add_bill`** — bill generated from PO receipts (the normal path).
- **`add_immediate_bill`** — bill without a prior receipt step.
- **`add_expense_bill`** / **`create_expense`** — non-inventory expenses.
- `purchase_orders_bills` (`create`/`list`) — direct CRUD; body e.g.
  `{"purchase_order": 1, "due_date": "2025-02-01", "reference": "INV-001"}`.

## 4. Pay it

- **`create_bill_payment`** — record a payment against one or more bills.
- **`create_prepayment`** — vendor prepayment before goods/billing
  (`purchase_orders_vendor_prepayments` to view).
- `purchase_orders_bill_payments` lists payments.

## Statuses to watch

- PO: `Open` → `Partial Received` → `Received/Pending Billing` → `Billed` (or `Voided`).
- Bill: `Open` → `Partially Paid` → `Paid` (or `Voided`).

## Cautions

- Receiving, billing, and paying all change financial/inventory state — confirm amounts and
  quantities with the user before each write.
- Don't over-receive: cap received qty at the outstanding ordered qty.
- Report `PO-…` / bill numbers, never raw ids.

## Related

- Stock-driven reorder lists → **inventory-reports**.
- Vendor setup → **customers-and-vendors**.
- AP exposure / spend reports → **purchasing-reports**, **financial-reports**.
