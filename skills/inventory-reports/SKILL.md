---
name: inventory-reports
description: Report on stock and inventory in the EasyEsuite ERP — on-hand / committed / available quantities by warehouse, low-stock and out-of-stock items, reorder candidates, and per-item availability across locations. Use for questions like "what's low on stock", "how many units of X across warehouses", or "what do we need to reorder".
---

# Inventory reports

Read-only reporting over the catalog and per-warehouse stock. Inventory is tracked **per item
per warehouse**, so most questions need both dimensions.

## Tools

| Need | Tool |
|------|------|
| Catalog list / item detail | `items` (`list`, `retrieve`) |
| Stock per item per warehouse (on-hand, committed, available) | `items_warehouse_inventory_items` (`list`, `retrieve`) |
| Available qty of one item across all warehouses | `items_available_quantity_in_warehouses` |
| Fuzzy item lookup | `find_item` |
| Item's stock in a specific warehouse | `find_warehouse_item` |
| Warehouse lookup | `find_warehouse` / `items_warehouses` |
| Inventory item master (costs, tracking) | `items_inventory_items` |
| Kit/bundle composition | `items_kit_package_items` |
| Variant parent/children | `items_variant_items` |

## Core recipes

**Stock of one item everywhere**
```
find_item(query="wireless mouse")                       # → item id + name
items_available_quantity_in_warehouses(... item ...)    # available qty per warehouse
```
Present a per-warehouse table: Warehouse | On hand | Committed | Available.

**All inventory in a warehouse**
```
find_warehouse(query="Miami")                           # → warehouse id
items_warehouse_inventory_items  action=list  query={"warehouse": <id>, "limit": 50}
```
Page through all rows; report `_total_records` for "how many SKUs".

**Low-stock / reorder candidates** — there's no single "low stock" filter, so page
`items_warehouse_inventory_items` and compare each row's available quantity against its
reorder point / min level (fields on the row). Collect rows where available ≤ reorder point,
then present them ranked by how far below they are. These are your reorder candidates — hand
them to **purchase-order-lifecycle** to raise POs.

**Out-of-stock** — same pass, filter to available ≤ 0.

## Presentation

- Distinguish **on-hand** (physically present), **committed** (reserved for open orders), and
  **available** (on-hand − committed). Users usually mean *available* when they say "in
  stock".
- For variant parents, report per-child stock, not the parent (which has none of its own).
- Kits don't hold stock directly — availability derives from component stock; note that when
  reporting a kit.
- Compact inline table, document-friendly names/SKUs, no raw ids, never an artifact.

## Gotchas

- Always page to completeness before claiming totals — `_total_records` tells you when there
  are more pages.
- The same SKU can exist in multiple warehouses; sum across warehouses for a company-wide
  number, but keep them separate when the user asks about a specific location.
