# Seller Creates Product Pricing For Storefront Display

This flow describes how seller-authored base pricing becomes backend-resolved storefront display pricing.

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
