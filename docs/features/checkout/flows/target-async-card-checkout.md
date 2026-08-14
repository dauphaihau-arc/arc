# Target Async Card Checkout

This flow describes the target transactional outbox design for asynchronous card checkout session creation.

```mermaid
sequenceDiagram
    actor Buyer
    participant Web as Storefront Web
    participant Order as Order Creation API
    participant Outbox as Checkout Outbox
    participant Worker as Checkout Worker
    participant Payment as Payment Gateway

    Web->>Order: Create card order with quote_id
    Order->>Order: Persist local orders as CHECKOUT_PENDING
    Order->>Outbox: Persist checkout-session-requested event
    Order-->>Web: checkout_pending and order ids
    Web-->>Buyer: Show checkout preparation state
    Worker->>Outbox: Claim event
    Worker->>Payment: Create checkout session
    Payment-->>Worker: Checkout session URL
    Worker->>Order: Update orders to AWAITING_PAYMENT
    Web->>Order: Poll session readiness
    Order-->>Web: checkout_session_url
    Web-->>Buyer: Redirect to payment provider
```

Summary:

- this is the target flow described by the checkout transactional outbox design
- card order creation can persist local orders as `CHECKOUT_PENDING` before the payment checkout session exists
- the worker claims the outbox event, creates the payment-provider session, and updates orders to `AWAITING_PAYMENT`
- the current storefront still expects `checkout_session_url` immediately, so adopting this flow requires an explicit frontend/API contract update
