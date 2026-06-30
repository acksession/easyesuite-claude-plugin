---
name: support-tickets
description: Work with Zendesk support tickets surfaced in the EasyEsuite ERP — list and read tickets, inspect the currently exposed tag metadata, and create new tickets. Use when the user asks about support tickets, customer issues, or wants to open a ticket.
---

# Support tickets

The ERP surfaces Zendesk support tickets so you can triage customer issues alongside their
orders and accounts.

## Read

- `zendesk_tickets` (`list`/`retrieve`) — browse and open tickets. Each ticket record carries
  its own `tags` array (and subject, status, priority, requester) — read tags off the row;
  there is no separate tags tool.

The current MCP surface does **not** expose a ticket-tags tool or a ticket-comments tool, so do
not instruct Claude to call `zendesk_tickets_tags_list` or `zendesk_tickets_comments_list`.

## Create

- **`create_ticket`** — open a new support ticket (subject, description, requester, priority
  as applicable).

## Typical flow

```
zendesk_tickets  action=list  query={...}        # find open/unresolved tickets (tags are on each row)
zendesk_tickets  action=retrieve  id=<id>        # read one
create_ticket(...)                               # or raise a new one
```

## Tips

- When a ticket is about an order/customer, cross-reference with **sales-reports** /
  **customers-and-vendors** to pull the order or account context into your answer.
- Summarize ticket threads concisely inline; quote the customer's key message rather than
  dumping the whole thread.
- Report ticket numbers/subjects, never raw ids.
