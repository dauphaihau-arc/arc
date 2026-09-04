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
    Order-->>Order: Commit local order transaction
    Order->>Outbox: Try processing the event once inline
    alt Checkout session ready within inline timeout
        Outbox->>Payment: Create checkout session
        Payment-->>Outbox: Checkout session URL
        Outbox->>Order: Update orders to AWAITING_PAYMENT
        Order-->>Web: checkout_session_url
        Web-->>Buyer: Redirect to payment provider
    else Checkout session still pending
        Order-->>Web: checkout_pending and order ids
        Web-->>Buyer: Keep checkout preparation state
        Worker->>Outbox: Claim event or retry pending event
        Worker->>Payment: Create checkout session
        Payment-->>Worker: Checkout session URL
        Worker->>Order: Update orders to AWAITING_PAYMENT
        Web->>Order: Poll session readiness by order ids
        Order-->>Web: checkout_session_url
        Web-->>Buyer: Redirect to payment provider
    end
```

Summary:

- card order creation persists local orders as `CHECKOUT_PENDING` before the payment checkout session exists
- after commit, the API tries to process the created checkout outbox event once inline so the normal fast path can still return `checkout_session_url`
- if inline processing times out or the event is not ready, the response remains `checkout_pending` with order ids and the storefront polls readiness by order ids
- the worker claims or retries pending checkout events, creates the payment-provider session, and updates orders to `AWAITING_PAYMENT`
