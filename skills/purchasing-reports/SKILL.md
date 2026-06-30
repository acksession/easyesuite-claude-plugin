---
name: purchasing-reports
description: Report on procurement in the EasyEsuite ERP — purchase order status and counts, open vs. received vs. billed POs, vendor spend, receipts, and outstanding bills. Use for questions like "open POs", "how much did we buy from vendor X", or "what's pending receipt".
---

# Purchasing reports

Read-only reporting over purchase orders, receipts, and bills. Follow the date and pagination
rules in **erp-quickstart**.

## Tools

| Need | Tool |
|------|------|
| PO list / counts / status | `purchase_orders` (`list`) |
| PO detail + line items | `get_po_details` or `purchase_orders` (`retrieve`) |
| PO line items | `purchase_order_item` |
| Goods received | `purchase_orders_receipts` |
| Supplier invoices (bills) | `purchase_orders_bills` |
| Expense / immediate bills | `purchase_orders_expense_bills`, `purchase_orders_immediate_bills` |
| Bill payments | `purchase_orders_bill_payments` |
| Vendor prepayments | `purchase_orders_vendor_prepayments` |
| Vendor lookup | `find_vendor` / `accounts_vendors` |

## Core recipes

**Open POs**
```
purchase_orders  action=list  query={"status":"Open"}
```
Valid statuses: `Open`, `Voided`, `Partially Received/Pending Billing`, `Partial Received`,
`Received/Pending Billing`, `Billed`. Report `_total_records`.

**POs for a period**
```
purchase_orders  action=list  query={"date_after":"2026-04-01T00:00:00Z","date_before":"2026-04-30T23:59:59Z"}
```
(`date_after`/`date_before` filter the PO order date.)

**Vendor spend**
```
find_vendor(query="Acme Supplies")            # → vendor id
purchase_orders  action=list  query={"vendor": <id>, "date_after":..., "date_before":...}
```
Page through and sum PO totals. For *paid* spend, use `purchase_orders_bills` (filter
`vendor_account` + date) and sum, or `purchase_orders_bill_payments` for cash actually paid.

**Pending receipt** — POs in `Open` / `Partial Received` status still have goods to receive.
**Pending billing** — `Received/Pending Billing` POs have arrived but aren't yet invoiced.

**Outstanding bills (AP)**
```
purchase_orders_bills  action=list  query={"status":"Open"}
```
Statuses: `Open`, `Paid`, `Partially Paid`, `Voided`. Sum open + partially-paid balances for
accounts-payable exposure.

## Presentation

- Report `_total_records` for counts; sum across all pages for spend.
- Show `PO-…` / bill numbers, never raw ids.
- Distinguish committed spend (PO total), received value (receipts), and cash paid (bill
  payments) — they answer different questions.
- Compact inline table, never an artifact.

## Gotchas

- Don't use `created_date_*` for "POs last week" — use `date_after`/`date_before`.
- A PO can be partially received and partially billed; status reflects the furthest stage
  reached, so read line items (`get_po_details`) when the user needs receipt/billing detail.
