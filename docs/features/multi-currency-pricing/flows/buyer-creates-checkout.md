# Buyer Creates Checkout From Storefront

This flow describes how storefront display pricing becomes authoritative transactional pricing during checkout.

```mermaid
sequenceDiagram
    actor Buyer
    participant Storefront
    participant ProductAPI as Storefront Product API
    participant Pricing as Price Resolution
    participant Quote as Checkout Quote
    participant Order as Order Creation
    participant Payment as Payment Gateway

    Buyer->>Storefront: Browse products
    Storefront->>ProductAPI: Request products with market/currency context
    ProductAPI->>Pricing: Resolve display pricing
    alt Indexed market/currency pair
        Pricing->>Pricing: Read projected storefront price
    else Non-indexed pair
        Pricing->>Pricing: Resolve canonical price with FX fallback
    end
    Pricing-->>ProductAPI: Display pricing
    ProductAPI-->>Storefront: Product response

    Buyer->>Storefront: Start checkout
    Storefront->>Quote: Create quote from cart or buy-now scope
    Quote->>Pricing: Resolve checkout currency and item prices
    Quote->>Quote: Persist quoted totals and provenance
    Quote-->>Storefront: quote_id and checkout totals
    Storefront->>Order: Create order with quote_id
    Order->>Quote: Reload and validate quote
    Order->>Order: Persist order money snapshots
    Order->>Payment: Create payment from quoted totals
    Payment-->>Order: Payment session or intent
```

Summary:

- storefront display currency may differ from checkout currency
- configured indexed market/currency pairs are served from projected Mongo/Atlas pricing documents
- non-indexed pairs fall back to live backend resolution from canonical prices, with short-lived caching
- transactional pricing becomes authoritative at quote creation
- payment uses persisted quote amounts and does not reprice
- orders currently keep quote linkage in `payment_details.quote_id`
