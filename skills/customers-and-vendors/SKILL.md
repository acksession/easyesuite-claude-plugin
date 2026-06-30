---
name: customers-and-vendors
description: Manage business contacts and accounts in the EasyEsuite ERP — create and look up customers and vendors, manage financial accounts and banks, and add ERP users. Use when the user wants to onboard a customer/vendor, find an account, or set up a financial account or user.
---

# Customers, vendors, and accounts

The master-data side: the people and accounts that orders, bills, and payments reference.
Create these before transacting against them.

## Customers

- **`create_customer`** — onboard a new customer (name, contact, addresses, terms).
- `accounts_customers` (`list`/`retrieve`/`update`) — find and edit existing customers.
- `find_item`-style lookup: `get_id(entity_type="customer", identifier="ABC Corp")` resolves
  a name to its id.

## Vendors

- **`create_vendor`** — onboard a new supplier.
- `accounts_vendors` (`list`/`retrieve`/`update`) and `find_vendor(query="…")` to find one;
  `get_id(entity_type="vendor", …)` for an exact resolve.

## Financial accounts & banks

- **`create_bank`** — add a bank record.
- `accounts_financial_accounts` / `find_financial_accounts` — GL/financial accounts used for
  deposits, expenses, discounts, etc.; `get_id(entity_type="financial_account", …)`.
- `accounts_banks` — bank account records; `get_id(entity_type="bank", …)`.

## Users

- **`add_user`** — create an ERP user. (User roles drive Cerbos permissions — what each user
  can do via the API and this MCP.)
- `users` / `users_me` — list users and the current identity.

## Typical flows

```
create_customer(... name, contact, address ...)     # then sell to them → create-sales-order
create_vendor(...)                                  # then buy from them → purchase-order-lifecycle
find_vendor(query="Acme")                           # before raising a PO
```

## Cautions

- Check for an existing record (`core_global_search` / `find_*`) before creating a duplicate
  customer/vendor.
- Customer/vendor records carry addresses and terms that flow into orders, invoices, and
  bills — get them right at creation to avoid downstream rework.
- Report names/account labels, never raw ids.

## Related

- Selling → **create-sales-order**; buying → **purchase-order-lifecycle**.
- Money in/out → **invoicing-and-payments**, **financial-reports**.
