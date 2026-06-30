---
name: manage-catalog-items
description: Create and maintain catalog items in the EasyEsuite ERP — add standard items, kits/bundles, and variant items; update item fields; refresh marketplace configurations for an existing item; and attach product images. Use when the user wants to add a product, build a bundle, configure marketplace data, or update an item.
---

# Manage catalog items

Tools for building and maintaining the product catalog. Items can be standard, **kits**
(bundles of components), or **variants** (a parent with child SKUs across axes like size/color).

## Create

- **`add_item`** — create a standard inventory item. The workflow expects every required
  attribute/field to have a non-empty value (never `""`/null), and can configure the item
  across the supported marketplaces (Amazon, Walmart, eBay, Target, Best Buy, Macy's, Mercado).
- **`add_kit`** — create a kit/bundle item built from component items.
- **`add_variant`** — create a variant parent with its child SKUs and variation axes.

This plugin does **not** currently expose a standalone MCP `product_lookup` tool, so do not
instruct Claude to call `product_lookup(...)` here. Use the product data the user provides, or
extend the MCP surface before relying on UPC enrichment.

## Update & configure

- **`update_item`** — change fields on an existing item. Resolve it first with `find_item`
  or `get_id(entity_type="item", ...)`, then pass the id and only the fields to change.
- **`configure_item_marketplaces`** — run the marketplace-config pipeline on an existing
  item. Required fields are `item_id`, `category_query`, and `search_query`. The tool
  generates the marketplace payloads automatically; do **not** pass raw `amazon_marketplace`,
  `ebay_marketplace`, or similar fields yourself.
- `items` (`retrieve`) and `items_variant_items` / `items_kit_package_items` for inspecting
  composition and current marketplace SKUs.

## Images

- **`upload_image`** — attach an image to an item: `upload_image(image_url="https://…",
  item_id=N)`. The URL must be a publicly reachable `https://` link; if the user only has a
  local file, ask them to share a link.
- `get_item_image` — fetch the current image for an item.

## Typical flows

**Add a new product**
```
add_item(... complete fields ...)                                         # create
configure_item_marketplaces(item_id=..., category_query="...", search_query="...")  # optional
upload_image(image_url="…", item_id=…)  # add image
```

**Build a bundle** → `add_kit` with the component items and their quantities.

**Create a variant family** → `add_variant` with axes and child SKUs; bind marketplace SKUs
per child via `configure_item_marketplaces`.

## Cautions

- Fill required fields completely — partial item records fail validation downstream and break
  marketplace listings.
- For variant parents, stock and marketplace SKUs live on the **children**, not the parent.
- Confirm before bulk edits; report items by SKU/name, never raw ids.

## Related

- Stock levels for these items → **inventory-reports**.
- Selling them → **create-sales-order**; listing them on marketplaces → **marketplace-connector-ops**.
