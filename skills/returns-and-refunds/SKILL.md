---
name: returns-and-refunds
description: Handle customer returns and refunds in the EasyEsuite ERP — set up a return for a sales order/invoice, process the return refund, issue credit memos and refund against them, and record invoice returns. Use when a customer is returning items or owed money back.
---

# Returns and refunds

The reverse-logistics side: take items back and give money back. Resolve the originating
sales order / invoice first (`get_id` for `sales_order` / `invoice`, or list-with-search).

## Set up and process a return

- **`create_return_setup`** — configure a return against a sales order / invoice (which
  items, quantities, and the return terms). `sales_orders_return_setups` lists/retrieves them.
- **`add_return_refund`** — process the refund for a return (money back to the customer for
  returned items).
- `sales_orders_invoice_returns` (`list`/`retrieve`/`create`) — the invoice-return records.

## Credit memos and refunds

- `sales_orders_credit_memos` — issue/inspect credit memos (a credit the customer can apply
  to future orders).
- **`refund_credit_memo`** — pay out a credit memo as an actual refund. Resolve the credit
  memo with `get_id(entity_type="credit_memo", …)` first.
- `sales_orders_customer_refunds` — the customer-refund records;
  `get_id(entity_type="refund", …)` resolves one by number.

## Typical flow

```
get_id(entity_type="invoice", identifier="IN-075558")     # → invoice id
create_return_setup(... items, quantities ...)            # authorize the return
add_return_refund(...)                                    # refund the returned items
# or, for a standalone credit:
sales_orders_credit_memos  action=create  body={...}
refund_credit_memo(...)                                   # pay it out
```

## Cautions

- Refunds move real money and returns move real stock — confirm the items, quantities, and
  amounts with the user before each write.
- A **credit memo** (store credit) and a **customer refund** (cash back) are different
  instruments; confirm which the customer is owed.
- Returned items affect inventory — check whether the return restocks the warehouse.
- Report `IN-…` / credit-memo / refund numbers, never raw ids.

## Related

- The original sale/invoice → **invoicing-and-payments**.
- Refund/credit exposure reporting → **financial-reports**.
