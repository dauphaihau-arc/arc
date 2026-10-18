---
status: accepted
---

# Record shop money in an append-only double-entry ledger fed by Order money events

Arc keeps collecting every checkout into one platform payment-provider account, and gains a Finance context that records the consequences of that money in an append-only double-entry ledger: one Journal Entry per Money Event, identified by the event that caused it, with Postings that balance to zero in their currency, and a shop's Seller Payable projected from those Postings instead of computed on read from Order facts. Payouts leave through a payout adapter port, so provider-side per-shop accounts can be adopted later without changing what the ledger records. Finance records money the provider has already moved; it never decides whether a payment succeeds, and it never re-prices a confirmed Order.

## Considered alternatives

- **Compute financial statements on read from the Order snapshot.** The Order already freezes its money facts, so statements could be derived per request. Rejected because it cannot represent a provider reversal that arrives after the fact, and because it silently re-prices history the moment a commission rule or refund policy changes.
- **Keep an immutable signed money-event table per shop instead of a ledger.** Simpler to write, but it has no counterparty account, so platform clearing and platform revenue cannot be separated and reconciliation against the provider has nothing to compare against.
- **Adopt provider-side per-shop accounts (connected accounts, destination charges) as part of this work.** Rejected as the starting point: shops have no country, tax identity, or destination account today, so per-shop accounts would gate all ledger work on a commercial and KYC decision and would change the live checkout path. The payout adapter port keeps this reachable.
- **Write Postings inside the Order confirmation and refund transactions.** Atomic with the Order fact, but it couples Ordering to Finance's tables and gives replays no protection. The repo already has a durable outbox with claim, lease, and retry semantics for exactly this shape of cross-context work.

## Consequences

- A Money Event is the ledger's trigger. Today the provider is an incomplete source of money facts: only `checkout.session.completed` and `checkout.session.expired` are handled, refunds are driven by an internal queue job rather than a provider webhook, and no `charge.refunded` event is handled at all. Finance consumes Order's recorded money facts and gets a provider-authoritative refund path later.
- One provider charge covers every shop Order in a cart, in one currency, and the order-level allocation seam already exists as each Order's own total. The existing checkout amount-agreement invariant keeps the provider charge and persisted Order rows in agreement; Finance asserts the same relationship between provider money and its Postings.
- Seller Payable accrues in the currency an Order was paid in; the ledger performs no currency conversion and holds no exchange rate.
- A shop's commission schedule and payout destination are new Finance-owned data: nothing on the shop or marketplace today carries a commission rate, tax identity, settlement currency, or payout account.
- Partial refunds require the payment port to carry an amount. Today a refund returns the whole Payment Intent.
- Journal Entries and their Postings must outlive Order retention: a ledger row is never deleted to follow an Order's lifecycle, and the two are related by identity, not by cascade.
- Reconciliation is a first-class job rather than an assumption: the ledger's positions and the provider's own record of money moved are compared on a cadence, and a disagreement is an exception for an operator, never a silent adjustment.

## Verification

Architecture checks must keep the domain's `domain/` layer free of persistence and provider imports, and the port for payouts must live outside `domain/` and `app/` so no vendor type crosses the boundary. Behavioral verification belongs at the persistence and HTTP seams this repo already uses: an integration test against real PostgreSQL that records a Money Event twice and proves no second Journal Entry appears, that a Journal Entry's Postings sum to zero in their currency, and that a shop's projected Seller Payable equals the sum of its Postings after a charge, a partial refund, and a payout. Payout execution is verified against the adapter port with a stub, and reconciliation against recorded provider outcomes, never against a live provider.
