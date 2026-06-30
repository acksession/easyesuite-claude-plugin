---
name: marketplace-connector-ops
description: Operate the narrow Connector V2 MCP surface currently exposed by EasyEsuite — inspect connector account records and carrier mappings. Use when the user wants to review connector account data that is actually available through this plugin.
---

# Marketplace connector operations

Connector V2 links the ERP to marketplaces (Temu, TikTok Shop). The current MCP plugin exposes
only a small subset of that connector surface. This skill intentionally documents only the
tools that are actually available.

## Exposed tools

- `connector_v2_accounts` — standard grouped CRUD over connector account records
  (`action=list|retrieve|create|update|destroy`).
- `connector_v2_carrier_mappings` — standard grouped CRUD over connector carrier-routing
  mappings (`action=list|retrieve|create|update|partial_update|destroy`).

## Not exposed by this plugin

Do **not** instruct Claude to call the following from this plugin today:

- `connector_v2_order_records`
- `connector_v2_accounts` custom sub-actions such as `alerts_retrieve`, `schema_retrieve`,
  `categories_retrieve`, `listings_mirror_retrieve`, `mapping_resolve_create`,
  `fulfillment_reconcile_create`, or `product_lookup`
- `connector_v2_warehouse_mappings`
- `connector_v2_payment_configs`
- `connector_v2_settlements`
- `connector_v2_settlements_summary`

Those workflows may exist elsewhere in the ERP/backend, but they are not part of the MCP
surface this plugin currently exposes.

## Cautions

- Ingesting an order **creates a real ERP sales order** — confirm SKU mappings are correct
  first; an unmapped line blocks ingest (and should — don't force around it).
- Muting alerts hides real sync problems; only mute with the user's intent and for a bounded
  period.
- Report marketplace order ids / SKUs and ERP `SO-…` numbers clearly; never raw DB ids.

## Related

- Catalog/SKU setup in ERP items → **manage-catalog-items**.
