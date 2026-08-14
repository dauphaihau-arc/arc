# Cart Checkout

This flow describes checkout from selected cart items.

```mermaid
sequenceDiagram
    actor Buyer
    participant Web as Storefront Web
    participant Cart as Cart State
    participant Quote as Checkout Quote API
    participant Order as Order Creation API

    Buyer->>Web: Open /cart/checkout
    Web->>Cart: Load selected cart items
    Buyer->>Web: Select address, payment, notes, and promo codes
    Web->>Quote: POST /checkout/cart/quote or authenticated equivalent
    Quote->>Cart: Validate selected cart scope
    Quote->>Quote: Persist money and shop adjustment snapshots
    Quote-->>Web: quote_id, totals, items, expires_at
    Web->>Order: POST /checkout/cart or authenticated equivalent
    Order->>Quote: Reload and validate quote
    Order-->>Web: Payment redirect or created order shops
```

Summary:

- cart checkout quote input comes from selected cart items plus buyer shipping and shop-level adjustments
- the current storefront route is `/cart/checkout`
- the quote response includes `quote_id`, totals, item snapshots, and expiration
- order creation reloads and validates the quote before returning a payment redirect or created order shops
