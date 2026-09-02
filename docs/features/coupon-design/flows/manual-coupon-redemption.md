# Manual Coupon Redemption

This flow describes a buyer entering a coupon code during checkout.

```mermaid
sequenceDiagram
    actor Buyer
    participant Web as Storefront Web
    participant Quote as Checkout Quote API
    participant Coupon as Coupon Pricing
    participant Order as Order Creation API

    Buyer->>Web: Enter promo code
    Web->>Quote: Create checkout quote with promo code
    Quote->>Coupon: Validate coupon code and rules
    Coupon-->>Quote: Discount adjustment or rejection
    Quote->>Quote: Persist item, shipping, and adjustment snapshots
    Quote-->>Web: quote_id and quote-owned totals
    Web->>Order: Create order with quote_id
    Order->>Quote: Reload validated quote
    Order-->>Web: Payment redirect or created order shops
```

Summary:

- manual coupon redemption happens at checkout quote creation
- checkout quote owns the final totals and adjustment snapshots
- manual redemption can apply usage limits, minimum order/product rules, and buyer context
- manual redemption does not depend on catalog auto-sale projections
