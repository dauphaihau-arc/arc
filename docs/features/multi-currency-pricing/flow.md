# Multi-Currency Pricing Flows

This document captures end-to-end lifecycle examples for the current multi-currency pricing model.

For the target model, invariants, and data design, see [README.md](README.md).

## Flow 1: Seller creates product pricing for storefront display

```mermaid
sequenceDiagram
    actor Seller
    participant Web as Seller Web
    participant API as Product Pricing API
    participant Catalog as Catalog Pricing
    participant Projection as Catalog Projection
    participant FX as FX Service
    participant Storefront as Storefront Reads

    Seller->>Web: Create or update product base price
    Web->>API: Submit seller pricing update
    API->>Catalog: Close active base row and insert new base price
    API-->>Web: Return updated base pricing snapshot
    API-->>Projection: Product pricing changed
    Projection->>Catalog: Resolve active base or market price
    opt Indexed market or currency needs conversion
        Projection->>FX: Resolve rate and rounding policy
    end
    Projection->>Projection: Write indexed storefront price snapshots
    Storefront->>Projection: Read configured market/currency pair
    Projection-->>Storefront: Backend-resolved display price
```

Summary:

- seller authors canonical base pricing
- the current seller pricing write API does not expose market override creation
- catalog projection precomputes storefront pricing for configured indexed market/currency pairs
- backend decides whether to use an existing market override or base-price conversion
- storefront receives backend-resolved display pricing

## Flow 2: Seller views product pricing in seller dashboard

```mermaid
sequenceDiagram
    actor Seller
    participant Web as Seller Dashboard
    participant API as Seller Product API
    participant Catalog as Catalog Pricing

    Seller->>Web: Open product pricing view
    Web->>API: Load seller product draft or detail
    API->>Catalog: Resolve canonical base pricing per inventory row
    Catalog-->>API: Base pricing snapshots
    API-->>Web: Product response with current base pricing
    Web-->>Seller: Display seller-authored pricing
```

Summary:

- seller dashboard currently exposes the base pricing snapshot used by the product draft summary
- market override rows may exist in persistence, but they are not surfaced through the normal seller pricing write/read flow today

## Flow 3: Buyer creates checkout from storefront

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

## Indexed Versus Fallback Price Resolution

```mermaid
flowchart TD
    A[Storefront request includes market and currency] --> B{Market/currency pair is indexed?}
    B -- Yes --> C[Read projected Mongo or Atlas storefront price]
    B -- No --> D[Load active canonical price from variant_prices]
    D --> E{Market override exists?}
    E -- Yes --> F[Use market override as source price]
    E -- No --> G[Use active base price]
    F --> H{Display currency matches source currency?}
    G --> H
    H -- Yes --> I[Return resolved display amount]
    H -- No --> J[Apply FX conversion and rounding policy]
    J --> K[Cache fallback result for short TTL when configured]
    K --> I
    C --> I
```

Summary:

- indexed pairs use precomputed storefront price documents for read performance
- fallback pairs continue to resolve from canonical `variant_prices`
- market override rows win over base-price conversion when present
- FX conversion is a browsing concern until quote creation persists transactional amounts
