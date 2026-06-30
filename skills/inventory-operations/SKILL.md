---
name: inventory-operations
description: Move and adjust stock in the EasyEsuite ERP — transfer inventory between warehouses (create, complete, void transfers) and make manual quantity adjustments (cycle counts, damage, shrinkage, corrections). Use when the user wants to move stock between locations or correct on-hand quantities.
---

# Inventory operations

Write operations on stock levels: **transfers** (move units between warehouses) and **manual
adjustments** (correct on-hand counts). Always check current levels first with
**inventory-reports** so you move/adjust the right quantities.

## Transfers (warehouse → warehouse)

A transfer has a lifecycle: created (in transit) → completed (received at destination), with
the option to void.

- **`create_transfer`** — start a transfer: source warehouse, destination warehouse, and the
  items + quantities. Resolves warehouse and item names automatically.
- **`complete_transfer`** — mark a transfer received at the destination (moves the units into
  destination on-hand). Resolve the transfer first with `get_id(entity_type="transfer", …)`.
- **`void_transfer`** — cancel a transfer that shouldn't proceed.
- `inventory_transfers` (`list`/`retrieve`) to inspect transfer records.

```
create_transfer(
  from_warehouse_name="Main Warehouse",
  to_warehouse_name="Miami",
  items=[{"item_upc_or_name": "212", "quantity": 25}],
)
```

Don't transfer more than the source has available — confirm with
`items_available_quantity_in_warehouses` / `find_warehouse_item` first.

## Manual adjustments

- **`add_adjustments`** — adjust on-hand quantity for items in a warehouse (cycle count
  correction, damage, shrinkage, found stock). Use `warehouse_name` plus
  `adjustment_items=[{"item_upc_or_name": "...", "quantity": ...}]`, with a reason/memo.
- `inventory_transfers_manual_adjustments` (`list`/`retrieve`) to view adjustment records;
  `get_id(entity_type="adjustment", …)` to resolve one by number.

## Cautions

- Adjustments and completed transfers **change physical stock records** — confirm the item,
  warehouse, and quantity with the user before writing, and capture a reason/memo for audit.
- A transfer in transit has left the source but not yet arrived — available counts at both
  ends reflect that until you `complete_transfer`.
- Report transfer/adjustment numbers and warehouse/SKU names, never raw ids.

## Related

- Before moving: **inventory-reports** to see what's where.
- Restocking from vendors instead of moving: **purchase-order-lifecycle**.
