# EasyEsuite ERP for Claude

**Run your EasyEsuite ERP in plain English.** This plugin connects Claude to your
EasyEsuite tenant so you can pull reports and drive real operational workflows —
sales orders, fulfillment, purchasing, invoicing, returns, and marketplace
connectors — just by asking.

Built for teams running e-commerce on EasyEsuite.

---

## Why you'll want it

- **Ask, don't click.** "How many orders did we get last week by marketplace?"
  "What's low on stock?" "Create a sales order for Acme with 5 units of SKU-1024."
  Claude routes to the right ERP action and answers.
- **Real workflows, end to end.** Not just lookups — Claude can place an order and
  fulfill it, raise a PO and receive it, invoice a customer and record the payment.
- **Your data, your permissions.** Everyone signs in with their own EasyEsuite
  account. Every action runs as that user and is authorized by their real role —
  a viewer can only read; the plugin never exposes data you aren't allowed to see.
- **Zero setup.** Install, sign in, go. No keys to manage, no scripts to run.

## What you can ask

**Reporting**

- **Sales** — "How did we do yesterday — orders and revenue?" · "Revenue by
  marketplace, this month vs. last." · "What sold best in April?"
- **Inventory** — "What's low or out of stock by warehouse?" · "How many units of
  SKU-1024 across all locations?" · "Give me a prioritized restock list."
- **Purchasing** — "Show me open POs and what's pending receipt." · "How much did
  we spend with Acme Supply this quarter?"
- **Finance** — "AR aging — who's overdue?" · "Payments received this month." ·
  "Which vendors are driving next week's cash-out?"
- **Backlog** — "How old is our unshipped-order backlog, bucketed by age?"

**Operations**

- **Order to cash** — "Create a sales order for Northwind: 3x SKU-1024 from the LA
  warehouse." · "Fulfill SO-018419 and buy a shipping label." · "Invoice it and
  record the payment."
- **Procure to pay** — "Raise a PO to restock everything below reorder point." ·
  "Receive PO-000762 and generate the vendor bill."
- **Returns** — "Set up a return for invoice IN-075558 and issue the refund."
- **Catalog & stock** — "Add a new bundle item." · "Transfer 50 units from LA to NJ."

…and more across customers & vendors and support tickets.

## Requirements

- An active **EasyEsuite account and subscription** — the plugin is your interface
  to *your* tenant; it grants no access on its own.
- Claude with plugin support (Claude Cowork).

## What's inside

A bundled MCP connection to the EasyEsuite ERP plus a library of **20 skills** that
teach Claude how to use it correctly — covering sales, inventory, purchasing and
financial reporting, and the full order-to-cash and procure-to-pay workflows.

## Support

- Website: <https://easyesuite.com>
- Questions or issues: **admin@easyesuite.com**

## License

Apache-2.0 — see [LICENSE](./LICENSE). Use of the EasyEsuite ERP itself is governed
by your EasyEsuite subscription and Terms of Service.
