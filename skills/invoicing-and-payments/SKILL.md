---
name: invoicing-and-payments
description: Invoice sales orders and record customer payments in the EasyEsuite ERP — create invoices (including immediate invoices), apply payments and payment receipts, and pay/settle open invoices. Use when the user wants to bill a customer, record money received, or close out an invoice.
---

# Invoicing and payments

The receivables side: turn fulfilled sales orders into **invoices**, then record **payments**
against them. Resolve the sales order / invoice first (`get_id`, or `sales_orders_invoices`
`list` with `search`).

## Create invoices

- **`add_invoice`** — create an invoice for an existing sales order (the normal path,
  typically after fulfillment).
- **`add_immediate_invoice`** — create an invoice immediately (combined sale + invoice flow).
- `sales_orders_invoices` (`create`) — direct CRUD when you already have the IDs/body.
- `sales_orders_immediate_invoices` for the immediate-invoice resource.

Invoice statuses: `Open`, `Partial Paid`, `Paid`, `Voided`.

## Record payments

- **`apply_payment`** — apply a customer payment to a sales order / invoice.
- **`pay_invoice`** — pay/settle a specific invoice.
- **`payment_receipt`** — record a payment receipt (cash in), with allocation to documents.
- `sales_orders_payments` (`create`/`list`) — direct payment CRUD; body e.g.
  `{"sales_order": 1, "payment_method": 1, "amount": "150.00", "payment_date": "2024-01-15"}`.

Resolve the payment method with `get_id(entity_type="payment_method", …)` and the deposit
account with `find_financial_accounts` when needed.

## Editing invoices

- `sales_orders_invoices` (`update`) changes **header** fields only (`due_date`, `memo`,
  `terms`) — pass the numeric `id` at top level.
- To change a **line item** quantity or price, use `sales_order_items` (`update`) instead —
  the invoice header update does NOT touch lines.
- A draft invoice that was never posted can be removed with `sales_orders_invoices`
  (`destroy`).

## Typical flow

```
sales_orders_invoices  action=list  query={"search":"IN-075558"}   # → numeric id
add_invoice(...)                                                   # or create from SO
apply_payment(... amount, method ...)                              # record money in
```

## Cautions

- Creating invoices and applying payments are financial commitments — confirm amounts,
  customer, and the target document with the user first.
- `retrieve` needs the **numeric** invoice id, not the `IN-…` string — `list` with `search`
  to get it.
- "Outstanding" = invoice balance (total − payments), not the total. Report `IN-…` numbers
  and amounts, never raw ids.

## Related

- Reporting on AR / aging → **financial-reports**.
- Returns and credit-driven refunds → **returns-and-refunds**.
