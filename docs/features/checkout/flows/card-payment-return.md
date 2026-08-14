# Card Payment Return

This flow describes how the success page recovers order shops after a payment-provider redirect.

```mermaid
sequenceDiagram
    actor Buyer
    participant Payment as Payment Gateway
    participant Web as Success Page
    participant API as Checkout Session API

    Buyer->>Payment: Complete card payment
    Payment-->>Web: Redirect to /success?session_id=:id
    Web->>API: GET /checkout/session/:id
    alt Order shops resolved
        API-->>Web: order_shops
        Web-->>Buyer: Show confirmed order shops
    else Backend waking up
        API-->>Web: 502, 503, or 504
        Web-->>Buyer: Show waiting state
    else Session not found
        API-->>Web: Not found
        Web-->>Buyer: Show not-found error
    end
```

Summary:

- card payment redirects back to `/success?session_id=:id`
- the success page can recover card-created order shops from the checkout session id rather than relying only on browser memory
- transient `502`, `503`, or `504` responses show a waiting state while the backend wakes up
- missing sessions show a not-found error
