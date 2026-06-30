---
name: inventory-replenishment-report
description: Build replenishment and low-stock reports from the EasyEsuite ERP MCP surface. Use when the user wants reorder candidates, low-stock or out-of-stock items by warehouse, available quantity by location, or a prioritized restock list that can feed purchasing workflows.
---

# Inventory replenishment report

Use this skill when the user is asking what needs to be reordered, restocked, or moved because
inventory is running low.

## Tools

- `items_warehouse_inventory_items` (`action=list`) — the main stock table by item and warehouse.
- `items_available_quantity_in_warehouses` — one item across all warehouses.
- `find_item` — fuzzy item lookup.
- `find_warehouse` — warehouse lookup.
- `items` (`action=list` / `retrieve`) — item details when needed.

## Core warehouse scan

```
items_warehouse_inventory_items(
  action="list",
  query={
    "warehouse": 3,
    "limit": 50,
    "offset": 0
  }
)
```

Page until complete. This is the base dataset for low-stock and reorder work.

## Replenishment logic

There is no single MCP "reorder report" action. Build it client-side:

1. Page `items_warehouse_inventory_items`.
2. Read on-hand / committed / available quantities per row.
3. Compare available quantity against the reorder point or minimum level present on the row.
4. Keep rows where available is less than or equal to the reorder threshold.
5. Rank by severity, for example:
   - out of stock first
   - then lowest available vs reorder gap
   - then highest-demand SKUs if the user wants priority

## Common variants

### Out of stock

Filter the paged results client-side to `available <= 0`.

### One SKU across all warehouses

```
find_item(query="wireless mouse")
items_available_quantity_in_warehouses(...)
```

Use this when the user wants to know whether stock can be moved instead of reordered.

### One warehouse reorder list

1. Resolve the warehouse with `find_warehouse(query="Miami")`.
2. Page `items_warehouse_inventory_items` for that warehouse.
3. Produce a ranked reorder table.

## Presentation

- Show `SKU/Item | Warehouse | On hand | Committed | Available | Reorder point | Gap`.
- Explain whether the recommendation is to **reorder** or **transfer** when another warehouse
  has available units.
- Keep variant children separate; do not report the parent as stocked.

## Gotchas

- Users usually mean **available**, not on-hand, when they say "in stock."
- Kits do not hold stock directly; their availability derives from components.
- Always page to completeness before claiming company-wide reorder totals.
