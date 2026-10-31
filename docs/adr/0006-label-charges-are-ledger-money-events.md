---
status: accepted
---

# Label Charges are ledger money events, not Order deductions

Purchasing a Label is a seller cost, so it posts a `label.charged` money event into Finance: `SELLER_PAYABLE` is debited and `PLATFORM_CLEARING` credited in the shop's Settlement Currency, as available immediately. Refunding an unused Label posts the reverse entry. The Order itself is untouched: its confirmed money facts and its `payout_availabilities` row are never re-priced, because a label is not part of what the buyer paid.

## Considered alternatives

- **Deduct the label cost from that Order's payout availability.** It looks simpler, but it rewrites an Order's confirmed money facts based on a later operational action, and a label for a multi-package Order has no single Order to deduct from.
- **Track label charges outside the ledger in a `label_charges` table with a shop balance.** It would create a second money authority that payouts do not read, so the seller could be paid out money already spent on labels.

## Consequences

- A shop's Seller Payable can go negative; payouts then offer less than the confirmed Orders would suggest.
- Label Charges appear in the shop's transactions like any other posting, so no separate statement is needed.
- `payout_availabilities` keeps meaning "this Order's Seller Payable became available", not "this Order is worth X after costs".
- Refunding an unused Label requires the reverse posting to be idempotent, keyed by the label, so a replayed refund cannot credit twice.
