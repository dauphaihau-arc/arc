# Buy-Now Checkout

This flow describes checkout from a temporary buy-now cart.

```mermaid
sequenceDiagram
    actor Buyer
    participant Web as Storefront Web
    participant TempCart as Temporary Cart
    participant Quote as Checkout Quote API
    participant Order as Order Creation API

    Buyer->>Web: Start buy-now from product detail
    Web->>TempCart: Create or resolve temporary cart id
    Web-->>Buyer: Open /checkout?c=:cart_id
    Buyer->>Web: Select address, payment, note, and promo codes
    Web->>Quote: POST /checkout/buy-now/quote or authenticated equivalent
    Quote->>TempCart: Validate temporary cart scope
    Quote->>Quote: Persist money snapshot
    Quote-->>Web: quote_id, totals, items, expires_at
    Web->>Order: POST /checkout/buy-now or authenticated equivalent
    Order->>Quote: Reload and validate quote
    Order-->>Web: Payment redirect or created order shops
```

Summary:

- buy-now checkout is scoped by the temporary cart id in the `c` route query
- the current storefront route is `/checkout?c=:cart_id`
- quote creation validates the temporary cart scope and persists the money snapshot
- order creation consumes the buy-now quote before returning a payment redirect or created order shops
