---
name: financial-reports
description: Build financial reports from the EasyEsuite ERP — accounts receivable (invoices, payments, aging), accounts payable (bills, bill payments), credit memos and refunds, bank/financial-account balances, and account transaction/movement history. Use for questions like "what's outstanding", "AR aging", "payments received this month", or "transactions on account X".
---

# Financial reports

Read-only reporting across receivables, payables, and the general ledger side of the ERP.
Follow the date and pagination rules in **erp-quickstart** (and remember invoices accept ONLY
`date_after`/`date_before`).

## Tools

| Area | Tools |
|------|-------|
| Receivables (AR) | `sales_orders_invoices`, `sales_orders_payments`, `sales_orders_immediate_invoices` |
| Credit / refunds | `sales_orders_credit_memos`, `sales_orders_customer_refunds`, `sales_orders_invoice_returns` |
| Payables (AP) | `purchase_orders_bills`, `purchase_orders_bill_payments` |
| Accounts | `accounts_financial_accounts` / `find_financial_accounts`, `accounts_banks` |
| Ledger | `transactions_account_transactions`, `transactions_movements` |
| Categories / journals | `accounting_categories`, `accounting_journals`, `accounting_operation` |

## Core recipes

**Outstanding receivables (open invoices)**
```
sales_orders_invoices  action=list  query={"status":"Open","ordering":"-date"}
```
Page through; sum balances. Add `Partial Paid` for everything not fully collected.

**Payments received this month**
```
sales_orders_payments  action=list  query={...}     # filter by date where supported
```
Or sum payment amounts from invoices marked `Paid` in the period.

**AR aging** — page all open / partial-paid invoices, bucket each by age of its date (or due
date) into 0–30 / 31–60 / 61–90 / 90+ days from today, and sum balances per bucket. Present
the buckets as a table with totals.

**Accounts payable** — mirror AR with `purchase_orders_bills` (status `Open` / `Partially
Paid`) for what's owed, and `purchase_orders_bill_payments` for what's been paid.

**Refund / credit exposure**
```
sales_orders_credit_memos  action=list  query={...}
sales_orders_customer_refunds  action=list  query={...}
```
Sum to offset gross revenue when computing net.

**Account balances & transactions**
```
find_financial_accounts(query="Operating")                 # → account id
transactions_account_transactions  action=list  query={"account": <id>, ...}
```
Use `transactions_movements` for the line-level ledger movements behind a transaction.

## Presentation

- Sum across all pages — never report a total from a single page.
- Net revenue = invoiced − credit memos − refunds.
- Show document numbers (`IN-…`, bill numbers) and account names, never raw ids.
- Aging and balance reports → small inline tables with explicit bucket/period labels.

## Gotchas

- Invoices: NEVER send `created_date_*` — it's silently ignored and returns all-time totals.
  Use `date_after`/`date_before`.
- Distinguish invoice **total** from **balance** (total − payments). "Outstanding" means
  balance, not total.
- A customer refund and a credit memo are different instruments — confirm which the user
  means before combining them.
